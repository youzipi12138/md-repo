**term（词）** 是 Elasticsearch 倒排索引里**真正存储和用来匹配的最小单位**。

简单说：你写入的原始文本经过 Analyzer 处理后，最终得到的一个个“标准词”，就叫 **term**。

---

### 1. term 到底是什么？

- 原始文本经过分析器（Analyzer）处理后输出的结果。
- 它是倒排索引的**键（key）**。
- 搜索的时候，查询语句也会被分析成 term，然后拿这些 term 去倒排索引里匹配。

**例子**：

原始文本："The Quick Brown Foxes"

经过 standard analyzer 后得到的 terms：

text

```
quick
brown
fox
```

这里的 quick、brown、fox 就是 term。

---

### 2. term 和 token 的区别（容易混淆）

|概念|说明|
|---|---|
|**token**|Tokenizer 刚切分出来的原始结果，还没经过 Token Filter 处理|
|**term**|token 经过所有 Token Filter（小写、去停用词、词干提取等）处理后的**最终结果**|

可以简单记：

- token = 半成品
- term = 最终成品（真正进入倒排索引的东西）

在大多数日常交流中，大家说的“词”基本都是指 term。

---

### 3. term 在倒排索引中的位置

倒排索引的核心结构就是：

text

```
term → postings list（哪些文档包含这个 term，以及位置、词频等信息）
```

例如：

text

```
term: "fox"
  ├── docID: 1, termFreq: 1, positions: [3]
  ├── docID: 8, termFreq: 2, positions: [1, 5]
  └── ...
```

所以：

- 有没有某个 term → 决定文档能不能被搜到
- term 的质量（分词是否合理）→ 直接决定搜索效果好不好

---

### 4. 不同类型字段产生的 term 不一样

|字段类型|产生的 term 特点|示例|
|---|---|---|
|**text**|会分词，一个字段可能产生多个 term|"Quick Brown Fox" → quick, brown, fox|
|**keyword**|**不分词**，整个值就是一个 term|"Quick Brown Fox" → Quick Brown Fox（原样）|
|数值 / 日期|不是传统文本 term，用专门的数值结构存储|内部不是按文本 term 处理|

这就是为什么 text 适合全文搜索，keyword 适合精确匹配、排序、聚合。

---

### 5. 为什么 term 很重要？

1. **搜索匹配的本质**就是 term 的匹配。
2. 分词器（Analyzer）做得好不好，直接决定生成的 term 是否合理。
3. 中文搜索效果差，很多时候就是 term 切得不好（切得太碎或太粗）。
4. 想做同义词、纠错、拼音搜索，本质上都是在操控 term。

---

**一句话总结**：  
term 就是文本被 Analyzer 处理之后，真正写进倒排索引、用来做搜索匹配的“标准词”。它是倒排索引的最小构建块。