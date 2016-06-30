# 安装 JAVA
```js
yum install java-1.7.0-openjdk -y
```

# 安装并运行 Elasticsearch

了解 Elasticsearch 最简单的方法就是去使用它，让我们一起开始探索之旅吧！((("Elasticsearch", "installing")))

安装 Elasticsearch 只有一个要求，那就是需要安装最新版本的 Java。而且最好从 Java 官网 http://www.java.com[_www.java.com_] 下载最新版本的 Java ((("Java", "installing")))来安装。

你可以从这里获取到最新版本的Elasticsearch：https://www.elastic.co/downloads/elasticsearch[_elastic.co/downloads/elasticsearch_]。

要安装 Elasticsearch，你需要下载并解压对应运行平台的压缩文件。更多相关信息请参考 Elasticsearch
Reference 中的主题：{ref}/_installation.html[Installation]。

[TIP]
====

当你在生产环境中安装 Elasticsearch 时，你可以选择使用 Debian 或者 RPM 的安装包，地址如下：http://www.elastic.co/downloads/elasticsearch[downloads page]。你也可以使用官方提供的 https://github.com/elasticsearch/puppet-elasticsearch[Puppet module] 或者
https://github.com/elasticsearch/cookbook-elasticsearch[Chef cookbook]。

====

解压完成后，Elasticsearch 就已经准备就绪，等待运行了。((("Elasticsearch", "running")))执行以下命令便可在前台启动它：

[source,sh]
--------------------------------------------------
cd elasticsearch-<version>
./bin/elasticsearch <1> <2>
--------------------------------------------------
<1> 如果你想在后台以守护进程模式运行它，请添加 `-d `参数。
<2> 如果你是在 Windows 中运行 Elasticsearch，只需要运行 `bin\elasticsearch.bat` 即可。

你可以在另一个终端窗口中运行以下命令来验证它是否成功运行：

[source,sh]
--------------------------------------------------
curl 'http://localhost:9200/?pretty'
--------------------------------------------------

TIP: 如果是在 Windows 中运行 Elasticsearch，你可以从 http://curl.haxx.se/download.html[`http://curl.haxx.se/download.html`] 下载 cURL。安装后，遍可以通过 cURL 简单地向 Elasticsearch 提交请求。并且你也可以直接复制粘贴本书中众多的例子，通过 cURL 来运行与试验。

你会看到如下的返回信息：

[source,js]
--------------------------------------------------
{
  "name" : "Tom Foster",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.1.0",
    "build_hash" : "72cd1f1a3eee09505e036106146dc1949dc5dc87",
    "build_timestamp" : "2015-11-18T22:40:03Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
  },
  "tagline" : "You Know, for Search"
}
--------------------------------------------------
// SENSE: 010_Intro/10_Info.json

这说明你已经开启并运行了一个 ELasticsearch 节点，接下来就可以开始各种实验了。_节点（node）_ 是 Elasticsearch 中的一个运行实例。_集群（cluster）_ ((("clusters", "defined")))是一个包含了多个拥有相同 `cluster.name` 节点的分组，这些节点协同工作以共享数据，并且提供了故障转移以及扩展的可能性。（一个节点也可以构成一个集群。）你可以在配置文件 `elasticsearch.yml` 中修改 `cluster.name`，每当节点启动时，这些配置文件就会被加载。更多相关信息可以参考本书最后关于生产部署部分的 <<important-configuration-changes, 重要的配置修改>> 章节。

TIP: 看到例子下方的 View in Sense 链接了吗？ <<sense, 安装 Sense 控制台>> 便可以在自己的 Elasticsearch 集群中运行本书中的例子，并直接查看结果了。

当 Elasticsearch 已经在前台运行时，你可以按下 Ctrl-C 来终止这个进程。

[[sense]]
==== 安装 Sense
Sense 是一个 https://www.elastic.co/guide/en/kibana/current/index.html[Kibana] 程序 ((("Sense console", "Kibana app")))，它的交互式控制台可以帮助你直接通过浏览器向 Elasticsearch 提交请求。
在本书的在线版中，众多的代码示例都包含了 View in Sense 链接。当你点击之后，它将自动在 Sense 控制台中运行这段代码。你并不是一定要安装 Sense，但那将失去很多与本书的互动以及直接在你本地的集群中的实验代码的乐趣。

((("Sense", "downloading and installing"))) 安装并运行 Sense:

. 在 Kibana 的目录中运行以下命令以下载并安装 Sense 程序:
+
[source,sh]
--------------------------------------------------
./bin/kibana plugin --install elastic/sense <1>
--------------------------------------------------
<1> Windows: `bin\kibana.bat plugin --install elastic/sense`.
+
NOTE: 如果需要 https://www.elastic.co/guide/en/sense/current/installing.html#manual_download[离线安装 Sense]，你可以从这里直接下载： https://download.elastic.co/elastic/sense/sense-latest.tar.gz。

. 运行 Kibana.
+
[source,sh]
--------------------------------------------------
./bin/kibana <1>
--------------------------------------------------
<1> Windows: `bin\kibana.bat`.

. 在浏览器中访问 `http://localhost:5601/app/sense` 就可以使用 Sense 了。
