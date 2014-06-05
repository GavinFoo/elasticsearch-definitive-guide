# 统计

最后，我们还有一个需求需要完成：可以让老板在职工目录中进行统计。Elasticsearch把这项功能称作_汇总 (aggregations)_，通过这个功能，我们可以针对你的数据进行复杂的统计。这个功能有些类似于SQL中的`GROUP BY`，但是要比它更加强大。

例如，让我们找一下员工中最受欢迎的兴趣是什么：

```js
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

请忽略语法，让我们先来看一下结果：

```js
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```
我们可以发现有两个员工喜欢音乐，还有一个喜欢森林，还有一个喜欢运动。这些数据并没有被预先计算好，它们是在文档被查询的同时实时计算得出的。如果你想要查询姓Smith的员工的兴趣汇总情况，你就可以执行如下查询：

```js
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```
这样，`all_interests`的统计结果就只会包含满足查询的文档了：

```js
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```
汇总还允许多个层面的统计。比如我们还可以统计每一个兴趣下的平均年龄：

```js
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```
虽然这次返回的汇总结果变得更加复杂了，但是它依旧很容易理解：

```js
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```
在这个丰富的结果中，我们不但可以看到兴趣的统计数据，还能针对不同的兴趣来分析喜欢这个兴趣的`平均年龄`。

即使你现在还不能很好地理解语法，但是相信你还是能发现，用这个功能来实现如此复杂的统计工作是这样的简单。你的极限取决于你存入了什么样的数据哟！
