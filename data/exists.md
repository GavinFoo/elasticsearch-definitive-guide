# 检查文档是否存在

If all you want to do is to check whether a document exists -- you're not
interested in the content at all -- then use the `HEAD` method instead
of the `GET` method. `HEAD` requests don't return a body, just HTTP headers:

```js
curl -i -XHEAD /website/blog/123
```

Elasticsearch will return a `200 OK` status code if the document exists:

```js
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

And a `404 Not Found` if it doesn't exist:

```js
curl -i -XHEAD /website/blog/124
```

```js
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

Of course, just because a document didn't exist when you checked it, doesn't
mean that it won't exist a millisecond later: another process might create the
document in the meantime.
