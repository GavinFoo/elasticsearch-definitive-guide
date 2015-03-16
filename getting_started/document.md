# 面向文档

程序中的对象很少是单纯的键值与数值的列表。更多的时候它拥有一个复杂的结构，比如包含了日期、地理位置、对象、数组等。

迟早你会把这些对象存储在数据库中。你会试图将这些丰富而又庞大的数据都放到一个由行与列组成的关系数据库中，然后你不得不根据每个字段的格式来调整数据，然后每次重建它你都要检索一遍数据。

Elasticsearch 是 _面向文档型数据库_，这意味着它存储的是整个对象或者 _文档_，它不但会存储它们，还会为他们建立**索引**，这样你就可以搜索他们了。你可以在 Elasticsearch 中索引、搜索、排序和过滤这些文档。不需要成行成列的数据。这将会是完全不同的一种面对数据的思考方式，这也是为什么 Elasticsearch 可以执行复杂的全文搜索的原因。


# JSON

Elasticsearch使用 [_JSON_](http://baike.baidu.com/view/136475.htm?fr=aladdin) (或称作JavaScript
Object Notation ) 作为文档序列化的格式。JSON 已经被大多数语言支持，也成为 NoSQL 领域的一个标准格式。它简单、简洁、易于阅读。

把这个 JSON 想象成一个用户对象:

```js
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "about": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01",
}
```

虽然 `user` 这个对象非常复杂，但是它的结构和含义都被保留到 JSON 中了。在 Elasticsearch 中，将对象转换为 JSON 并作为索引要比在表结构中做相同的事情简单多了。

***
>###将你的数据转换为 JSON

几乎所有的语言都有将任意数据转换、机构化成 JSON，或者将对象转换为JSON的模块。查看 `serialization` 以及 `marshalling` 两个 JSON 模块。[The official Elasticsearch clients](http://www.elasticsearch.org/guide) 也可以帮你自动结构化 JSON。

***
