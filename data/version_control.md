# 处理冲突

当你使用`索引`API来更新一个文档时，我们先看到了原始文档，然后修改它，最后一次性地将**整个新文档**进行再次索引处理。Elasticsearch会根据请求发出的顺序来选择出最新的一个文档进行保存。但是，如果在你修改文档的同时其他人也发出了指令，那么他们的修改将会丢失。

很长时间以来，这其实都不是什么大问题。或许我们的主要数据还是存储在一个关系数据库中，而我们只是将为了可以搜索，才将这些数据拷贝到Elasticsearch中。或许发生多个人同时修改一个文件的概率很小，又或者这些偶然的数据丢失并不会影响到我们的正常使用。

但是有些时候如果我们丢失了数据就会出**大问题**。想象一下，如果我们使用Elasticsearch来存储一个网店的商品数量。每当我们卖出一件，我们就会将这个数量减少一个。

突然有一天，老板决定来个大促销。瞬间，每秒就产生了多笔交易。并行处理，多个进程来处理交易：

![无并发控制的后果](/images/03-01_concurrency.png "无并发控制的后果")

`web_1`中`库存量`的变化丢失的原因是`web_2`并不知道它所得到的`库存量`数据是是过期的。这样就会导致我们误认为还有很多货存，最终顾客就会对我们的行为感到失望。

当我们对数据修改得越频繁，或者在读取和更新数据间有越长的空闲时间，我们就越容易丢失掉我们的数据。

以下是两种能避免在并发更新时丢失数据的方法：

### 悲观并发控制（PCC）

这一点在关系数据库中被广泛使用。假设这种情况很容易发生，我们就可以组织对这一资源的访问。典型的例子就是当我们在读取一个数据前先锁定这一行，然后确保只有读取到数据的这个线程可以修改这一行数据。

### 乐观并发控制（OCC）

Elasticsearch所使用的。假设这种情况并不会经常发生，也不会去组织某一数据的访问。然而，如果基础数据在我们读取和写入的间隔中发生了变化，更新就会失败。这时候就由程序来决定如何处理这个冲突。例如，它可以重新读取新数据来进行更新，又或者它可以将这一情况直接反馈给用户。

## 乐观并发控制

Elasticsearch是分布式的。当文档被创建、更新或者删除时，新版本的文档就会被复制到集群中的其他节点上。Elasticsearch即是同步的又是异步的，也就是说复制的请求被平行发送出去，然后可能会**混乱地**到达目的地。这就需要一种方法能够保证新的数据不会被旧数据所覆盖。

我们在上文提到每当有`索引`、`put`和`删除`的操作时，无论文档有没有变化，它的`_version`都会增加。Elasticsearch使用`_version`来确保所有的改变操作都被正确排序。如果一个旧的版本出现在新版本之后，它就会被忽略掉。

我们可以利用`_version`的优点来确保我们程序修改的数据冲突不会造成数据丢失。我们可以按照我们的想法来指定`_version`的数字。如果数字错误，请求就是失败。

我们来创建一个新的博文:

```js
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```
反馈告诉我们这是一个新建的文档，它的`_version`是`1`。假设我们要编辑它，把这个数据加载到网页表单中，修改完毕然后保存新版本。

首先我们先要得到文档：

```js
GET /website/blog/1
```


返回结果显示`_version`为`1`：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```
现在，我们试着重新索引文档以保存变化，我们这样指定了`version`的数字：

```js
PUT /website/blog/1?version=1 <1>
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```
1. 我们只希望当索引中文档的`_version`是`1`时，更新才生效。

请求成功相应，返回内容告诉我们`_version`已经变成了`2`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```
然而，当我们再执行同样的索引请求，并依旧指定`version=1`时，Elasticsearch就会返回一个`409 Conflict`的响应码，返回内容如下：

```js
{
  "error" : "VersionConflictEngineException[[website][2] [blog][1]:
             version conflict, current [2], provided [1]]",
  "status" : 409
}
```
这里面指出了文档当前的`_version`数字是`2`，而我们要求的数字是`1`。

我们需要做什么取决于我们程序的需求。比如我们可以告知用户已经有其它人修改了这个文档，你应该再保存之前看一下变化。而对于上文提到的`库存量`问题，我们可能需要重新读取一下最新的文档，然后显示新的数据。

所有的有关于更新或者删除文档的API都支持`version`这个参数，有了它你就通过修改你的程序来使用乐观并发控制。


### 使用外部系统的版本

还有一种常见的情况就是我们还是使用其他的数据库来存储数据，而Elasticsearch只是帮我们检索数据。这也就意味着主数据库只要发生的变更，就需要将其拷贝到Elasticsearch中。如果多个进程同时发生，就会产生上文提到的那些并发问题。

如果你的数据库已经存在了版本号码，或者也可以代表版本的`时间戳`。这是你就可以在Elasticsearch的查询字符串后面添加`version_type=external`来使用这些号码。版本号码必须要是大于零小于`9.2e+18`（Java中long的最大正值）的整数。

Elasticsearch在处理外部版本号时会与对内部版本号的处理有些不同。它不再是检查`_version`是否与制定中的数组_相同_,而是检查当前的`_version`是否比指定的数值小。如果请求成功，那么外部的版本号就会被存储到文档中的`_version`中。

外部版本号不仅可以在索引和删除请求时使用，还可以在_创建_时使用。

例如，创建一篇使用外部版本号为`5`的博文，我们可以这样操作：


```js
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

在返回结果中，我们可以发现`_version`是`5`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 5,
  "created":  true
}
```

现在我们更新这个文档，并指定`version`为`10`：

```js
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
```

请求被成功执行并且`version`也变成了`10`：

```js
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```
如果你再次执行这个命令，你会得到之前的错误提示信息，因为你所指定的版本号并没有大于当前Elasticsearch中的版本号。
