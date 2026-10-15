**Mapping** 是 Elasticsearch 里定义**文档结构和字段类型**的机制，相当于传统数据库里的“表结构 + 字段类型”。

它决定了每个字段怎么被存储、怎么被分析、能不能被搜索、用什么方式搜索。

---

### 1. Mapping 的核心作用

当你往索引里写文档时，Elasticsearch 需要知道：

- 这个字段是字符串？数字？日期？布尔？还是对象？
- 字符串要不要分词？（text 还是 keyword）
- 用哪个分词器（[[analyzer]]）？
- 要不要建立倒排索引？
- 是否支持聚合、排序？
- 日期格式是什么？
- 数值精度怎么存？

这些全部由 **Mapping** 决定。

---

### 2. Mapping 的两种来源

|类型|说明|推荐程度|
|---|---|---|
|**动态 Mapping（Dynamic Mapping）**|写入文档时，ES 自动根据字段值推断类型|方便，但不精确，生产环境慎用|
|**显式 Mapping（Explicit Mapping）**|创建索引时手动定义好每个字段的类型和属性|**强烈推荐**，生产环境标配|

动态 Mapping 示例：你写入 {"age": 25}，ES 会自动把 age 推断为 long 类型。

---

### 3. 常见字段类型

|类型|用途|特点|
|---|---|---|
|**text**|全文搜索（标题、内容、描述）|会分词，建立倒排索引，**不能**直接用于排序/聚合|
|**keyword**|精确匹配、过滤、排序、聚合（状态、标签、ID）|不分词，整个值作为一个 term|
|**long / integer / short / byte**|整数|-|
|**double / float / half_float**|浮点数|-|
|**date**|日期时间|支持多种格式|
|**boolean**|true / false|-|
|**object**|嵌套 JSON 对象|默认扁平化存储|
|**nested**|真正的嵌套数组对象|可独立查询内部对象|
|**geo_point / geo_shape**|地理位置|-|
|**ip**|IP 地址|-|

最常用、也最容易搞混的就是 **text 和 keyword**。

---

### 4. 一个完整的 Mapping 示例

JSON

```
PUT /products
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",                // 全文搜索
        "analyzer": "ik_max_word",     // 中文分词器
        "fields": {
          "keyword": {                 // 多字段（multi-fields）
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "price": {
        "type": "double"
      },
      "status": {
        "type": "keyword"              // 精确匹配、聚合
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "tags": {
        "type": "keyword"
      },
      "description": {
        "type": "text",
        "analyzer": "ik_smart"
      }
    }
  }
}
```

这里 title 用了 **multi-fields** 技巧：

- title 本身是 text，用于全文搜索
- title.keyword 是 keyword，用于精确匹配、排序、聚合

这是非常常见的最佳实践。

---

### 5. Mapping 的重要规则

1. **一旦字段类型确定，不能直接修改**
    - 想把 text 改成 keyword？不行，只能新建索引 + reindex。
2. **动态 Mapping 可以控制严格程度**
    
    JSON
    
    ```
    "dynamic": "strict"   // 遇到未知字段直接报错（最严格）
    "dynamic": "false"    // 忽略未知字段
    "dynamic": "true"     // 自动添加（默认）
    ```
    
3. **可以给不同字段指定不同 analyzer**
    - 索引时用粗粒度分词（ik_max_word）
    - 搜索时用细粒度分词（ik_smart）——这就是 search_analyzer

---

### 6. 和倒排索引的关系

Mapping 直接决定了倒排索引怎么建：

- text 类型 → 会走 analyzer → 生成多个 term → 写入倒排索引
- keyword 类型 → 整个值作为一个 term → 写入倒排索引
- long / date 等数值类型 → 用专门的数值索引结构（BKD 树等），不是传统倒排

---

**一句话总结**：  
Mapping 就是告诉 Elasticsearch“每个字段是什么类型、怎么分析、怎么索引”的规则说明书。它是倒排索引能否正确、高效工作的基础。