**Function Calling / Tool Calling，本质上就是让大模型具备调用外部系统、执行具体业务操作的能力。**

比如，你对一个普通 ChatGPT 说：

> 帮我找一下抖音上粉丝量 50 万以上、主要做 AI 内容、近 30 天互动率比较高的达人。

普通大模型只能根据自己已有的知识回答，不能直接查询你公司的达人数据库。

但是有了 Tool Calling，它就可以：

1. 理解用户想找什么达人。
    
2. 判断需要查询达人数据库。
    
3. 自动选择 `search_talent` 工具。
    
4. 提取参数：平台、粉丝量、行业、时间范围。
    
5. 调用你后端提供的 API。
    
6. 获取真实达人数据。
    
7. 分析数据并组织成推荐结果。
    

这就是 Agent 最核心的工作机制之一。

下面从原理、协议、代码、架构和实际业务几个角度讲。

# **一、先理解：为什么需要 Function Calling？**

假设你使用一个大模型：

```
用户：帮我查一下张三的粉丝量。
```

大模型本身并不知道你公司的 MySQL 数据库里面有什么。

它也不能直接执行：

```
SELECT fans_count
FROM talent
WHERE talent_name = '张三';
```

因为大模型本质上是一个文本生成模型。

它擅长：

- 理解自然语言。
    
- 分析上下文。
    
- 推理。
    
- 生成内容。
    
- 识别用户意图。
    

但它本身没有：

- 数据库访问能力。
    
- HTTP 请求能力。
    
- 文件读写能力。
    
- 调用公司内部系统的能力。
    
- 操作第三方平台的能力。
    

那么如何解决？

我们给大模型提供一组可以调用的工具。

例如：

|工具名称|功能|
|---|---|
|search_talent|查询达人|
|get_talent_detail|查询达人详情|
|get_talent_stats|查询达人数据表现|
|search_brand|查询品牌|
|create_order|创建商单|
|get_order_status|查询订单状态|

然后告诉大模型：

这些工具分别能做什么，需要什么参数，什么情况下可以调用。

这样，大模型就能够根据用户需求决定调用哪个工具。

**没有 Tool Calling**

用户：帮我查询达人张三的粉丝量

**LLM 只能根据已有知识回答**

无法访问公司真实数据库

**有 Tool Calling**

用户：帮我查询达人张三的粉丝量

**LLM 理解需求并选择工具**

get_talent_detail(name="张三")

**你的 Java / Python 后端执行工具**

查询 MySQL / OpenSearch

**真实数据返回给 LLM**

粉丝量：85 万

**LLM 生成最终回答**

张三目前有 85 万粉丝。

注意：**大模型负责决定调用什么，真正执行代码的是你的后端。**

这是理解 Function Calling 最重要的一点。

# **二、Function Calling 到底是怎么工作的？**

我们用一个完整的例子。

假设你给大模型提供一个工具：

`search_talent`

它的功能是根据条件搜索达人。

首先，你需要定义工具的描述和参数。

```
{
  "type": "function",
  "name": "search_talent",
  "description": "根据平台、行业、粉丝量等条件搜索达人",
  "parameters": {
    "type": "object",
    "properties": {
      "platform": {
        "type": "string",
        "enum": ["douyin", "xiaohongshu"],
        "description": "达人所属平台"
      },
      "industry": {
        "type": "string",
        "description": "达人所属行业"
      },
      "min_fans": {
        "type": "integer",
        "description": "最低粉丝数量"
      }
    },
    "required": ["platform", "industry"]
  }
}
```

这个 JSON 就是工具的定义。

你可以把它理解为：**向大模型注册了一个可以使用的函数说明书。**

它告诉大模型：

- 函数叫什么。
    
- 函数有什么作用。
    
- 需要什么参数。
    
- 参数是什么类型。
    
- 哪些参数是必须的。
    

然后我们开始真正调用。

# **三、Function Calling 的完整执行流程**

这里是重点。你要真正理解大模型和后端之间是如何交互的。

假设用户说：

> 帮我找抖音上 AI 行业粉丝量超过 50 万的达人。

整个过程分为 6 步。

1. **第一步：用户发送请求**
    
    前端把用户的问题发送给 Agent 后端。
    
    ```
    帮我找抖音上 AI 行业
    粉丝量超过 50 万的达人
    ```
    
2. **第二步：后端向 LLM 发送工具定义**
    
    除了用户的问题，还会把可用工具列表一起传过去。
    
    ```
    System Prompt
    User Message
    Tool Definitions
    ```
    
3. **第三步：LLM 决定调用工具**
    
    模型识别出需要查询达人数据，并生成结构化参数。
    
    ```
    {
      "name": "search_talent",
      "arguments": {
        "platform": "douyin",
        "industry": "AI",
        "min_fans": 500000
      }
    }
    ```
    
    模型提出调用请求，还没有执行查询
    
4. **第四步：后端执行函数**
    
    你的 Agent 服务接收工具名称和参数，找到对应的 Java / Python 函数。
    
    ```
    searchTalent(
        "douyin",
        "AI",
        500000
    );
    ```
    
    然后查询 MySQL、OpenSearch 等系统。
    
5. **第五步：把执行结果返回给 LLM**
    
    例如查询到 3 个达人。
    
    ```
    {
      "talents": [
        {"name": "达人A", "fans": 800000},
        {"name": "达人B", "fans": 650000},
        {"name": "达人C", "fans": 520000}
      ]
    }
    ```
    
6. **第六步：LLM 生成最终回答**
    
    模型根据真实查询结果组织语言，返回给用户。
    
    ```
    找到了 3 位符合条件的达人：
    
    1. 达人A：80万粉丝
    2. 达人B：65万粉丝
    3. 达人C：52万粉丝
    ```
    

你会发现一个很重要的细节：

**大模型并没有直接连接 MySQL，而是通过工具调用，让你的后端去连接 MySQL。**

这是一个典型的 LLM + Backend 架构。

# **五、Function Calling 和 Tool Calling 有什么区别？**

这两个概念经常被混用，但严格来说存在一定区别。

**Function Calling 更强调调用一个函数；Tool Calling 是更广泛的工具调用概念。**

|对比|Function Calling|Tool Calling|
|---|---|---|
|核心概念|调用函数|调用工具|
|主要用途|执行某个函数|执行各种外部能力|
|工具类型|通常是函数|函数、搜索、代码执行等|
|参数|通常是结构化 JSON|根据工具类型决定|
|执行者|通常是应用后端|应用后端或工具运行环境|