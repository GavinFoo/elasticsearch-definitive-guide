# 更新文档中的一部分

在《更新》一章中，我们讲到了要是想更新一个文档，那么就需要去取回数据，更改数据然后将整个文档进行重新索引。当然，你还可以通过使用`更新`API来做部分更新，比如增加一个计数器。

正如我们提到的，文档不能被修改，它们只能被替换掉。`更新`API也**必须**遵循这一法则。从表边看来，貌似是文档被替换了。对内而言，它必须按照_找回-修改-索引_的流程来进行操作与管理。不同之处在于这个流程是在一个片(shard) 中完成的，因此可以节省多个请求所带来的网络开销。除了节省了步骤，同时我们也能减少多个进程造成冲突的可能性。

使用`更新`请求最简单的一种用途就是添加新数据。新的数据会被合并到现有数据中，而如果存在相同的字段，就会被新的数据所替换。例如我们可以为我们的博客添加`tags`和`views`字段：

```js
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

如果请求成功，我们就会收到一个类似于`索引`时返回的内容:

```js
{
   "_index" :   "website",
   "_id" :      "1",
   "_type" :    "blog",
   "_version" : 3
}
```

再次取回数据，你可以在`_source`中看到更新的结果：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  3,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags": [ "testing" ], <1>
      "views":  0 <1>
   }
}
```

1. 新的数据已经添加到了字段`_source`中。


#### 使用脚本进行更新

****

我们将会在《脚本》一章中学习更详细的内容，我们现在只需要了解一些在Elasticsearch中使用API无法直接完成的自定义行为。默认的脚本语言叫做MVEL，但是Elasticsearch也支持JavaScript, Groovy 以及 Python。

MVEL是一个简单高效的JAVA基础动态脚本语言，它的语法类似于Javascript。你可以在[Elasticsearch scripting docs](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html) 以及 [MVEL website](http://mvel.codehaus.org/Getting+Started+for+2.0)了解更多关于MVEL的信息。

****

脚本语言可以在`更新`API中被用来修改`_source`中的内容，而它在脚本中被称为`ctx._source`。例如，我们可以使用脚本来增加博文中`views`的数字：
```js
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```
我们同样可以使用脚本在`tags`数组中添加新的tag。在这个例子中，我们把新的tag生命为一个变量，而不是将他写死在脚本中。这样Elasticsearch就可以重新使用这个脚本进行tag的添加，而不用再次重新编写脚本了：


```js
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}
```

获取文档，后两项发生了变化：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "1",
   "_version":  5,
   "found":     true,
   "_source": {
      "title":  "My first blog entry",
      "text":   "Starting to get the hang of this...",
      "tags":  ["testing", "search"], <1>
      "views":  1 <2>
   }
}
```
1. `tags`数组中出现了`search`。
2. `views`字段增加了。

我们甚至可以使用`ctx.op`来根据内容选择是否删除一个文档：

```js
POST /website/blog/1/_update
{
   "script" : "ctx.op = ctx._source.views == count ? 'delete' : 'none'",
    "params" : {
        "count": 1
    }
}
```

### 更新一篇可能不存在的文档

想象一下，我们可能需要在Elasticsearch中存储一个页面计数器。每次用户访问这个页面，我们就增加一下当前页面的计数器。但是如果这是个新的页面，我饿们不能确保这个计数器已经存在。如果我们试着去更新一个不存在的文档，更新操作就会失败。

为了防止上述情况的发生，我们可以使用`upsert`参数来设定文档不存在时，它应该被创建的内同：

```js
POST /website/pageviews/1/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```
首次运行这个请求时，`upsert`的内容会被索引成新的文档，它将`views`字段初始化为`1`。当之后再请求时，文档已经存在，所以`脚本`更新就会被执行，`views`计数器就会增加。


### 更新和冲突

在本节的开篇我们提到了当_取回_与_重新索引_两个步骤间的时间越少，发生改变冲突的可能性就越小。但它并不能被完全消除，在`更新`的过程中还能可能存在另一个进程进行重新索引的可能性。

为了避免丢失数据，`更新`API会在_获取_步骤中获取当前文档中的`_version`，然后将其传递给_重新索引_步骤中的`索引`请求。如果其他的进程在这两部之间修改了这个文档，那么`_version`就会不同，这样更新就会失败。

对于很多的局部更新来说，文档有没有发生变化实际上是不重要的。例如，两个进行都要增加页面浏览的计数器，谁先谁后其实并不重要 —— 发生冲突是之需要重新来过即可。

你可以通过设定`retry_on_conflict`参数来设置自动完成这项请求的次数，它的默认值是`0`。

```js
POST /website/pageviews/1/_update?retry_on_conflict=5 <1>
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 0
   }
}
```
1. 失败前重新尝试5次

这个参数非常适用于类似于增加技术起这种无关顺序的请求，但是还有些情况就属于顺序**很重要**的。例如上一节提到的情况，你可以参考乐观并发控制以及悲观并发控制来设定文档的版本号。
