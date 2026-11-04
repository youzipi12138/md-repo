EIP 就是云厂商提供给你的一个公网 IP 地址，可以独立分配、绑定和解绑，并且通常可以在符合条件的云资源之间灵活调整。

你可以把它理解成一个可以灵活更换绑定对象的公网网络地址。

## 一、为什么需要 EIP？

假设你购买了一台云服务器：

云服务器 ECS

# 192.168.1.10

内网 IP

没有公网 IP，外部互联网无法直接通过这个地址访问

这时候，你想让别人通过互联网访问你的服务器，就需要公网 IP。

云厂商可以给你分配一个 EIP：

弹性公网 IP（EIP）

# 123.123.123.123

公网 IP

云服务器 ECS

`192.168.1.10`

绑定之后，外部用户就可以通过：

```
http://123.123.123.123
```

访问你的服务器（前提是安全组、防火墙和 Web 服务都配置正确）。

## 二、EIP 最重要的特点是什么？

它最重要的特点就是：公网 IP 和云服务器可以相对独立地管理。

举个例子。

假设你有两台云服务器：

|服务器|内网 IP|
|---|---|
|ECS-A|192.168.1.10|
|ECS-B|192.168.1.20|

你有一个 EIP：

`123.123.123.123`

原来绑定 ECS-A。

后来 ECS-A 出现故障，你希望把公网 IP 转移到 ECS-B。

在云厂商支持的网络架构和绑定条件下，可以解绑 EIP，再绑定到 ECS-B。

这样外部用户仍然可以通过同一个公网 IP 访问新的服务器。

[High-Availability Virtual IP Address Overview--Virtual Private Cloud-Byteplus](https://images.openai.com/static-rsc-4/nJyc-gnPLuLt9QFYC1OpY9l6wVaCwG14Lg2wpazljnNg6_5JgFCzishdKzRXc3x6su48M3pcK6c2428ftdABGZ57UYoPu7rqipM1MeB6cP8UiPvctsTRE35KzIYcEaLlDFbTAYvTOTJB1-4aqT-MwMVyUgAMKCczBxM4wokrhiQ?purpose=inline)

[docs.byteplus.com](https://docs.byteplus.com/en/docs/vpc/HAVIP-overview?utm_source=chatgpt.com)

这就是为什么叫 Elastic IP（弹性公网 IP）。