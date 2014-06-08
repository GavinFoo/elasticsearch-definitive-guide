# 删除一个文档

删除文档的基本模式和之前的基本一样，只不过是需要更换成`DELETE`方法：

```js
DELETE /website/blog/123
```
如果文档存在，那么Elasticsearch就会返回一个`200 OK`的HTTP相应码，返回的结果就会像下面展示的一样。请注意`_version`的数字已经增加了。

```js
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```
如果文档不存在，那么我们就会得到一个`404 Not Found`的响应码，返回的内容就会是这样的：

```js
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```
尽管文档并不存在（`"found"`值为`false`），但是`_version`的数值仍然增加了。这个。这个就是内部管理的一部分，它保证了我们在多个节点间的不同操作都被标记了正确的顺序。

****

正如我在《更新》一章中提到的，删除一个文档也不会立即生效，它只是被标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内容的清理。

****

