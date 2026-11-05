# 一、VPC 到底是什么？

VPC 全称是 Virtual Private Cloud（虚拟私有云）。

简单理解：

VPC 就是云厂商在物理网络基础设施上，为你创建的一块逻辑隔离的私有网络。

你可以把它理解成：你在云厂商那里租了一块属于自己的网络空间。

这块网络空间有自己的：

- IP 地址范围。
    
- 子网划分。
    
- 路由表。
    
- 网络隔离规则。
    
- 网络连接方式。
    

比如，你在腾讯云、阿里云或者 AWS 上创建了一台云服务器，这台服务器通常需要放在某个 VPC 里面。

[Route Table_Basic Concepts_Service Overview_Virtual Private Cloud-Huawei Cloud](https://images.openai.com/static-rsc-4/6Sh_poBuz9b7s3909R58GCr8PAVnEC0Lg5xWVlidkw6UMJW6ItZ1ShH8yevhcXO7zm2x19BL0sMm5gKnu80fA-94JbHy3LD0NgvwxgpB44R1S_MFwmKZ7M__tmlhvQ-3c_rgaozhYVgtCTLhOjscM_0sv0kC2IqGsxrHcKJAHfI?purpose=inline)

[support.huaweicloud.com](https://support.huaweicloud.com/intl/en-us/productdesc-vpc/en-us_topic_0038263963.html?utm_source=chatgpt.com)

# 二、为什么需要 VPC？

假设有一家云厂商，它有 100 万台物理服务器。

现在有三家公司购买了云服务器：

- A 公司：购买了 10 台服务器。
    
- B 公司：购买了 20 台服务器。
    
- C 公司：购买了 50 台服务器。
    

如果所有公司的服务器都放在同一个网络里，会发生什么？

例如：

A 公司的一台服务器 IP 是：

`192.168.1.10`

B 公司也有一台服务器：

`192.168.1.10`

这就涉及两个问题：

1. 不同公司的网络可能发生 IP 地址冲突。
    
2. 不同公司的服务器不应该随便互相访问。
    

所以云厂商需要给每个客户划分独立的网络空间。

[Connect, Manage & Extend Your Private Cloud Environment | Nutanix Flow](https://images.openai.com/static-rsc-4/pfjgDxTQguTV0wJWNDG21Myp9jmFRZht6-ttwXIYmGZhcoUhD3QRXTYg_iHsHMRiEL9dmDnCXxDNn1HRETKFmxa7B_b_Tl5IQtdO1QqIlj08s7v87_Gqi5ETqvYCn-fzMAiC-2JsLGDh9-kEsIJnNllN370wBFRrUTkIBkuUD6c?purpose=inline)

[nutanix.com](https://www.nutanix.com/en_sg/products/flow/networking?utm_source=chatgpt.com)

例如：

|公司|VPC|IP 网段|
|---|---|---|
|A 公司|VPC-A|10.0.0.0/16|
|B 公司|VPC-B|10.0.0.0/16|
|C 公司|VPC-C|10.0.0.0/16|

注意：三个 VPC 可以使用相同的 IP 网段。

因为它们是逻辑隔离的网络，彼此之间不会因为 IP 相同就发生冲突。

这就是 VPC 最核心的价值：网络隔离。

# 三、VPC 里面有什么？

这里是重点。

一个 VPC 并不只是一块网络，它里面还有很多网络组件。

我们来看一个典型结构：

[GCP Routing Basics Part 1](https://images.openai.com/static-rsc-4/bSjrTOnc1OzyAUlYICz75F2rPu5n7cVVJBitnoxiel365KrLrPk-e9lyuCR3OTh6oUturXcWgXZr47mA7DKM7xDzAomAmBcfc6g0X9LABY6UalkvYWduA75ZRDNkpphK-c0Z8B0udf1aS2wxsoX6ULroae5tfMshNEYP_uDRnNs?purpose=inline)

[linkedin.com](https://www.linkedin.com/pulse/gcp-routing-basics-part-1-mohammad-alhyari-zu41f?utm_source=chatgpt.com)

假设我们创建一个 VPC：

`10.0.0.0/16`

里面可以划分多个子网。

## 1. CIDR：VPC 的 IP 地址范围

例如：

`10.0.0.0/16`

这是什么意思？

它代表 VPC 可以使用的一个 IP 地址网段。

其中：

- `10.0.0.0` 是网段起始地址。
    
- `/16` 表示网络前缀长度为 16 位。
    
- 整个网段包含 65,536 个地址（实际可分配数量视云厂商规则而定）。
    

你可以把它理解成：

VPC 划定了自己这块网络的 IP 地址地盘。

## 2. Subnet：子网

VPC 创建好之后，通常还会继续划分子网。

比如：

VPC：`10.0.0.0/16`

里面划分：

|子网|IP 网段|用途|
|---|---|---|
|Subnet-A|10.0.1.0/24|Web 服务器|
|Subnet-B|10.0.2.0/24|Java 后端服务器|
|Subnet-C|10.0.3.0/24|MySQL 数据库|

为什么需要划分子网？

因为不同类型的服务器可能有不同的网络需求。

例如：

- Web 服务需要接受公网访问。
    
- Java 后端只需要被 Web 服务访问。
    
- MySQL 数据库只需要被 Java 后端访问。
    

这样就可以通过子网、路由和安全规则来规划网络。

## 3. Route Table：路由表

这个非常重要。

路由表的作用是：

决定一个数据包应该往哪里走。

比如：

Java 后端服务器想访问 MySQL 数据库。

它要根据目标 IP 地址查询路由表，确定下一跳应该怎么走。

再比如：

Java 后端服务器需要访问互联网中的某个 API。

路由表可能配置：

|目标网段|下一跳|
|---|---|
|10.0.0.0/16|VPC 内部路由|
|0.0.0.0/0|NAT 网关|

这里：

`0.0.0.0/0` 代表所有 IPv4 目标地址。

也就是说，除了有更具体路由匹配的流量，其他流量都可以交给 NAT 网关处理。

## 4. Security Group：安全组

安全组可以理解成服务器网卡或云服务器实例层面的虚拟防火墙。

比如：

你的 Java 服务开放了：

`8080` 端口。

你可以设置：

- 允许 Web 服务器访问 8080。
    
- 不允许公网直接访问 8080。
    
- 允许 Java 服务器访问 MySQL 的 3306。
    

这样可以控制哪些流量能够进入服务器。

注意：安全组和路由表不是一回事。

路由表负责决定流量往哪里走，安全组负责控制流量能不能通过。

# 四、结合 Java 后端开发理解 VPC

假设你在一家达人营销公司，公司的系统需要处理大量视频数据。

你们可能有这样的架构：

[Amazon Elastic File System (EFS) | Cloud File Storage | Resources](https://images.openai.com/static-rsc-4/he1A-4V2_qmWNmW_SSV9HSE1jDXa0jdi1gKuNaWfCLFphN75lbsMAA3WdV0UT92W1WLokIw6FMGr8uW4M8F5s7ODvfpChnidyu84-1OHj5aQcWe_Kr9-IOt-oMT9CPH4hPQr13-EZhdoMcXI0chUa7M17rfxeyg31326C1_iEzE?purpose=inline)

[aws.amazon.com](https://aws.amazon.com/efs/resources/?utm_source=chatgpt.com)

假设创建了一个 VPC：

`10.0.0.0/16`

里面部署：

|组件|内网 IP|作用|
|---|---|---|
|CLB|10.0.1.10|负载均衡|
|Java 服务 A|10.0.2.10|后端业务|
|Java 服务 B|10.0.2.11|后端业务|
|Redis|10.0.3.10|缓存|
|MySQL|10.0.3.20|数据库|

现在发生几个场景。

场景一：用户访问你的网站

用户浏览器：

```
https://www.example.com
```

请求经过：

```
用户
  ↓
公网
  ↓
CLB
  ↓
Java 服务 A / Java 服务 B
```

CLB 把请求转发给 Java 服务。

场景二：Java 服务访问 Redis

```
Java 服务
    ↓
10.0.3.10:6379
    ↓
Redis
```

由于 Java 和 Redis 都在同一个 VPC 内，因此可以通过内网 IP 通信。

不需要经过公网。

场景三：Java 服务访问外部 API

例如，你的 Java 服务需要调用抖音开放平台的 API。

```
Java 服务
    ↓
NAT 网关
    ↓
公网
    ↓
抖音开放平台
```

这里 NAT 网关可以让没有公网 IP 的私有服务器访问互联网。

# 五、VPC、Subnet、EIP、NAT、CLB 有什么关系？

这几个概念容易混淆，我给你放在一起：

|概念|作用|通俗理解|
|---|---|---|
|VPC|创建独立的私有网络|一个小区|
|Subnet|划分 VPC 内部网络|小区里的楼栋|
|内网 IP|VPC 内部通信地址|楼栋里的房间号|
|EIP|提供公网 IP 地址|小区对外的公网地址|
|NAT|实现内外网地址转换|小区的出入口|
|CLB|分发外部请求|小区的前台接待|
|Route Table|决定流量转发方向|道路指示牌|
|Security Group|控制网络访问|门禁系统|

# 六、最后理解一个核心问题

你一定要记住：

VPC 本身不是一台服务器，也不是一个公网 IP，而是一套逻辑隔离的网络环境。

它解决的是：

1. 我的服务器应该处于哪个网络？
    
2. 我的服务器应该使用什么内网 IP？
    
3. 我的服务器能不能访问其他服务器？
    
4. 我的服务器能不能访问公网？
    
5. 公网请求应该如何进入我的服务器？
    

这才是 VPC 的核心价值。