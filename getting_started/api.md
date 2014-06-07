# 与Elasticsearch通信

如何与Elasticsearch通信要取决于你是否使用JAVA。

## Java API

如果你使用的是JAVA，Elasticsearch内置了两个客户端，你可以在你的代码中使用：

节点客户端::
    节点客户端以一个_无数据节点_的身份加入了一个集群。换句话说，它自身是没有任何数据的，但是他知道什么数据在集群中的哪一个节点上，然后就可以请求转发到正确的节点上并进行连接。

传输客户端::
    更加轻量的传输客户端可以被用来向远程集群发送请求。他并不加入集群本身，而是把请求转发到集群中的节点。

这两个客户端都使用Elasticsearch的_传输_协议，通过**9300端口**与java客户端进行通信。集群中的各个节点也是通过9300端口进行通信。如果这个端口被禁止了，那么你的节点们将不能组成一个集群。

**************************************************
> ###TIP

Java的客户端的版本号必须要与Elasticsearch节点所用的版本号一样，不然他们之间可能无法识别。
**************************************************
更多关于Java API的说明可以在这里找到 [Guide](http://www.elasticsearch.org/guide/).


## 通过HTTP向RESTful API传送json

其他的语言可以通过*9200端口*与Elasticsearch的RESTful API进行通信。事实上，如你所见，你甚至可以使用行命令`curl`来与Elasticsearch通信。

**************************************************

Elasticsearch官方提供了很多种编程语言的客户端，也有和许多社区化软件的集成插件，这些都可以在[Guide](http://www.elasticsearch.org/guide/)里面找到。

**************************************************

向Elasticsearch发出的请求和其他所有的HTTP请求的组成部分是一致的。例如，计算集群中文件的数量，我们就可以使用：

```js
      <1>     <2>                   <3>    <4>
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{  <5>
    "query": {
        "match_all": {}
    }
}
'
```
1. 相应的HTTP_请求方法_或者_变量_: `GET`, `POST`, `PUT`, `HEAD` 或者 `DELETE`。
2. 集群中任意一个节点的访问协议、主机名以及端口。
3. 请求的路径。
4. 任意一个查询后再加上`?pretty` 就可以生成_更加美观_的JSON反馈，以增强可读性。
5. 一个JSON编码的请求主体（如果需要的话）。

Elasticsearch将会返回一个HTTP状态码类似于'200 OK'，以及一个JSON格式的主体（除了单纯的'HEAD'请求），上面的请求会得到下方的JSON主体：

```js
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

在反馈中，我们并没有看见HTTP的头部信息，因为我们没有告知`curl`显示这些内容。如果你想看到头部信息，可以在使用`curl`命令的时候再加上`-i`这个参数：

```js
curl -i -XGET 'localhost:9200/'
```

从现在开始，本书里所有涉及`curl`命令的部分我们都会进行简写，因为主机、端口等信息都是相同的，缩减前的样子:

```js
curl -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```

我们将会简写成这样:

```js
GET /_count
{
    "query": {
        "match_all": {}
    }
}
```


