# 批量更高效

与`mget`能同时允许帮助我们获取多个文档相同，`bulk` API可以帮助我们同时完成执行多个请求，比如：`create`，`index`, `update`以及`delete`。当你在处理类似于log等海量数据的时候，你就可以一下处理成百上千的请求，这个操作将会极大提高效率。

`bulk`的请求主体的格式稍微有些不同：

```js
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```
这种格式就类似于一个用`"\n"`字符来连接的单行json一样。下面是两点注意事项：

* 每一行都结尾处都必须有换行字符`"\n"`，**最后一行也要有**。这些标记可以有效地分隔每行。


* 这些行里不能包含非转义字符，以免干扰数据的分析 — — 这也意味着JSON**不能**是pretty-printed样式。

**************************************************
> ###TIP

在《bulk格式》一章中，我们将解释为何`bulk` API要使用这种格式。

**************************************************

_action/metadata_ 行指定了将要在**哪个文档**中执行**什么操作**。

其中_action_必须是`index`, `create`, `update`或者`delete`。_metadata_ 需要指明需要被操作文档的`_index`, `_type`以及`_id`，例如删除命令就可以这样填写：

```js
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```
在你进行`index`以及`create`操作时，_request body_ 行必须要包含文档的`_source`数据——也就是文档的所有内容。

同样，在执行`update` API: `doc`, `upsert`,`script`的时候，也需要包含相关数据。而在删除的时候就不需要_request body_行。

```js
{ "create":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
```

如果没有指定`_id`，那么系统就会自动生成一个ID：

```js
{ "index": { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
```

完成以上所有请求的`bulk`如下：

```js
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} <1>
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } <2>
```

1. 注意`delete`操作是如何处理_request body_的,你可以在它之后直接执行新的操作。

2. 请记住最后有换行符

Elasticsearch会返回含有`items`的列表、它的顺序和我们请求的顺序是相同的：


```js
{
   "took": 4,
   "errors": false, <1>
   "items": [
      {  "delete": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 2,
            "status":   200,
            "found":    true
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 3,
            "status":   201
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "EiwfApScQiiy7TIKFxRCTw",
            "_version": 1,
            "status":   201
      }},
      {  "update": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 4,
            "status":   200
      }}
   ]
}}
```
1. 所有的请求都被成功执行。

每一个子请求都会被单独执行，所以一旦有一个子请求失败了，并不会影响到其他请求的成功执行。如果一旦出现失败的请求，`error`就会变为`true`，详细的错误信息也会出现在返回内容的下方：


```js
POST /_bulk
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
```
请求中的`create`操作失败，因为`123`已经存在，但是之后针对文档`123`的`index`操作依旧被成功执行：

```js
{
   "took": 3,
   "errors": true, <1>
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, <2>
            "error":    "DocumentAlreadyExistsException <3>
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 <4>
      }}
   ]
}
```
1. 至少有一个请求错误发生。
2. 这条请求的状态码为`409 CONFLICT`。
3. 错误信息解释了导致错误的原因。
4. 第二条请求的状态码为`200 OK`。

这也更好地解释了`bulk`请求是独立的，每一条的失败与否 都不会影响到其他的请求。

### 能省就省

或许你在批量导入大量的数据到相同的`index`以及`type`中。每次都去指定每个文档的metadata是完全没有必要的。在`mget` API中，`bulk`请求可以在URL中声明`/_index` 或者`/_index/_type`：

```js
POST /website/_bulk
{ "index": { "_type": "log" }}
{ "event": "User logged in" }
```
你依旧可以在metadata行中使用`_index`以及`_type`来重写数据，未声明的将会使用URL中的配置作为默认值：

```js
POST /website/log/_bulk
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
```

### 最大有多大？

整个数据将会被处理它的节点载入内存中，所以如果请求量很大的话，留给其他请求的内存空间将会很少。`bulk`应该有一个最佳的限度。超过这个限制后，性能不但不会提升反而可能会造成宕机。

最佳的容量并不是一个确定的数值，它取决于你的硬件，你的文档大小以及复杂性，你的索引以及搜索的负载。幸运的是，这个_平衡点_ 很容易确定：

试着去批量索引越来越多的文档。当性能开始下降的时候，就说明你的数据量太大了。一般比较好初始数量级是1000到5000个文档，或者你的文档很大，你就可以试着减小队列。
有的时候看看批量请求的物理大小是很有帮助的。1000个1KB的文档和1000个1MB的文档的差距将会是天差地别的。比较好的初始批量容量是5-15MB。
