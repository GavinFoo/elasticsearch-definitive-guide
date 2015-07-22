### 主从库之间是如何通信的

For explanation purposes, let's imagine that we have a cluster
consisting of 3 nodes. It contains one index called `blogs` which has
two primary shards. Each primary shard has two replicas. Copies of
the same shard are never allocated to the same node, so our cluster
looks something like <<img-distrib>>.

[[img-distrib]]
.A cluster with three nodes and one index
image::images/04-01_index.png["A cluster with three nodes and one index"]

We can send our requests to any node in the cluster. Every node is fully
capable of serving any request.  Every node knows the location of every
document in the cluster and so can forward requests directly to the required
node. In the examples below, we will send all of our requests to `Node 1`,
which we will refer to as  the _requesting node_.

TIP: When sending requests, it is good practice to round-robin through all the
nodes in the cluster, in order to spread the load.

出于解释的目的，让我们想象我们有一个集群有3个节点，这个集群中有2个主分片，每个主分片有2个副本，相同分片的副本是不会放在同一个节点上，所以我们的集群看起来是这样的
我们可以发送我们的请求到集群中的任何一个节点上，每个节点都完全能服务任何请求，每个节点知道集群中的每个文档的位置并且能重定向请求到需要的节点上，在下面的例子中，我们将发送我们的所有的请求到"node 1",这个节点我们叫做“请求节点”
tip 当发送请求的时候，为了加快加载速度，将循环遍历所有的节点
