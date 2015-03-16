# 安装 JAVA
```js
yum install java-1.7.0-openjdk -y
```

# 安装 Elasticsearch

了解 Elasticsearch 最简单的方法就是去尽情的玩儿它（汗），准备好了我们就开始吧。

安装 Elasticsearch 只有一个要求，就是要安装最新版本的JAVA。你可以到官方网站下载它：[www.java.com](http://www.java.com).

你可以在这里下载到最新版本的 Elasticsearch：
[elasticsearch.org/download](http://www.elasticsearch.org/download/).

```js
curl -L -O http://download.elasticsearch.org/PATH/TO/LATEST/$VERSION.zip
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION
```

提示: 当你安装 Elasticsearch 时，你可以到 [下载页面](http://www.elasticsearch.org/downloads) 选择Debian或者RP安装包。或者你也可以使用官方提供的 [Puppet module](https://github.com/elasticsearch/puppet-elasticsearch) 或者 [Chef cookbook](https://github.com/elasticsearch/cookbook-elasticsearch).



# 安装 Marvel
###### 这是个付费的监控插件 暂时先不翻译
[Marvel](http://www.elasticsearch.com/marvel) is a management and monitoring
tool for Elasticsearch which is free for development use. It comes with an
interactive console called Sense which makes it very easy to talk to
Elasticsearch directly from your browser.

Many of the code examples in this book include a ``View in Sense'' link. When
clicked, it will open up a working example of the code in the Sense console.
You do not have to install Marvel, but it will make this book much more
interactive by allowing you to  experiment with the code samples on your local
Elasticsearch cluster.

Marvel is available as a plugin. To download and install it, run this command
in the Elasticsearch directory:

```js
./bin/plugin -i elasticsearch/marvel/latest
```

You probably don't want Marvel to monitor your local cluster, so you can
disable data collection with this command:

```js
echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml
```

# 运行 Elasticsearch

Elasticsearch 已经蓄势待发，现在你便可以运行它了：

```js
./bin/elasticsearch
```
如果你想让它在后台保持运行的话可以在命令后面再加一个 `-d`

开启后你就可以使用另一个终端窗口来进行测试了:

```js
curl 'http://localhost:9200/?pretty'
```


你应该看到如下提示：

```js
{
   "status": 200,
   "name": "Shrunken Bones",
   "version": {
      "number": "1.4.0",
      "lucene_version": "4.10"
   },
   "tagline": "You Know, for Search"
}
```

这就说明你的 Elasticsearch _集群_ 已经上线运行了，这时我们就可以进行各种实验了。

****
> ###集群和节点

_节点_ 是 Elasticsearch 运行的实例。_集群_ 是一组有着同样`cluster.name`的节点，它们协同工作，互相分享数据，提供了故障转移和扩展的功能。当然一个节点也可以是一个集群。

****
