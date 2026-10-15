### Character Filters 的作用

在文本还没被分词之前，对**原始字符流**进行预处理。它直接操作字符串本身，而不是已经切好的 token。

常见用途包括：

- 去除或替换 HTML 标签
- 把特殊符号、全角字符转成半角
- 映射某些字符（比如把 & 转成 and，把 ß 转成 ss）
- 清除控制字符、零宽字符等

### 常用的内置 Character Filters

|名称|作用|典型场景|
|---|---|---|
|html_strip|去掉 HTML 标签，保留文本内容|处理网页抓取的内容|
|mapping|按自定义规则做字符替换|全角转半角、特殊符号映射|
|pattern_replace|用正则表达式替换字符|清理特定格式的文本|

### 简单示例

JSON

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_html_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

输入：<p>Hello &amp; World</p>  
经过 html_strip 后变成：Hello & World  
再经过后续 tokenizer 和 filter 处理。