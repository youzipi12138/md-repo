**Elasticsearch 中的 Segment（段）**

Segment 是 Elasticsearch（底层 Lucene）中**最基本的倒排索引单元**。  
可以简单理解为一份**不可变的、完整的小倒排索引**。

---

### 1. Segment 是什么？

每个 Segment 都包含：

- 自己的 Term Dictionary（词典）
- 自己的 Postings List（倒排列表）
- 正排数据（_source、doc values 等）
- 删除标记等其他结构

它是一个**自包含、不可修改**的索引文件集合。

一个 **Shard** 通常由**多个 Segment** 组成。

---

### 2. Segment 是怎么产生的？

回顾写入流程：

text

```
文档写入
   ↓
Analyzer 分析成 term
   ↓
先进入内存 Buffer
   ↓
Buffer 满了 / refresh 时间到了 / 手动 refresh
   ↓
生成一个新的 Segment（不可变）
   ↓
这个 Segment 立刻对搜索可见
```

所以：

- **Refresh** 的本质，就是把内存 Buffer 里的数据写成一个新的 Segment。
- 每次 refresh 都可能产生一个新的小 Segment。

---

### 3. Segment 的核心特性：不可变（Immutable）

一旦 Segment 生成，就**不能再修改**里面的内容。

好处：

- 读写可以完全并发，不需要加锁
- 文件可以被操作系统缓存（OS Cache）
- 实现简单，稳定性高

代价：

- 删除文档时，不会立刻从 Segment 里物理删除，只是打上一个**删除标记**（.del 文件）
- 更新文档 = 先删除旧文档 + 写入新文档（新旧文档可能在不同 Segment）

真正的物理清理，要等到后面的 **Segment Merge** 时才会发生。

---

### 4. 一个 Shard 里有很多 Segment

随着不断写入和 refresh，一个 Shard 里会积累很多小 Segment：

text

```
Shard
├── Segment_1（较早生成，较大）
├── Segment_2
├── Segment_3
├── Segment_4（最新 refresh 出来的小段）
└── ...
```

搜索时，协调节点把请求发到 Shard 后，这个 Shard 会**同时查询它内部的所有 Segment**，再把结果合并返回。

Segment 数量太多会带来问题：

- 每个 Segment 都有内存开销（Term Dictionary 等）
- 搜索时要查更多文件，性能下降
- 文件句柄消耗增加

---

### 5. Segment Merge（段合并）——非常关键

为了解决“Segment 太多”的问题，Elasticsearch 后台会不断做 **Merge**：

- 把多个小 Segment 合并成一个更大的 Segment
- 合并过程中会真正物理删除那些被标记删除的文档
- 合并后老的小 Segment 会被删除

默认使用 TieredMergePolicy 策略，优先合并大小相近的 Segment。

Merge 是比较消耗 CPU 和磁盘 I/O 的操作，所以写入很重的时候，有时需要控制 merge 的并发度（index.merge.scheduler.max_thread_count）。

---

### 6. Segment 和搜索、写入的关系总结

|操作|和 Segment 的关系|
|---|---|
|**写入 + Refresh**|产生新的小 Segment|
|**搜索**|同时查询一个 Shard 内的所有 Segment，再合并结果|
|**删除/更新**|只打删除标记，不立刻改 Segment|
|**Merge**|合并小 Segment，真正清理被删文档，减少 Segment 数量|
|**Flush**|把 Segment 持久化到磁盘，并清空 Translog|

---

### 7. 实际影响

- **refresh_interval 越小**：Segment 产生越频繁 → 近实时性越好，但 Segment 数量容易暴涨，搜索性能可能下降。
- **写入高峰时**：适当加大 refresh_interval，减少小 Segment 的产生。
- **Force Merge**：可以手动触发把 Segment 合并到指定数量（生产环境慎用，会消耗大量资源）。