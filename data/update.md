# 更新整个文档

在Documents中的文档是不可改变的。所以如果我们需要改变已经存在的文档，我们可以使用《索引》中提到的`index`API来_重新索引_或者替换掉它：

```js
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```
在反馈中，我们可以发现Elasticsearch已经将`_version`数值增加了：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false <1>
}
```
1. `created`被标记为 `false`是因为在同索引、同类型下已经存在同ID的文档。

在内部，Elasticsearch已经将旧文档标记为删除并且添加了新的文档。旧的文档并不会立即消失，但是你也无法访问他。Elasticsearch会在你继续添加更多数据的时候在后台清理已经删除的文件。

在本章的后面，我们将会在《局部更新》中介绍最新更新的API。这个API允许你修改局部，但是原理和下方的完全一样：

1. 从旧的文档中检索JSON
2. 修改它
3. 删除修的文档
4. 索引一个新的文档

唯一不同的是，使用了`update`API你就不需要使用`get`然后再操作`index`请求了。
