# 获取多个文档

尽管Elasticsearch已经很快了，但是它依旧可以更快。你可以将多个请求合并到一个请求中以节省网络开销。如果你需要从Elasticsearch中获取多个文档，你可以使用_multi-get_ 或者 `mget` API来取代一篇又一篇文档的获取。

`mget`API需要一个`docs`数组，每一个元素包含你想要的文档的`_index`, `_type`以及`_id`。你也可以指定`_source`参数来设定你所需要的字段：

```js
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```
返回值包含了一个`docs`数组，这个数组以请求中指定的顺序每个文档包含一个响应。每一个响应都和独立的`get`请求返回的响应相同：


```js
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```
如果你所需要的文档都在同一个`_index`或者同一个`_type`中，你就可以在URL中指定一个默认的`/_index`或是`/_index/_type`。

你也可以在单独的请求中重写这个参数：

```js
GET /website/blog/_mget
{
   "docs" : [
      { "_id" : 2 },
      { "_type" : "pageviews", "_id" :   1 }
   ]
}
```
事实上，如果所有的文档拥有相同的`_index` 以及 `_type`，直接在请求中添加`ids`的数组即可：

```js
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```
请注意，我们所请求的第二篇文档不存在，这是就会返回如下内容：

```js
{
  "docs" : [
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "2",
      "_version" : 10,
      "found" :    true,
      "_source" : {
        "title":   "My first external blog entry",
        "text":    "This is a piece of cake..."
      }
    },
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "found" :    false  <1>
    }
  ]
}
```
1. 文档没有被找到。

当第二篇文档没有被找到的时候也不会影响到其它文档的获取结果。每一个文档都会被独立展示。

注意：上方请求的HTTP状态码依旧是`200`，尽管有个文档没有找到。事实上，即使**所有的**文档都没有被找到，响应码也依旧是`200`。这是因为`mget`这个请求本身已经成功完成。要确定独立的文档是否被成功找到，你需要检查`found`标识。
