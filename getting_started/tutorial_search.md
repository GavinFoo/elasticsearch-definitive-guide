# 检索文档

现在，我们已经在Elasticsearch中存储了一些数据，我们可以开始根据这个项目的需求进行工作了。第一个需求就是要能搜索每一个员工的数据。

对于Elasticsearch来说，这是非常简单的。我们只需要执行一次HTTP GET请求，然后指出文档的**地址**，也就是索引、类型以及ID即可。通过这三个部分，我们就可以得到原始的JSON文档：

```js
GET /megacorp/employee/1
```

返回的内容包含了这个文档的元数据信息，而John Smith的原始JSON文档也在`_source`字段中出现了：

```js
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

****

我们通过将HTTP后的请求方式由`GET`改变为`PUT`来获取文档，同理，我们也可以将其更换为`DELETE`来删除这个文档，`HEAD`是用来查询这个文档是否存在的。如果你想替换一个已经存在的文档，你只需要使用`PUT`再次发出请求即可。

****

# 简易搜索

`GET`命令真的相当简单，你只需要告诉它你要什么即可。接下来，我们来试一下稍微复杂一点的搜索。

我们首先要完成一个最简单的搜索命令来搜索全部员工：

```js
GET /megacorp/employee/_search
```

你可以发现我们正在使用`megacorp`索引，`employee`类型，但是我们我们并没有指定文档的ID，我们现在使用的是`_search`端口。你可以再返回的`hits`中发现我们录入的三个文档。搜索会默认返回最前的10个数值。


```js
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

注意：反馈值中不仅会告诉你匹配到哪些文档，同时也会把这个文档都会包含到其中：我们需要搜索的用户的所有信息。

接下来，我们将要尝试着实现搜索一下哪些员工的姓氏中包含`Smith`。为了实现这个，我们需要使用一种**轻量**的搜索方法。这种方法经常被称做_查询字符串(query string)_搜索，因为我们通过URL来传递查询的关键字：

```js
GET /megacorp/employee/_search?q=last_name:Smith
```

我们依旧使用`_search`端口，然后可以将参数传入给`q=`。这样我们就可以得到姓Smith的结果：

```js
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

# 使用Query DSL搜索

查询字符串是通过命令语句完成_点对点(ad hoc)_的搜索，但是这也有它的局限性（可参阅《搜索局限性》章节）。Elasticsearch提供了更加丰富灵活的查询语言，它被称作_Query DSL_，通过它你可以完成更加复杂、强大的搜索任务。

DSL (_Domain Specific Language_ 领域特定语言)需要使用JSON作为主体，我们还可以这样查询姓Smith的员工：

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

这个请求会返回同样的结果。你会发现我们在这里没有使用_查询字符串_，而是使用了一个由JSON构成的请求体，其中使用了`match`查询法，随后我们还将会学习到其他的查询类型。


# 更加复杂的搜索

接下来，我们再提高一点儿搜索的难度。我们依旧要寻找出姓Smith的员工，但是我们还将添加一个年龄大于30岁的限定条件。我们的查询语句将会有一些细微的调整来以识别结构化搜索的限定条件 _filter（过滤器）_:

```js
GET /megacorp/employee/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } <1>
                }
            },
            "query" : {
                "match" : {
                    "last_name" : "Smith" <2>
                }
            }
        }
    }
}
```
1. 这一部分的语句是`range`_filter_ ，它可以查询所有超过30岁的数据 -- `gt`代表 **greater than （大于）**。

2. 这一部分我们前一个操作的`match`_query_ 是一样的

先不要被这么多的语句吓到，我们将会在之后带你逐渐了解他们的用法。你现在只需要知道我们添加了一个_filter_，可以在`match`的搜索基础上再来实现区间搜索。现在，我们的只会显示32岁的名为**Jane Smith**的员工了：

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

# 全文搜索

上面的搜索都很简单：名字搜索、通过年龄过滤。接下来我们来学习一下更加复杂的搜索，全文搜索——一项在传统数据库很难实现的功能。
我们将会搜索所有喜欢**rock climbing**的员工:

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```
你会发现我们同样使用了`match`查询来搜索`about`字段中的**rock climbing**。我们会得到两个匹配的文档：

```js
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, <1>
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, <1>
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```
1. 相关评分

通常情况下，Elasticsearch会通过相关性来排列顺序，第一个结果中，John Smith的`about`字段中明确地写到**rock climbing**。而在Jane Smith的`about`字段中，提及到了**rock**，但是并没有提及到**climbing**，所以后者的`_score`就要比前者的低。

这个例子很好地解释了Elasticsearch是如何执行全文搜索的。对于Elasticsearch来说，相关性的感念是很重要的，而这也是它与传统数据库在返回匹配数据时最大的不同之处。


# 段落搜索

能够找出每个字段中的独立单词最然很好，但是有的时候你可能还需要去匹配精确的短语或者_段落_。例如，我们只需要查询到`about`字段只包含**rock climbing**的短语的员工。

为了实现这个效果，我们将对`match`查询变为`match_phrase`查询：

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

这样，系统会没有异议地返回John Smith的文档：

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```

# 高亮我们的搜索

很多程序希望能在搜索结果中_高亮_匹配到的关键字来告诉用户这个文档是_如何_匹配他们的搜索的。在Elasticsearch中找到高亮片段是非常容易的。

让我们回到之前的查询，但是添加一个`highlight`参数：

```js
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```
当我们运行这个查询后，相同的命中结果会被返回，但是我们会得到一个新的名叫`highlight`的部分。在这里包含了`about`字段中的匹配单词，并且会被`<em></em>`HTML字符包裹住：

```js
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" <1>
               ]
            }
         }
      ]
   }
}
```

1. 在原有文本中高亮关键字。
