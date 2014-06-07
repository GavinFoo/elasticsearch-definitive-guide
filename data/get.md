# 搜索文档

要从Elasticsearch中获取文档，我们需要使用同样的`_index`，`_type`以及 `_id`但是不同的HTTP变量`GET`：
```js
GET /website/blog/123?pretty
```
返回结果包含了之前提到的内容，以及一个新的字段`_source`，它包含我们在最初创建索引时的原始JSON文档。

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
      "date":  "2014/01/01"
  }
}
```

****
>#### `pretty`

在任意的查询字符串中添加`pretty`参数，类似上面的请求，Elasticsearch就可以得到_优美打印_的更加易于识别的JSON结果。`_source`字段不会执行优美打印，它的样子取决于我们录入的样子。

****

GET请求的返回结果中包含`{"found": true}`。这意味着这篇文档确实被找到了。如果我们请求了一个不存在的文档，我们依然会得到JSON反馈，只是`found`的值会变为`false`。

同样，HTTP返回码也会由`'200 OK'`变为`'404 Not Found'`。我们可以在`curl`后添加`-i`，这样你就能得到反馈头文件：

```js
curl -i -XGET /website/blog/124?pretty
```

反馈结果就会是这个样子：

```js
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=UTF-8
Content-Length: 83

{
  "_index" : "website",
  "_type" :  "blog",
  "_id" :    "124",
  "found" :  false
}
```

### 检索文档中的一部分

通常，`GET`请求会将整个文档放入`_source`字段中一并返回。但是可能你只需要`title`字段。你可以使用`_source`得到指定字段。如果需要多个字段你可以使用逗号分隔：

```js
GET /website/blog/123?_source=title,text
```
现在`_source`字段中就只会显示你指定的字段：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "exists" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
或者你只想得到`_source`字段而不要其他的元数据，你可以这样请求：

```js
GET /website/blog/123/_source
```
这样结果就只返回:

```js
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```
