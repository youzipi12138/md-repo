## 1. CLB 是干什么的？

假设你现在开发了一个 Java 后台系统，部署了三台服务器：

- 服务器 A：运行 Spring Boot
    
- 服务器 B：运行 Spring Boot
    
- 服务器 C：运行 Spring Boot
    

如果用户直接访问服务器 A，那么所有请求都会打到 A 上。

这会出现几个问题：

- A 的压力很大，B、C 却没什么流量。
    
- A 宕机了，整个系统可能无法访问。
    
- 用户量增加时，单台服务器可能扛不住。
    

这时候就需要 CLB。

[Medium](https://images.openai.com/static-rsc-4/Qm_yN1pTkjlDKyB_ZmqFOF9K3a6eea4LMJRk5sMcqzfqYATm_WbEUk8cALnk4D7ZSMjtMwYfNFhSEAd6HzzjseJEhPOom6AZ4NhInfbM_-R9BvVm9sHeXEx2DUc5qZ1L-7F6-Lfty246RI7Hf_OdPhWYsdy5M2roulKH3Hdm8mg?purpose=inline)

[towardsdev.com](https://towardsdev.com/setting-up-an-external-load-balancer-in-azure-2be44c3168ff?utm_source=chatgpt.com)

## 2. CLB 的工作流程

假设用户访问：

```
https://www.example.com
```

请求经过以下流程：

```
          用户
           |
           |
       www.example.com
           |
           v
        DNS解析
           |
           v
      CLB负载均衡器
           |
     +-----+-----+
     |     |     |
     v     v     v
   Java A Java B Java C
   8080   8080   8080
```

CLB 收到请求之后，根据负载均衡策略，把请求转发给后端服务器。

例如：

|请求|转发目标|
|---|---|
|请求1|服务器 A|
|请求2|服务器 B|
|请求3|服务器 C|
|请求4|服务器 A|

这就是负载均衡。

## 3. CLB 有哪些核心功能？

① 流量分发

常见策略：

- 轮询：依次分配请求。
    
- 加权轮询：性能好的服务器分配更多请求。
    
- 最小连接数：优先转发给连接数较少的服务器。
    

② 健康检查

假设服务器 B 挂了：

```
CLB
 |
 +--- A：正常       接收请求
 |
 +--- B：异常       停止转发
 |
 +--- C：正常       接收请求
```

CLB 可以通过健康检查发现 B 不可用，然后停止向 B 转发流量。

③ HTTPS 证书管理

可以在 CLB 上配置 HTTPS 证书，由 CLB 负责 TLS 连接终止，再将请求转发到后端服务器。

④ 高可用

通过多台后端服务器提供服务，降低单台服务器故障对整个业务的影响。

## 4. CLB 和 Nginx 有什么区别？

这个问题很重要，Java 后端开发经常会遇到。

|对比|CLB|Nginx|
|---|---|---|
|本质|云厂商提供的负载均衡服务|Web服务器/反向代理软件|
|部署位置|云平台基础设施|通常部署在服务器上|
|运维|云厂商负责底层维护|自己维护|
|负载均衡|支持|支持|
|反向代理|支持（取决于类型）|支持|
|健康检查|通常内置|可通过配置或模块实现|

可以这样理解：

- CLB：云平台帮你提供一个负载均衡入口。
    
- Nginx：你自己部署一套软件来实现反向代理和负载均衡。
    

两者也可以一起使用。