**Structured Output 就是让大模型按照你规定的数据结构输出内容，而不是随意生成一段自然语言。**

例如，你问大模型：

> 帮我分析一下这个达人是否适合 AI 品牌推广。

普通 LLM 可能返回：

```
这个达人比较适合 AI 品牌推广。
他的粉丝主要集中在科技领域，
互动率也比较不错，推荐合作。
```

这段话人能看懂，但是程序不好处理。

如果使用 Structured Output，可以要求模型必须返回：

```
{
  "talent_name": "科技达人A",
  "industry": "AI",
  "suitable": true,
  "score": 92,
  "reason": [
    "粉丝群体与 AI 用户高度匹配",
    "近期互动率较高",
    "历史商业合作表现良好"
  ]
}
```

这就叫结构化输出。

# **一、为什么需要 Structured Output？**

假设你正在开发达人推荐 Agent。

你的 Agent 需要完成：

1. 理解用户需求。
    
2. 提取筛选条件。
    
3. 查询达人。
    
4. 分析达人。
    
5. 生成推荐评分。
    
6. 返回前端展示。
    

如果每一步都返回自然语言，就会有问题。

例如模型第一次返回：

```
我认为应该找一些粉丝超过50万的AI达人。
```

第二次返回：

```
平台选择抖音，粉丝数大于50万。
```

第三次返回：

```
推荐这个达人，评分92分。
```

这些内容格式不统一，后端很难稳定地解析。

所以我们需要规定：

- 必须返回 JSON。
    
- 字段名称固定。
    
- 字段类型固定。
    
- 必填字段不能缺失。
    
- 枚举值只能从指定范围选择。
    

例如：

```
{
  "platform": "douyin",
  "industry": "AI",
  "min_fans": 500000,
  "sort_by": "engagement_rate"
}
```

这样 Java 后端就能直接反序列化成 DTO。

**Structured Output 的核心价值：让大模型的输出从不可预测的文本，变成可以被程序消费的数据。**

# **二、Structured Output 是如何工作的？**

可以把它理解为：你提前给大模型一份数据结构说明书。

例如，我们规定达人推荐结果必须满足下面这个 Schema：

```
{
  "type": "object",
  "properties": {
    "talent_name": {
      "type": "string"
    },
    "score": {
      "type": "integer"
    },
    "suitable": {
      "type": "boolean"
    },
    "reason": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": [
    "talent_name",
    "score",
    "suitable",
    "reason"
  ],
  "additionalProperties": false
}
```

这个 Schema 告诉模型：

|字段|类型|含义|
|---|---|---|
|talent_name|string|达人名称|
|score|integer|推荐分数|
|suitable|boolean|是否适合|
|reason|array|推荐原因|

然后把 Schema 交给支持 Structured Output 的模型接口。

模型就会按照这个结构返回结果。

**用户需求**

分析达人是否适合 AI 品牌推广

**LLM + JSON Schema**

模型根据指定的数据结构生成结果

Structured Output

```
{
  "score": 92,
  "suitable": true
}
```

**Java DTO / 前端页面**

直接读取字段、展示评分和推荐理由

这里有一个细节：

仅仅在 Prompt 中写一句「请返回 JSON」，不等于严格的 Structured Output。

真正的结构化输出通常通过 JSON Schema 等约束实现。不同模型和 API 对 Schema 的支持范围不同，仍然需要处理拒绝回答、截断和业务校验等情况。