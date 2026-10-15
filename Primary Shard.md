**Primary Shard（主分片）** 是 Elasticsearch 中负责**数据写入和权威数据存储**的分片。

[[为什么文档写入要先写入主分片 Primary Shard]]

### 简单理解

一个索引（Index）会被水平拆分成多个 **Shard**（分片）。每个分片又分为：

- **Primary Shard（主分片）**：真正接收写入请求、处理数据更新的分片。
- **Replica Shard（副本分片）**：主分片的完整拷贝，主要用于读取负载分担和故障容灾。

### 核心特点

1. **写入只走 Primary**
    - 所有的增、删、改操作（indexing、update、delete）都必须先落到 Primary Shard 上。
    - Primary 处理完后，再把变更同步（复制）给对应的 Replica。
2. **每个文档只属于一个 Primary Shard**
    - 文档通过路由算法（默认是文档 _id 的 hash）决定它属于哪个 Primary Shard。
    - 公式大致是：shard_num = hash(_id) % number_of_primary_shards
3. **Primary 是权威数据源**
    - 查询时可以打到 Primary 或 Replica，但写入必须经过 Primary。
    - 如果某个 Primary 挂了，集群会从它的某个 Replica 中**提升**一个新的 Primary（这个过程叫 Primary Election）。
4. **数量一旦设定，创建索引后不能直接修改**
    - 创建索引时通过 number_of_shards 指定 Primary 数量（默认 1，老版本默认是 5）。
    - 想改 Primary 数量，只能重建索引（reindex）或使用 split/shrink API。

### 举个例子

假设你创建了一个索引：

JSON

```
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,      // 3 个 Primary Shard
    "number_of_replicas": 1     // 每个 Primary 有 1 个 Replica
  }
}
```

实际会形成：

- Primary-0 + Replica-0
- Primary-1 + Replica-1
- Primary-2 + Replica-2

总共 6 个分片。写入文档时，ES 会根据文档 ID 把请求路由到对应的 Primary，再由 Primary 同步给它的 Replica。

---

**一句话总结**：  
Primary Shard 是 Elasticsearch 分片中真正“干活写数据”的那个主副本，所有写入必须经过它，它是数据的权威来源。