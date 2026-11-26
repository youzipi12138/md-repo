
Tool Registry：注册了哪些工具、工具描述、参数 Schema。

**Tool Registry 就是 Agent 的工具管理系统。它负责管理 Agent 有哪些工具、每个工具能做什么、需要什么参数、谁有权限使用，以及应该由哪个程序执行。**

你可以把它理解为传统后端中的 Spring Bean 容器 + API 管理中心 + 工具调度入口的结合体。

不过它不只是一个简单的工具列表。

如果你们公司未来要做一个类似巨量星图小星 Pro 的达人推荐 Agent，可能会有：

- 达人搜索工具。
    
- 达人画像工具。
    
- 品牌分析工具。
    
- 水下商单识别工具。
    
- 达人推荐评分工具。
    
- 报价分析工具。
    
- 订单查询工具。
    
- 合同生成工具。
    

这些工具可能由不同开发人员维护，甚至有些是 Python 服务，有些是 Java 服务，有些是 MCP Server。

如果全部写死在一个 Java 类里面，后面维护会非常痛苦。

这就是为什么需要 Tool Registry。

# **一、先理解没有 Tool Registry 会发生什么**

假设你现在使用 Java 开发 Agent。

最开始只有三个工具：

```
if ("search_talent".equals(toolName)) {
    return talentService.searchTalent(args);
}

if ("get_talent_detail".equals(toolName)) {
    return talentService.getTalentDetail(args);
}

if ("search_brand".equals(toolName)) {
    return brandService.searchBrand(args);
}
```

这时候没有什么问题。

但是随着业务发展，你们有了 50 个工具：

```
if ("search_talent".equals(toolName)) {
    // ...
} else if ("get_talent_detail".equals(toolName)) {
    // ...
} else if ("search_brand".equals(toolName)) {
    // ...
} else if ("get_commercial_history".equals(toolName)) {
    // ...
} else if ("calculate_score".equals(toolName)) {
    // ...
} else if ("generate_report".equals(toolName)) {
    // ...
}
```

后面还要继续增加：

- 100 个工具。
    
- 不同 Agent 使用不同工具。
    
- 不同用户拥有不同工具权限。
    
- 工具需要启用、禁用。
    
- 工具需要修改参数。
    
- 工具需要切换执行服务。
    
- 工具需要记录调用日志。
    

这时候就会出现几个问题：

1. 增加一个工具就要修改核心代码。
    
2. Agent 和具体业务 Service 耦合。
    
3. 无法方便地动态管理工具。
    
4. 工具权限散落在不同地方。
    
5. 工具描述和实际执行代码容易不一致。
    

所以我们需要把工具定义和工具执行逻辑分离。

# **二、Tool Registry 到底是什么？**

先看一个整体架构。

**Agent / LLM**

根据用户需求选择工具

**Tool Registry**

工具注册中心

**工具元数据**

名称、描述、Schema

**权限管理**

用户、角色、Agent

**状态管理**

启用、禁用、版本

**执行器映射**

Java、HTTP、MCP

**Tool Executor**

根据注册信息找到真正的执行程序

**Java**

业务 Service

**HTTP**

外部 API

**MCP**

MCP Server

**真实业务系统**

MySQL / OpenSearch / Lakehouse / 第三方平台

这里你要先区分三个概念：

|模块|负责什么|
|---|---|
|Tool Registry|管理工具，告诉系统有哪些工具|
|Tool Executor|根据工具定义执行具体操作|
|Agent Runtime|管理模型调用、工具调度和多轮执行|

这三个模块不要混在一起。

# **三、一个工具在注册中心里面应该包含什么？**

这是设计 Tool Registry 最核心的地方。

假设你要注册一个达人搜索工具：

`search_talent`

你不能只保存一个名字。

至少需要保存以下信息：

|属性|示例|作用|
|---|---|---|
|tool_name|search_talent|工具唯一标识|
|description|根据条件搜索达人|让 LLM 理解工具|
|parameters|JSON Schema|告诉模型需要什么参数|
|executor_type|JAVA|告诉系统由谁执行|
|executor_config|TalentService|执行器配置|
|category|talent|工具分类|
|enabled|true|是否启用|
|version|1.0.0|工具版本|
|timeout|5000|执行超时时间|
|permission|talent:search|所需权限|

例如，完整的工具定义：

```
{
  "name": "search_talent",
  "description": "根据平台、行业、粉丝量搜索达人",
  "category": "talent",
  "version": "1.0.0",
  "enabled": true,
  "executor": {
    "type": "JAVA",
    "handler": "searchTalentTool"
  },
  "parameters": {
    "type": "object",
    "properties": {
      "platform": {
        "type": "string",
        "enum": ["douyin", "xiaohongshu"]
      },
      "industry": {
        "type": "string"
      },
      "min_fans": {
        "type": "integer"
      }
    },
    "required": [
      "platform",
      "industry"
    ],
    "additionalProperties": false
  },
  "permission": {
    "required": true,
    "code": "talent:search"
  },
  "timeout": 5000
}
```

你可以看到，这里实际上包含两类信息：

**第一类：给大模型看的信息。**

- name
    
- description
    
- parameters
    

这些内容用于生成 Function Calling 的工具定义。

**第二类：给后端看的信息。**

- executor
    
- permission
    
- timeout
    
- enabled
    
- version
    

这些内容用于真正执行工具。

这个区分非常重要。

LLM 不需要知道你的 Java 类名、数据库地址、内部服务密钥等信息。

这些属于后端执行层的配置。

# **四、Tool Registry 的数据库应该怎么设计？**

如果让你来负责公司这个模块，我建议不要把所有东西都塞进一张表。

可以先设计 5 张核心表。

## **1. 工具基本信息表：agent_tool**

保存每个工具的基础信息。

```
CREATE TABLE agent_tool (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,

    tool_code VARCHAR(100) NOT NULL UNIQUE,

    tool_name VARCHAR(100) NOT NULL,

    description TEXT,

    category VARCHAR(50),

    version VARCHAR(30),

    executor_type VARCHAR(30),

    executor_config JSON,

    enabled TINYINT DEFAULT 1,

    timeout_ms INT DEFAULT 5000,

    created_at DATETIME,

    updated_at DATETIME
);
```

数据示例：

|id|tool_code|category|executor_type|enabled|
|---|---|---|---|---|
|1|search_talent|talent|JAVA|1|
|2|get_talent_detail|talent|HTTP|1|
|3|analyze_commercial|commercial|MCP|1|
|4|calculate_score|analysis|JAVA|1|

注意：`executor_config` 可以存执行器的类型相关配置，但不要把 API 密钥、数据库密码等敏感信息直接存进去。

## **2. 工具参数 Schema 表：agent_tool_schema**

保存工具输入参数的 JSON Schema。

```
CREATE TABLE agent_tool_schema (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,

    tool_id BIGINT NOT NULL,

    schema_type VARCHAR(30),

    schema_content JSON,

    created_at DATETIME
);
```

为什么单独设计？

因为一个工具可能随着业务变化，参数会发生改变。

例如：

V1：

```
{
  "industry": "AI",
  "min_fans": 500000
}
```

V2：

```
{
  "industry": "AI",
  "min_fans": 500000,
  "max_fans": 5000000,
  "platform": "douyin"
}
```

把 Schema 独立出来，方便做版本管理。

## **3. Agent 与工具关联表：agent_tool_relation**

一个 Agent 可以使用多个工具，一个工具也可以被多个 Agent 使用。

这是典型的多对多关系。

```
CREATE TABLE agent_tool_relation (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,

    agent_id BIGINT NOT NULL,

    tool_id BIGINT NOT NULL,

    enabled TINYINT DEFAULT 1,

    created_at DATETIME,

    UNIQUE KEY uk_agent_tool (agent_id, tool_id)
);
```

例如：

|Agent|工具|
|---|---|
|达人推荐 Agent|search_talent|
|达人推荐 Agent|get_talent_detail|
|达人推荐 Agent|calculate_score|
|品牌分析 Agent|search_brand|
|品牌分析 Agent|analyze_competitors|

这样你就可以控制不同 Agent 能使用哪些工具。

## **4. 工具权限表：agent_tool_permission**

控制哪些角色、用户或业务范围能够调用工具。

```
CREATE TABLE agent_tool_permission (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,

    tool_id BIGINT NOT NULL,

    role_code VARCHAR(100),

    permission_code VARCHAR(100),

    created_at DATETIME
);
```

例如：

|工具|角色|权限|
|---|---|---|
|search_talent|SALES|talent:search|
|create_order|SALES_MANAGER|order:create|
|delete_order|ADMIN|order:delete|

生产环境还需要考虑租户、数据范围、用户级授权以及服务身份，不能仅靠角色表判断。

## **5. 工具调用日志表：agent_tool_call_log**

记录每次工具调用。

```
CREATE TABLE agent_tool_call_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,

    conversation_id VARCHAR(100),

    agent_id BIGINT,

    tool_code VARCHAR(100),

    input_params JSON,

    output_result JSON,

    status VARCHAR(30),

    error_message TEXT,

    duration_ms BIGINT,

    created_at DATETIME
);
```

为什么需要它？

因为 Agent 经常会出现：

- 工具选择错误。
    
- 参数提取错误。
    
- 工具执行超时。
    
- 返回数据异常。
    
- 模型重复调用工具。
    

如果没有日志，你很难定位到底是模型的问题，还是 Java 业务代码的问题。

实际生产还应该对敏感字段进行脱敏，并限制日志保留时间和结果体积。

## **这 5 张表之间是什么关系？**

**agent****_****tool**

工具基本信息

**agent****_****tool****_****schema**

参数定义

**agent****_****tool****_****permission**

权限定义

**agent****_****tool****_****relation**

Agent 关联

**agent****_****tool****_****call****_****log**

执行记录

这是一套初始设计。真正上线时，我还建议增加工具版本发布记录、执行器配置版本、审核状态等能力。

# **五、Java 后端如何实现 Tool Registry？**

下面讲最重要的代码设计。

我建议使用策略模式 + 注册表模式。

你应该熟悉 Java 的策略模式，它在这里非常适合。

## **1. 定义统一的工具执行接口**

所有工具都实现同一个接口。

```
public interface ToolExecutor {

    /**
     * 工具唯一标识
     */
    String getToolCode();

    /**
     * 执行工具
     */
    Object execute(
        Map<String, Object> arguments
    );
}
```

这样不管工具是查询达人，还是查询品牌，都遵循同一个执行规范。

## **2. 实现达人搜索工具**

```
@Component
public class SearchTalentTool implements ToolExecutor {

    private final TalentService talentService;

    public SearchTalentTool(TalentService talentService) {
        this.talentService = talentService;
    }

    @Override
    public String getToolCode() {
        return "search_talent";
    }

    @Override
    public Object execute(
        Map<String, Object> arguments
    ) {
        String platform =
            (String) arguments.get("platform");

        String industry =
            (String) arguments.get("industry");

        Integer minFans =
            (Integer) arguments.get("min_fans");

        return talentService.searchTalent(
            platform,
            industry,
            minFans
        );
    }
}
```

## **3. 实现工具注册中心**

这里是核心。

```
@Component
public class ToolRegistry {

    private final Map<String, ToolExecutor> registry
        = new ConcurrentHashMap<>();

    public ToolRegistry(
        List<ToolExecutor> executors
    ) {
        for (ToolExecutor executor : executors) {
            registry.put(
                executor.getToolCode(),
                executor
            );
        }
    }

    public ToolExecutor getTool(String toolCode) {

        ToolExecutor executor = registry.get(toolCode);

        if (executor == null) {
            throw new RuntimeException(
                "Tool not found: " + toolCode
            );
        }

        return executor;
    }

    public Collection<ToolExecutor> getAllTools() {
        return registry.values();
    }
}
```

这里利用了 Spring 的自动注入机制。

当 Spring 启动时：

```
Spring 容器
    ↓
扫描所有 ToolExecutor 实现类
    ↓
SearchTalentTool
GetTalentDetailTool
SearchBrandTool
    ↓
注入 List<ToolExecutor>
    ↓
ToolRegistry 自动注册
```

以后你增加一个工具：

```
@Component
public class SearchBrandTool implements ToolExecutor {
    // ...
}
```

就不需要修改 `ToolRegistry`。

Spring 启动时就会自动发现它。

这就是注册表模式的价值。

## **4. 工具调用分发**

接下来模型返回：

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

你的执行器：

```
@Service
public class ToolDispatcher {

    private final ToolRegistry toolRegistry;

    public ToolDispatcher(ToolRegistry toolRegistry) {
        this.toolRegistry = toolRegistry;
    }

    public Object dispatch(
        String toolCode,
        Map<String, Object> arguments
    ) {

        ToolExecutor executor =
            toolRegistry.getTool(toolCode);

        return executor.execute(arguments);
    }
}
```

这样就不需要写大量的 `if-else`。

调用关系变成：

```
LLM 返回工具名称
       ↓
ToolDispatcher
       ↓
ToolRegistry
       ↓
根据 toolCode 获取 Executor
       ↓
执行对应 Java Tool
       ↓
返回结果
```

不过，这个版本只是基础实现。生产环境还必须在 `dispatch` 之前完成工具白名单检查、参数 Schema 校验、用户权限校验和执行策略判断，不能让模型随便传一个工具名称就执行。

# **六、工具注册中心如何做到动态配置？**

这是你前面提到的一个关键需求。

假设你们产品经理说：

> 我想临时关闭达人推荐 Agent 中的水下商单识别工具。

传统写法：

```
if (enableCommercialTool) {
    // 执行商单识别
}
```

然后修改代码、重新部署。

这显然不够灵活。

如果有 Tool Registry，就可以设计一个管理后台。

**Agent 工具管理**

管理后台示意

**达人推荐 Agent**

已关联 5 个工具

运行中

**工具列表**

4 / 5 已启用

**search_talent**

搜索符合条件的达人 · JAVA

已启用

**get_talent_detail**

查询达人详细资料 · JAVA

已启用

**analyze_commercial**

识别历史商业合作 · MCP

已启用

**calculate_score**

计算达人推荐评分 · JAVA

已启用

**generate_report**

生成达人推荐报告 · HTTP

已禁用

这是交互示意，开关只改变当前演示状态，不会修改公司的实际配置。

实际系统中，后台可以提供：

|功能|说明|
|---|---|
|新增工具|注册一个新的工具|
|编辑工具|修改描述、参数 Schema|
|启用/禁用|控制工具是否可以调用|
|工具分组|按达人、品牌、商单分类|
|Agent 关联|控制哪个 Agent 可以使用|
|权限配置|控制谁可以调用|
|版本管理|发布和回滚工具版本|
|调用日志|查看调用次数、耗时、成功率|

动态配置的实际执行流程：

```
管理员在后台关闭工具
        ↓
MySQL 更新 enabled = 0
        ↓
配置变更通知 / 定时刷新缓存
        ↓
Tool Registry 更新工具状态
        ↓
Agent 构建可用工具列表时排除它
        ↓
模型不再获得这个工具的调用入口
        ↓
执行层再次校验，拒绝调用
```

注意最后一步很重要。

**不能只从 LLM 的工具列表中删除工具，还要在执行层再次检查启用状态和权限。**

否则模型仍然可能通过旧上下文或恶意构造的调用请求触发工具。

## **动态配置到底能做到什么程度？**

这里有一个很容易误解的地方。

动态注册工具，不等于动态生成 Java 代码。

例如：

- 修改工具描述：可以动态配置。
    
- 修改 JSON Schema：可以动态配置。
    
- 关闭某个工具：可以动态配置。
    
- 将工具分配给不同 Agent：可以动态配置。
    
- 修改 HTTP 工具的目标服务：在安全允许的配置范围内可以动态配置。
    
- 新增一个完全没有实现过的 Java 业务能力：通常还是需要开发代码并部署。
    

因此，建议你采用：

**工具元数据动态配置 + 执行器代码固定实现。**

对于 HTTP、MCP 等通用执行器，可以通过配置新增不同的远程工具，但仍然需要严格控制可访问目标、凭证和调用权限。

# **七、Tool Registry 如何支持不同类型的工具？**

这是企业级设计中非常重要的一点。

你不能假设所有工具都是 Java 方法。

你们公司未来可能同时存在：

- Java 业务工具。
    
- Python AI 分析工具。
    
- HTTP API 工具。
    
- MCP 工具。
    
- SQL 查询工具。
    

所以建议抽象统一的执行器接口。

例如：

```
public interface ToolExecutor {

    Object execute(
        ToolDefinition tool,
        Map<String, Object> arguments
    );
}
```

然后实现不同的执行器。

**Tool Executor 类型体系**

**ToolExecutor**

统一执行接口

**JavaExecutor**

调用 Spring Service

达人查询、订单管理

**HttpExecutor**

调用 HTTP API

第三方数据服务

**McpExecutor**

调用 MCP Server

外部工具服务

**PythonExecutor**

调用 Python 服务

AI 分析、数据计算

例如：

你的 `analyze_commercial` 工具原本由 Java 实现。

后来你们有了一个专门的 Python 视频分析服务。

你可以把执行器配置切换为 HTTP，调用 Python 服务。

Agent 不需要知道底层换了什么技术。

对 LLM 来说，工具仍然叫：

```
analyze_commercial
```

输入和输出 Schema 保持兼容即可。

这就是工具层抽象的价值。

不过，如果输出结构、执行语义发生了变化，就不能只修改执行器配置，还需要做兼容性验证和版本升级。

# **八、真正的企业级 Tool Registry 还需要什么？**

如果只是做一个 Demo，上面的实现已经够了。

但如果你准备把它做成公司正式的 Agent 工具平台，我建议额外关注以下能力。

|能力|解决的问题|优先级|
|---|---|---|
|Schema 校验|防止模型传入错误参数|P0|
|权限控制|防止越权调用|P0|
|工具启停|控制工具是否可用|P0|
|超时与限流|防止调用失控|P0|
|调用日志|定位问题|P0|
|版本管理|工具升级、兼容、回滚|P1|
|缓存|减少重复查询|P1|
|熔断机制|防止故障扩散|P1|
|动态加载|不重启 Agent 更新工具配置|P1|
|工具测试|发布前验证参数和执行结果|P1|
|工具评估|评估模型是否正确选择工具|P2|

尤其要注意三个设计细节：

**1. 工具版本管理**

例如：

```
search_talent
    ├── v1.0.0
    ├── v1.1.0
    └── v2.0.0
```

不要直接覆盖正在使用的工具定义。建议采用版本发布机制，让 Agent 绑定明确的工具版本，测试通过后再切换。

**2. 工具缓存**

达人画像、行业信息等相对稳定的数据，可以适当缓存。

但是订单、付款、实时账户余额等数据不能随意使用过期缓存。

**3. 工具执行幂等性**

例如模型重复调用：

```
create_order()
create_order()
create_order()
```

如果不做幂等控制，可能创建三个订单。

涉及下单、付款、删除等操作，一定要考虑幂等键、业务状态校验和人工确认。