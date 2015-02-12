# 增加故障转移

在单一节点上运行意味着有单点故障的风险，没有数据冗余备份。幸运的是，我们可以启用另一个节点来保护我们的数据。

****
> ### 启动第二个节点

为了测试在增加第二个节点后发生了什么，你可以使用与第一个节点相同的方式启动第二个节点（你可以参考 入门-》安装-》运行Elasticsearch 一章），而且在同一个目录——多个节点可以分享同一个目录。

只要第二个节点与第一个节点的`cluster.name`相同（参见`./config/elasticsearch.yml`文件中的配置），它就能自动发现并加入到第一个节点的集群中。如果没有，请结合日志找出问题所在。这可能是多播（multicast）被禁用，或者防火墙阻止了节点间的通信。
****

如果我们启动了第二个节点，这个集群应该叫做**双节点集群(cluster-two-nodes)**

双节点集群——所有的主分片和从分片都被分配:
![双节点集群](../images/02-03_two_nodes.png)

当第二个节点加入后，就产生了三个 **从分片(replica shards)** ，它们分别于三个主分片一一对应。也就意味着即使有一个节点发生了损坏，我们可以保证数据的完整性。

所有被索引的新文档都会先被存储在主分片中，之后才会被平行复制到关联的从分片上。这样可以确保我们的文档在主节点和从节点上都能被检索。

当前，`cluster-health`的状态为`green`，这意味着所有的6个分片（三个主分片和三个从分片）都已激活：

```Js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       2,
   "number_of_data_nodes":  2,
   "active_primary_shards": 3,
   "active_shards":         6,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

1. 集群的`status`是`green`.

我们的集群不仅功能齐全的，并且具有**高可用性**。
