# 空集群

If we start a single node, with no data and no indices, our cluster looks like
![img-cluster](/images/02-01_cluster.png "A cluster with one empty node")


A _node_ is a running instance of Elasticsearch, while a _cluster_ consists of
one or more nodes with the same `cluster.name` that are working together to
share their data and workload. As nodes are added to or removed from the
cluster, the cluster reorganizes itself to spread the data evenly.

One node in the cluster is elected to be the _master_ node, which is in charge
of managing cluster-wide changes like creating or deleting an index, or adding
or removing a node from the cluster.  The master node does not need to be
involved in document level changes or searches, which means that having just
one master node will not become a bottleneck as traffic grows. Any node can
become the master. Our example cluster has only one node, so it performs the
master role.

As users, we can talk to *any node in the cluster*, including the master node.
Every node knows where each document lives and can forward our request
directly to the nodes that hold the data we are interested in. Whichever node
we talk to manages the process of gathering the response from the node or
nodes holding the data and returning the final response to the client. It is
all managed transparently by Elasticsearch.

如果我们启用一个既没有数据，也没有索引的单一节点，那我们的集群看起来就像是这样
![img-cluster](/images/02-01_cluster.png "A cluster with one empty node")

一个_节点_是Elasticsearch的一个运行中的实例，而一个_集群_则包含一个或多个具有相同`cluster.name`的节点，它们协同工作来共享数据，共同分担工作负荷。由于节点是从属集群的，集群会自我重组来均匀地分发数据。

集群中的一个节点会被选为_master_节点，它将负责管理集群范畴的变更，例如创建或删除索引，添加节点到集群或从集群删除节点。master节点无需参与文档层面的变更和搜索，这意味着仅有一个master节点并不会因流量增长而成为瓶颈。任意一个节点都可以成为master节点。我们例举的集群只有一个节点，因此它会扮演master节点的角色。

作为用户，我们可以访问包括master节点在内的*集群中的任一节点*。每个节点都知道各个文档的位置，并能够将我们的请求直接转发到拥有我们想要的数据的节点。无论我们访问的是哪个节点，它都会控制从拥有数据的节点收集响应的过程，并返回给客户端最终的结果。这一切都是由Elasticsearch透明管理的。
