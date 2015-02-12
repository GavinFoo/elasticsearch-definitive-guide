# 映射与统计

当我们在进行搜索的事情，我们会发现有一些奇怪的事情。比如有一些内容似乎是被打破了：在我们的索引中有12条推文，中有一个包含了`2014-09-15`这个日期，但是看看下面的查询结果中的总数量：

``` js
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results !
GET /_search?q=date:2014-09-15   # 1  result
GET /_search?q=date:2014         # 0  results !
```
为什么我们使用字段`_all`搜索全年就会返回所有推文，而使用字段`date`搜索年份却没有结果呢？为什么使用两者所得到的结果是不同的？

推测大概是因为我们的数据在`_all`和`date`在索引时没有被相同处理。我们来看看Elasticsearch是如何处理我们的文档结构的。我们可以对`gb`的`tweet`使用_mapping_请求：

```js
GET /gb/_mapping/tweet
```
我们得到：

```js
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```
Elasticsearch会根据系统自动判断字段类型并生成一个映射。返回结果告诉我们`date`字段被识别成了`date`类型。`_all`没有出现是因为他是默认字段，但是我们知道字段`_all`实际上是`string`类型的。

所以类型为`date`的字段和类型为`string`的字段的索引方式是不同的。

So fields of type `date` and fields of type `string` are indexed differently,
and can thus be searched differently.  That's not entirely surprising.
You might expect that each of the core data types -- strings, numbers, booleans
and dates -- might be indexed slightly differently. And this is true:
there are slight differences.

But by far the biggest difference is actually between fields that represent
_exact values_ (which can include `string` fields) and fields that
represent _full text_. This distinction is really important -- it's the thing
that separates a search engine from all other databases.

