#You Know, for Search...

Elasticsearch 是一个建立在全文搜索引擎框架 [Apache Lucene(TM)](https://lucene.apache.org/core/) 基础上的开源搜索引擎， 无论是开源软件还是私有软件，Lucene 都毫无疑问是当今最先进、性能最高和功能最全的搜索引擎框架。

但是 Lucene 只是一个框架，要充分使用它的功能，你需要使用 JAVA 作为开发语言，并且在你的程序中集成 Lucene。更糟的是，你需要深入了解相关知识才能明白它是如何运行的，Lucene **确实**非常复杂。

Elasticsearch 也是使用 Java 编写的，并且采用了 Lucene 来实现索引与搜索的功能。而且，在你使用它做全文搜索时，只需要使用简单流畅的 RESTful API 即可，并不需要了解 Lucene 背后复杂的的运行原理。

当然 Elasticsearch 还有很多地方超越了 Lucene，它**不仅**可以实现全文搜索功能，还可以完成以下工作:

* 分布式实时文档存储，并将_每一个_字段都编入索引，使其可以被搜索。
* 分布式实时分析与搜索引擎。
* 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

这么多的功能被集成到一台服务器上，你可以轻松地通过客户端或者任何你喜欢的程序语言与 Elasticsearch 的 RESTful API 进行通信。

Elasticsearch 的上手是非常简单的。它附带了很多非常合理的默认值，这让初学者很好地避免一上手就要面对复杂的理论。安装完成就可以马上_投入使用_了。不需要了解很多，你就能变得非常有生产力。

随着学习的深入，你还可以使用 Elasticsearch 更多高级的功能。整个引擎可以很灵活地进行配置。你可以根据自身需求来定制属于你自己的 Elasticsearch。

你可以随意下载、使用、修改 Elasticsearch。它采用 [Apache 2 license](http://www.apache.org/licenses/LICENSE-2.0.html) 进行授权，这是当前最灵活的开源授权方式。源代码托管在 Github 之上：[github.com/elastic/elasticsearch](https://github.com/elastic/elasticsearch)
Elasticsearch 在 Apache 2 license下许可使用，可以免费下载、使用和修改。如果你愿意加入我们非常优秀的贡献者社区，请参考：[Elasticsearch 贡献](https://github.com/elastic/elasticsearch/blob/master/CONTRIBUTING.md)。

如果你对 Elasticsearch 的功能、插件、SDK 等任何方面有疑问，都可以在这里提出 [discuss.elastic.co](https://discuss.elastic.co)，[elastic 中文社区](http://elasticsearch.cn/)。

####回忆时光
***************************************

多年以前，有个叫 Shay Banon 的失业开发者跟随他的新婚妻子来到了伦敦，因为他妻子将在那里学习厨艺。Shay 在寻找工作的同时，开始研究还是早期版本的 Lucene，并打算为她妻子制作一个烹饪菜谱搜索引擎。

直接基于 Lucene 工作会比较困难，所以 Shay 开始实现一个 Lucene 之上的抽象层，这样 Java 程序员可以很方便的为他们的应用程序添加搜索功能。于是他发布了自己的第一个开源项目 “Compass”。

后来 Shay 找到了一份工作，这份工作主要围绕在高性能与分布式内存数据存储的环境中。高性能、实时、分布式搜索引擎是必不可少的需求。因此他决定重写 Compass 库，使其成为一个独立的服务器，这便是 Elasticsearch。

2010年的2月份，第一个公开版本发布了。从此之后，Elasticsearch 成为 Github 上最受欢迎的项目之一，超过300个代码贡献者。一家关于 Elasticsearch 的公司就此成立，它们不仅提供商业支持还在进行新功能的开发，但是 Elasticsearch 一定会永远向大众开放，永远开源给所有需要的人们。

噢，对了，Shay 的妻子还在等待着她的菜谱搜索引擎。

***************************************
