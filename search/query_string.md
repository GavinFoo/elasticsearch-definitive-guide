# _精简_ 搜索

搜索的API分为两种：其一是通过参数来传递查询的“精简版”_查询语句（query string）_，还有一种是通过JSON来传达丰富的查询的完整版_请求体（request body）_，这种搜索语言被称为查询DSL。


查询语句在行命令中运行点对点查询的时候非常实用。比如我想要查询所有`tweet`类型中，所有`tweet`字段为`"elasticsearch"`的文档：

```js
GET /_all/tweet/_search?q=tweet:elasticsearch
```

下一个查询是想要寻找`name`字段为`"john"`且`tweet`字段为`"mary"`的文档，实际的查询就是：

    +name:john +tweet:mary

但是经过_百分号编码（percent encoding）_处理后，会让它看起来稍显神秘:

```js
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```
前缀`"+"`表示**必须要**满足我们的查询匹配条件，而前缀`"-"`则表示**绝对不能**匹配条件。没有`+`或者`-`的表示可选条件。匹配的越多，文档的相关性就越大。



### 字段`_all`

下面这条简单的搜索将会返回所有包含`"mary"`字符的文档：

```js
GET /_search?q=mary
```

在之前的例子中，我们搜索`tweet`或者`name`中的文字。然而，搜索的结果显示`"mary"`在三个不同的字段中：

* 用户的名字为"Mary"
* 6个"Mary"发送的推文
* 1个"@mary"

那么Elasticsearch是如何找到三个不同字段中的内容呢？

当我们在索引一个文档的时候，Elasticsearch会将所有字段的数值都汇总到一个大的字符串中，并将它索引成一个特殊的字段`_all`：

```js
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}
```
就好像我们已经添加了一个叫做`_all`的字段：

```js
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```
除非指定了字段名，不然查询语句就会搜索字段`_all`。

TIP: 在你刚开始创建程序的时候你可能会经常使用`_all`这个字段。但是慢慢的，你可能就会在请求中指定字段。当字段`_all`已经没有使用价值的时候，那就可以将它关掉。之后的《字段all》一节中将会有介绍


## 更加复杂的查询

再实现一个查询：

* 字段`name`包含`"mary"`或`"john"`
* `date`大于`2014-09-10`
* `_all`字段中包含`"aggregations"`或`"geo"`

```js
+name:(mary john) +date:>2014-09-10 +(aggregations geo)
```

最终处理完的语句可读性可能很差：

```
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```
正如你所看到的，这个_简明_查询语句是出奇的强大。在[查询语句语法](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current//query-dsl-query-string-query.html#query-string-syntax)中，有关于它详细的介绍。借助它我们就可以在开发的时候提高很多效率。

不过，你也会发现简洁带来的易读性差和难以调试，以及它的脆弱：当其中出现`-`, `:`, `/` 或者 `"`时，它就会返回错误提示。

最后要提一句，任何用户都可以通过查询语句来访问臃肿的查询，或许会得到一些私人的信息，或许会通过大量的运算将你的集群压垮！


****
> ### TIP

出于以上原因，我们不建议你将查询语句直接暴露给用户，除非是你信任的可以访问数据与集群的权限用户。

****

与此同时，在生产环境中，我们经常会使用到查询语句。在了解更多关于搜索的知识前，我们先来看一下它是怎样运作的。

