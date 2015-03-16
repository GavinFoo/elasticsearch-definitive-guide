# 集群健康

在 Elasticsearch 集群中可以监控统计很多信息，其中最重要的就是：**集群健康(cluster health)**。它的 `status` 有 `green`、`yellow`、`red` 三种；

```
GET /_cluster/health
```

在一个没有索引的空集群中，它将返回如下信息：

```Js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
1.  `status` 是我们最应该关注的字段。

`status` 可以告诉我们当前集群是否处于一个可用的状态。三种颜色分别代表：

| 状态     | 意义                                     |
| -------- | ---------------------------------------- |
| `green`  | 所有主分片和从分片都可用               |
| `yellow` | 所有主分片可用，但存在不可用的从分片 |
| `red`    | 存在不可用的主要分片                 |

在接下来的章节，我们将学习一下什么是**主要分片(primary shard)** 和 **从分片(replica shard)**，并说明这些状态在实际环境中的意义。
