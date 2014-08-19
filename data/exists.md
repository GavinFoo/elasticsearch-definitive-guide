# 检查文档是否存在

如果确实想检查一下文档是否存在，你可以试用`HEAD`来替代`GET`方法，这样就是会返回HTTP头文件：

```js
curl -i -XHEAD /website/blog/123
```
如果文档存在，Elasticsearch将会返回`200 OK`的状态码：

```js
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
如果不存在将会返回`404 Not Found`状态码：

```js
curl -i -XHEAD /website/blog/124
```

```js
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

当然，这个反馈只代表了你查询的那一刻文档不存在，但是不代表几毫秒后它是否存在，很可能与此同时，另一个进程正在创建文档。
