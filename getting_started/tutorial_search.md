# 检索文档

现在，我们已经在Elasticsearch中存储了一些数据，我们可以开始根据这个项目的需求进行工作了。第一个需求就是要能搜索每一个员工的数据。

对于Elasticsearch来说，这是非常简单的。我们只需要执行一次HTTP GET请求，然后指出文档的**地址**，也就是索引、类型以及ID即可。通过这三个部分，我们就可以得到原始的JSON文档：

```
GET /megacorp/employee/1
```

返回的内容包含了这个文档的元数据信息，而John Smith的原始JSON文档也在`_source`字段中出现了：

```
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

```
GET /megacorp/employee/_search
```

你可以发现我们正在使用`megacorp`索引，`employee`类型，但是我们我们并没有指定文档的ID，我们现在使用的是`_search`端口。你可以再返回的`hits`中发现我们录入的三个文档。搜索会默认返回最前的10个数值。


```
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

注意：反馈值中不仅会到诉你匹配到那些文档，同时也会把这个文档都会包含到其中：我们需要搜索的用户的所有信息。

接下来，我们将要尝试着实现搜索一下哪些员工的姓氏中包含`Smith`。为了实现这个，我们需要使用一种**轻量**的搜索方法。这种方法经常被称做_查询字符串(query string)_搜索，因为我们通过URL来传递查询的关键字：

```
GET /megacorp/employee/_search?q=last_name:Smith
```

我们依旧使用`_search`端口，然后可以将参数传入给`q=`。这样我们就可以得到姓Smith的结果：

```
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

=== Search with Query DSL

Query-string search is handy for _ad hoc_ searches from the command line, but
it has its limitations (see <<search-lite>>). Elasticsearch provides a rich,
flexible, query language called the _Query DSL_, which allows us to build
much more complicated, robust queries.

The DSL (_Domain Specific Language_) is specified using a JSON request body.
We can represent the previous search for all Smith's like so:


[source,js]
--------------------------------------------------
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "smith"
        }
    }
}
--------------------------------------------------
// SENSE: 010_Intro/30_Simple_search.json

This will return the same results as the previous query.  You can see that a
number of things have changed.  For one, we are no longer using _query string_
parameters, but instead a request body.  This request body is built with JSON,
and uses a `match` query (one of several types of queries, which we will learn
about later).

=== More complicated searches

Let's make the search a little more complicated.  We still want to find all
employees with a last name of ``Smith'', but  we only want employees who are
older than 30.  Our query will change a little to accommodate a _filter_,
which allows us to execute structured searches efficiently:

[source,js]
--------------------------------------------------
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
                    "last_name" : "smith" <2>
                }
            }
        }
    }
}
--------------------------------------------------
// SENSE: 010_Intro/30_Query_DSL.json

<1> This portion of the query is a `range` _filter_, which will find all ages
    older than 30 -- `gt` stands for ``greater than''.
<2> This portion of the query is the same `match` _query_ that we used before.

Don't worry about the syntax too much for now, we will cover it in great
detail later on.  Just recognize that we've added a _filter_ which performs a
range search, and reused the same `match` query as before.  Now our results
only show one employee who happens to be 32 and is named ``Jane Smith'':

[source,js]
--------------------------------------------------
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
--------------------------------------------------

=== Full-text search

The searches so far have been simple:  single names, filtering by age. Let's
try a more advanced, full-text search -- a task which traditional databases
would really struggle with.

We are going to search for all employees who enjoy ``rock climbing'':

[source,js]
--------------------------------------------------
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
--------------------------------------------------
// SENSE: 010_Intro/30_Query_DSL.json

You can see that we use the same `match` query as before to search the `about`
field for ``rock climbing''. We get back two matching documents:

[source,js]
--------------------------------------------------
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
--------------------------------------------------
<1> The relevance scores.

By default, Elasticsearch sorts matching results by their relevance score,
that is: by how well each document matched the query.  The first and highest
scoring result is obvious: John Smith's `about` field clearly says ``rock
climbing'' in it.

But why did Jane Smith, come back as a result?  The reason her document was
returned is because the word ``rock'' was mentioned in her `about` field.
Because only ``rock'' was mentioned, and not ``climbing'', her `_score` is
lower than John's.

This is a good example of how Elasticsearch can search *within* full text
fields and return the most relevant results first. This concept of _relevance_
is important to Elasticsearch, and is a concept that is completely foreign to
traditional relational databases where a record either matches or it doesn't.

=== Phrase search

Finding individual words in a field is all well and good, but sometimes you
want to match exact sequences of words or _phrases_. For instance, we could
perform a query that will only match  employees that contain both  ``rock''
_and_ ``climbing'' _and_ where the words are next to each other in the phrase
``rock climbing''.

To do this, we use a slight variation of the `match` query called the
`match_phrase` query:

[source,js]
--------------------------------------------------
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
--------------------------------------------------
// SENSE: 010_Intro/30_Query_DSL.json

Which, to no surprise, returns only John Smith's document:

[source,js]
--------------------------------------------------
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
--------------------------------------------------

[[highlighting-intro]]
=== Highlighting our searches

Many applications like to _highlight_ snippets of text from each search result
so that the user can see *why* the document matched their query.  Retrieving
highlighted fragments is very easy in Elasticsearch.

Let's rerun our previous query, but add a new `highlight` parameter:

[source,js]
--------------------------------------------------
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
--------------------------------------------------
// SENSE: 010_Intro/30_Query_DSL.json

When we run this query, the same hit is returned as before, but now we get a
new section in the response called `highlight`.  This contains a snippet of
text from the `about` field with the matching words wrapped in `<em></em>`
HTML tags:

[source,js]
--------------------------------------------------
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
--------------------------------------------------

<1> The highlighted fragment from the original text.
