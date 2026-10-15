Analyzer（**分析器**） 是 Elasticsearch 在建立倒排索引和搜索时，用来**把文本处理成一个个 term（词）** 的核心组件。

简单说：它负责把你写入的原始文本，变成倒排索引里真正存储和搜索的那些“词”。

---

### 1. Analyzer 的作用

当你往一个 text 类型字段写入内容，或者执行全文搜索时，Analyzer 会做以下事情：

1. 清洗文本（去掉 HTML、转换字符等）
2. 把文本切分成一个个 token（分词）
3. 对 token 做标准化处理（小写、去停用词、词干提取、同义词等）

最终输出的就是一组 **term**，这些 term 才会被写入倒排索引。

---

### 2. Analyzer 的三大组成部分

一个完整的 Analyzer 由三个部分按顺序组成：

text

```
原始文本
   ↓
① Character Filters（字符过滤器）
   ↓
② Tokenizer（分词器）
   ↓
③ Token Filters（词元过滤器）
   ↓
最终的 terms
```

| 组件                    | 作用                 | 常见例子                                                 |
| --------------------- | ------------------ | ---------------------------------------------------- |
| **Character Filters** | 在分词前对原始字符流做处理      | 去掉 HTML 标签、把 & 转成 and、全角转半角                          |
| **[[Tokenizer]]**     | 把文本切成一个个 token     | standard（按空格/标点切）、ik_max_word（中文）、keyword（不分词）       |
| **Token Filters**     | 对切出来的 token 做进一步处理 | lowercase（转小写）、stop（去停用词）、stemmer（词干提取）、synonym（同义词） |

---

### 3. 内置常用 Analyzer

|Analyzer|特点|适用场景|
|---|---|---|
|**standard**|默认分析器。按语法切分 + 转小写|英文为主的文本|
|**simple**|按非字母字符切分 + 转小写|简单场景|
|**whitespace**|只按空格切分，不做其他处理|需要保留原始大小写和标点时|
|**keyword**|**完全不分词**，整个字段值作为一个 term|和 keyword 类型配合使用|
|**ik_max_word** / **ik_smart**|中文分词（需要安装 IK 插件）|中文搜索（最常用）|
|**english**|英文专用，带词干提取和停用词|纯英文内容|

---

### 4. 实际例子

假设原始文本："The Quick Brown Foxes!"

使用 **standard analyzer** 处理后得到的 terms：

text

```
quick
brown
fox
```

过程大致是：

1. Tokenizer 切成：The Quick Brown Foxes
2. lowercase 过滤器：变成小写
3. 去掉停用词 the
4. 词干提取：foxes → fox

这些 terms 才会进入倒排索引。

---

### 5. 索引时 vs 搜索时的 Analyzer

Mapping 里可以分别指定：

JSON

```
"title": {
  "type": "text",
  "analyzer": "ik_max_word",        // 写入时用（尽量切得细）
  "search_analyzer": "ik_smart"     // 搜索时用（切得相对合理）
}
```

- **analyzer**：文档写入时使用，决定倒排索引里存什么[[ term]]
- **search_analyzer**：用户搜索时使用，决定查询词怎么被分析

这是中文搜索里非常常见的优化手段。