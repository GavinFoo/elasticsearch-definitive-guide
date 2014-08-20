# 索引一个文档

文档通过`索引`API被_索引_——存储并使其可搜索。但是最开始我们需要决定我们将文档存储在哪里。正如之前提到的，一篇文档通过`_index`, `_type`以及`_id`来确定它的唯一性。我们可以自己提供一个`_id`，或者也使用`index`API 帮我们生成一个。


## 使用自己的ID

如果你的文档拥有天然的标示符（例如`user_account`字段或者文档中其他的标识值），这时你就可以提供你自己的`_id`，这样使用`index`API：

```js
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```
几个例子。如果我们的索引叫做`"website"`，我们的类型叫做 `"blog"`，然后我们选择`"123"`作为ID的编号。这时，请求就是这样的：
```js
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
```

Elasticsearch回馈内容：

```js
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "123",
   "_version":  1,
   "created":   true
}
```
这个回馈意味着我们的索引请求已经被成功创建，其中还包含了`_index`, `_type`以及`_id`的元数据，以及一个新的元素`_version`。

在Elasticsearch中，每一个文档都有一个版本号码。每当文档产生变化时（包括删除），`_version`就会增大。在《版本控制》中，我们将会详细讲解如何使用`_version`的数字来确认你的程序不会随意替换掉不想覆盖的数据。

### 自增ID

如果我们的数据中没有天然的标示符，我们可以让Elasticsearch为我们自动生成一个。请求的结构发生了变化：我们把`PUT`——“把文档存储在这个地址中”变量变成了`POST`——“把文档存储在这个**地址下**”。

这样一来，请求中就只包含 `_index`和`_type`了：

```js
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

这次的反馈和之前基本一样，只有`_id`改成了系统生成的自增值:

```
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "wM0OSFhDQXGZAWDf0-drSA",
   "_version":  1,
   "created":   true
}
```
自生成ID是由22个字母组成的，安全
_universally unique identifiers_ 或者被称为[UUIDs](http://baike.baidu.com/view/1052579.htm?fr=aladdin)。




