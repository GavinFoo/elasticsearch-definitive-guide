# 创建一个文档

我们是如何确定当我们索引一个文档时，我们是创建了一个新的文档还是覆盖了一个已经存在的文档呢？


请牢记`_index`,`_type`以及`_id`组成了唯一的文档标记，所以为了确定我们创建的内容是全新的的最简单的方法就是使用`POST`方法，让Elasticsearch自动创建不同的`_id`：

```js
POST /website/blog/
{ ... }
```

然而，我们可能已经决定好了`_id`，所以我们需要告诉Elasticsearch只有当同`_index`，`_type`以及`_id`的文档不存在时才接受我们的请求。实现这个目的有两种方法，他们实质上是一样的，你可以选择你认为方便的那种：

第一种是在查询中添加`op_type`参数：

```js
PUT /website/blog/123?op_type=create
{ ... }
```

或者在请求最后添加 `/_create`:

```js
PUT /website/blog/123/_create
{ ... }
```

如果成功创建了新的文档，Elasticsearch将会返回常见的元数据以及`201 Created`的HTTP反馈码。

而如果存在同名文件，Elasticsearch将会返回一个`409 Conflict`的HTTP反馈码，以及如下方的错误信息：

```js
{
  "error" : "DocumentAlreadyExistsException[[website][4] [blog][123]:
             document already exists]",
  "status" : 409
}
```

