云服务器中的 NAT，全称是 Network Address Translation（网络地址转换）。

简单来说：

NAT 就是让一台服务器或者多台服务器，通过转换 IP 地址和端口，实现内网与外网之间通信的一种技术。

你可以把 NAT 理解成一个「网络中转站」或者「快递转发中心」。

我从云服务器的实际使用场景给你讲，这样更容易理解。

## 一、先理解为什么需要 NAT

假设你购买了一台云服务器，里面运行着三个服务：

|服务|内网 IP|端口|
|---|---|---|
|Web 网站|192.168.1.10|80|
|MySQL|192.168.1.10|3306|
|Redis|192.168.1.10|6379|

这里有一个问题：

这些 IP 都是内网 IP，外部互联网不能直接通过这些 IP 访问。

这时候就需要 NAT 来帮助内网和公网之间进行通信。

## 二、NAT 是怎么工作的？

假设：

- 云服务器公网 IP：`123.123.123.123`
    
- 云服务器内网 IP：`192.168.1.10`
    

用户访问：

`123.123.123.123:8080`

NAT 可以将请求转发到：

`192.168.1.10:80`

这个过程叫做 DNAT（目标地址转换）。

[Destination IP NAT 쉽게 이해하기](https://images.openai.com/static-rsc-4/Ezm_GoGIa79JzTZGaZcEcnsqRckVR6xmAsWiCdHaJyhToIKoNQrhfYhzWQ3izxkhDO013d2e-JMlzDiW4HR6y_iy0tx_8KBvaFaXrXjwKjUaIW7HthuLXw3K1gS8xO5uGgEm62-hoZ0vxom7k30fRD-KzH6s108WUfHJ5HitKRo?purpose=inline)

[aws-hyoh.tistory.com](https://aws-hyoh.tistory.com/167?utm_source=chatgpt.com)

也就是说：

公网用户访问公网 IP 和端口，NAT 帮你把请求转发到真正提供服务的内网服务器。

## 三、NAT 最重要的两种类型

云服务器中最常见的是 SNAT 和 DNAT。

### 1. SNAT：源地址转换

英文：Source Network Address Translation。

作用：让内网服务器主动访问互联网。

举个例子：

你的云服务器没有公网 IP，只有一个内网 IP：

`192.168.1.10`

它想访问百度：

`www.baidu.com`

但是互联网无法直接识别和访问这个内网 IP。

于是 NAT 将源 IP 转换成公网 IP：

内网服务器

`192.168.1.10`

SNAT 转换

源 IP 替换成公网 IP

NAT 网关

`123.123.123.123`

互联网

`www.baidu.com`

百度看到的访问来源是 NAT 网关的公网 IP，而不是服务器的内网 IP。

百度返回数据时，NAT 再把数据转回原来的内网服务器。

这就是 SNAT。

常见用途：

- 内网服务器访问互联网。
    
- 内网服务器下载软件。
    
- Docker 容器访问外部网络。
    
- 多台云服务器共用一个公网 IP 出网。
    

### 2. DNAT：目标地址转换

英文：Destination Network Address Translation。

作用：让外部用户访问内网服务器。

例如，你有一台云服务器：

- 公网 IP：`123.123.123.123`
    
- 内网 IP：`192.168.1.10`
    
- 网站端口：`80`
    

你配置了 DNAT 规则：

|公网访问地址|转发到内网地址|
|---|---|
|123.123.123.123:8080|192.168.1.10:80|
|123.123.123.123:3307|192.168.1.10:3306|

当用户访问：

`123.123.123.123:8080`

NAT 网关就会将请求转发到：

`192.168.1.10:80`

这就是 DNAT，也经常被称为端口映射或端口转发。

[iptables | SAILOR](https://images.openai.com/static-rsc-4/cQgTiCCrQGeHLYaQmoBA6z_Bddv3pl833Tjak6YmMJocQUjBwVCrxfZGUbWfssWrSUMi3UW7emCAPn8cMKnG-0xwNougJE3XBCfxgfwcQLNq0MDUj-P2K2crmE4qZYHNhtA0bVPMqeN8vMQ-ENr2ZxJ8bTN1ulShwSZLMiEMeO4?purpose=inline)

[90vfan.github.io](https://90vfan.github.io/2019/08/26/system_and_network/iptables/?utm_source=chatgpt.com)

常见用途：

- 公网访问内网 Web 服务。
    
- 将公网端口映射到内网服务器。
    
- 多台内网服务器共用一个公网 IP 对外提供服务。
    

## 四、云服务器中的 NAT 网关是什么？

你在阿里云、腾讯云、华为云等平台上，可能会看到一个产品叫：

NAT 网关（NAT Gateway）

它其实就是云厂商提供的一种网络地址转换服务。

[Connect Amazon ECS applications to the internet - Amazon Elastic Container Service](https://images.openai.com/static-rsc-4/16losx65UVrf_haZAXPD_iUSJGBd5DWqRw-an5szz7XhaI8smMn8lZHQ9gqYSPUtGoh_oTds-JrO-HH7yr-3-xPZ22w55tJNuPLmyPUZ0_EI0R98zgDZFJI4CWc39MS5yRc9Zcs8j84msC20AUmqJOI5DuqO2SGidi2DFaqnjZw?purpose=inline)

[docs.aws.amazon.com](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-outbound.html?utm_source=chatgpt.com)

假设你有一个云服务器集群：

|服务器|内网 IP|公网 IP|
|---|---|---|
|Web 服务器|192.168.1.10|无|
|Java 服务器|192.168.1.11|无|
|MySQL 服务器|192.168.1.12|无|

你可以购买一个 NAT 网关，绑定一个弹性公网 IP（EIP）。

这样：

1. 三台服务器可以通过 NAT 网关访问互联网。
    
2. 三台服务器可以共用一个公网 IP 出网。
    
3. 不需要给每台服务器单独分配公网 IP。
    
4. 可以通过 DNAT 规则，将公网请求转发到指定内网服务器。
    

需要注意：仅配置 SNAT 并不意味着外网用户就能主动访问内网服务器。 如果需要外部主动访问，通常还需要 DNAT 规则，并配置好安全组和防火墙。

Google Cloud Documentation+2

## 五、SNAT 和 DNAT 的区别

这个表建议记住：

|对比|SNAT|DNAT|
|---|---|---|
|转换对象|源 IP|目标 IP|
|主要方向|内网 → 外网|外网 → 内网|
|主要作用|让内网访问互联网|让外网访问内网服务|
|典型场景|服务器下载软件|公网访问网站|
|常见功能|共享公网 IP|端口映射|

## 六、再举一个实际开发中的例子

假设你搭建了一个 Java 微服务系统：

## 你的云服务器架构

互联网用户

通过公网访问

NAT 网关

公网 IP：123.123.123.123

DNAT + SNAT

Web 服务

`192.168.1.10`

80 端口

Java 服务

`192.168.1.11`

8080 端口

MySQL 数据库

`192.168.1.12:3306`

仅允许内网访问

这个架构中：

- 用户访问网站，通过 DNAT 到达 Web 服务。
    
- Java 服务通过 SNAT 访问互联网，例如调用第三方 API。
    
- MySQL 不需要公网 IP，只允许内网中的 Java 服务访问。
    

这种设计可以减少公网 IP 的使用，也能降低数据库直接暴露在公网的风险。

## 七、最后记住一句话

SNAT：内网出去，换个公网身份。

DNAT：外网进来，找到内网目标。

NAT 网关：负责帮你完成这些地址转换的网络设备或云服务。