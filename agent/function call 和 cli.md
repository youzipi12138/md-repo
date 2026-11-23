**Function Call（函数调用）和 CLI（命令行接口）的核心区别**，在做「达人推荐 Agent」时非常关键。

### 1. 本质区别

|维度|Function Call|CLI|
|---|---|---|
|**交互方式**|模型直接输出结构化参数（通常是 JSON），调用预定义好的函数|模型输出一段文本命令，让系统去执行 shell 命令|
|**输入**|严格遵循函数的 schema（参数名、类型、必填项）|自由文本，像人在终端里敲命令|
|**输出**|函数直接返回结构化结果（dict/list/对象）|通常是纯文本 stdout/stderr，需要自己解析|
|**可靠性**|高（类型检查、参数校验、错误可预期）|较低（命令拼错、输出格式变化、解析失败都很常见）|
|**安全性**|相对可控（只能调用你定义的函数）|风险更高（可能执行危险命令）|

### 2. 在 Agent 里分别是什么样子

**Function Call 示例**（推荐达人场景）：

Python

```
# 你提前定义好工具
tools = [
    {
        "name": "search_influencers",
        "description": "根据条件搜索达人",
        "parameters": {
            "type": "object",
            "properties": {
                "platform": {"type": "string", "enum": ["抖音", "小红书", "B站"]},
                "category": {"type": "string"},
                "min_followers": {"type": "integer"},
                "max_price": {"type": "number"}
            },
            "required": ["platform", "category"]
        }
    }
]
```

模型会直接输出类似：

JSON

```
{
  "name": "search_influencers",
  "arguments": {
    "platform": "小红书",
    "category": "美妆",
    "min_followers": 100000,
    "max_price": 5000
  }
}
```

然后你的代码直接调用对应的 Python 函数，拿到结构化结果继续推理。

**CLI 方式**：

模型输出类似：

Bash

```
python search_influencer.py --platform 小红书 --category 美妆 --min_followers 100000
```

或者更底层的：

Bash

```
curl "https://api.xxx.com/influencers?..."
```

你需要把命令丢给 shell 执行，再自己解析返回的文本。

### 3. 实际做达人推荐 Agent 时怎么选？

**优先用 Function Call 的场景（强烈推荐）：**

- 搜索达人、过滤条件、查报价、查历史数据、计算匹配度
- 需要稳定、可预期、可复现的结果
- 后续还要做多步推理（比如先搜 → 再排序 → 再生成推荐理由）
- 要接入数据库、内部 API、向量检索等

**适合用 CLI 的场景：**

- 临时跑一些脚本、数据处理、爬虫调试
- 调用一些没有现成 API、只能通过命令行工具完成的操作
- 快速原型验证（先用 CLI 跑通，再封装成 Function Call）

### 4. 工程建议（做达人推荐 Agent）

1. **核心能力全部做成 Function Call**  
    比如：`search_influencers`、`get_influencer_detail`、`calculate_match_score`、`generate_recommendation_reason` 等。
    
2. **CLI 只作为兜底或辅助**  
    比如执行一些一次性的数据分析脚本、导出 Excel、跑评估脚本等。
    
3. **如果必须用 CLI，最好再包一层**  
    把常用的 CLI 命令封装成 Function Call，而不是让模型直接自由输出命令。这样既保留了灵活性，又大幅提升稳定性。
    

简单总结：

- **Function Call** = 给模型一套「标准工具箱」，让它按规范调用。
- **CLI** = 让模型直接「打开终端敲命令」。

做达人推荐这种业务型 Agent，**Function Call 是主流且更稳的选择**。CLI 更适合作为补充手段。
# 那飞书为什么不使用function call呢
**飞书不是“不用 Function Call”，而是选择了更通用、更落地的路径：先做好 CLI + Skills，再让各种 Agent 自己用 Function Call 去调用它。**

### 核心原因拆解

|原因|说明|
|---|---|
|**1. 通用性更强**|Function Call 的格式取决于模型厂商（OpenAI、Anthropic、Gemini 等 schema 不完全一样）。CLI 是统一的文本接口，任何能跑 shell 的 Agent（Claude Code、Cursor、Codex、自研 Agent）都能直接用，不用为每个模型单独适配。|
|**2. 覆盖面巨大**|飞书有 18 个业务域、200+ 精选命令、2500+ 原始 API。全部做成精细的 Function Call schema 维护成本极高。CLI 用「快捷命令 + API 命令 + 原生调用」三层结构，灵活很多。|
|**3. 人和 Agent 都能用**|CLI 对开发者本身就很有用（批量处理多维表格、同步文档、脚本自动化）。Function Call 主要只服务模型。|
|**4. 生态已经验证**|当前主流 Coding Agent 对 CLI 的支持非常成熟，调用成功率高。官方还专门做了 26 个 Skills，把 CLI 命令包装成 Agent 更容易理解的结构化工具描述。|
|**5. 实际效果更好**|官方做过实测：经过调优的 CLI 命令（智能默认值、友好错误提示、结构化输出），Agent 调用成功率明显高于直接让模型拼复杂的 OpenAPI 参数。|

### 飞书实际做的事情是：

1. **官方提供 CLI**（`lark-cli`）——统一、稳定、覆盖广的执行层。
2. **提供 Skills**——把 CLI 能力包装成结构化描述，方便 Agent 用 Function Call / Tool Use 的方式调用。
3. **也支持 MCP**——给需要标准协议的场景用。

所以飞书并没有否定 Function Call，而是：

> **用 CLI 做底层执行能力，用 Skills / MCP 向上暴露给 Function Call。**

这相当于：

- Function Call = Agent 的“说话方式”
- CLI = 飞书给的“手”
- Skills = 教 Agent 怎么正确用这双手的说明书

### 简单类比

- 纯 Function Call：你得为每个模型单独写一套“飞书专用工具定义”。
- CLI + Skills：飞书先把能力做成标准命令，再告诉所有 Agent“你可以用这些命令”，Agent 自己决定用 Function Call 还是直接 shell 调用。

后一种方式扩展性更强，也更符合当前 Agent 工具生态的现状。

**总结**：飞书不是不用 Function Call，而是选择了**“CLI 做能力底座 + Skills 做结构化封装”**的路线，这样既能服务各种 Agent，又能同时方便人类开发者使用，维护成本也更可控。