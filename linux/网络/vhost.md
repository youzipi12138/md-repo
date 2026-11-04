VHost 是 Virtual Host（虚拟主机） 的缩写，中文叫「虚拟主机」。

简单来说：

VHost 就是让一台服务器，通过一个 IP 地址或同一个端口，同时运行多个网站或多个服务，并根据用户访问的信息，把请求分配给不同的网站。

你可以把它理解成：一栋大楼只有一个地址，但是里面有很多家公司，每家公司都有自己的房间。

## 一、为什么需要 VHost？

假设你有一台云服务器：

- 公网 IP：`123.123.123.123`
    
- 服务器端口：`80`
    

现在你想在这台服务器上运行三个网站：

|网站|域名|
|---|---|
|网站 A|`www.baidu.com`|
|网站 B|`www.taobao.com`|
|网站 C|`www.jd.com`|

如果没有 VHost，可能需要给每个网站单独配置 IP 或端口。

但是有了 VHost，三个网站可以共用一个 IP 和一个 80 端口。

[[Nginx] 호스트 기반 라우팅 Host based routing - Jongya’s blog](https://images.openai.com/static-rsc-4/QaNhYFZkJx4mCrnDRaRumCZyXndy55EGb61vuP81SIIsDhHPU_s1F2BHX1AihKTtvQC4Rutn5r1QEF5mJHpKaX7GFgfLgGsjnbabyxb-2P0DruPrZW-h6lSiQE5weX1lC0xhV7NzylWpL52393EU32SvqCISVhKCTB9e3GFU1dU?purpose=inline)

[whdrns2013.github.io](https://whdrns2013.github.io/ops/20250624_001_nginx_host_routing/?utm_source=chatgpt.com)

当用户访问不同域名时，服务器就能识别用户想访问哪个网站。

## 二、VHost 是怎么工作的？

这里以 Nginx 为例。

假设三个域名都解析到了同一个公网 IP：

```
www.baidu.com  → 123.123.123.123
www.taobao.com → 123.123.123.123
www.jd.com     → 123.123.123.123
```

用户访问：

```
http://www.baidu.com
```

浏览器发送 HTTP 请求，其中包含：

```
GET / HTTP/1.1
Host: www.baidu.com
```

Nginx 收到请求后，会读取 HTTP 请求头中的 `Host` 字段。

然后进行匹配：

- `www.baidu.com` → 网站 A
    
- `www.taobao.com` → 网站 B
    
- `www.jd.com` → 网站 C
    

这就是基于域名的虚拟主机（Name-based Virtual Hosting）。

## 三、Nginx 中如何配置 VHost？

例如，你有三个网站：

```
/www/baidu
/www/taobao
/www/jd
```

Nginx 可以这样配置：

```
server {
    listen 80;
    server_name www.baidu.com;

    root /www/baidu;
    index index.html;
}

server {
    listen 80;
    server_name www.taobao.com;

    root /www/taobao;
    index index.html;
}

server {
    listen 80;
    server_name www.jd.com;

    root /www/jd;
    index index.html;
}
```

这里有几个重要配置：

|配置|含义|
|---|---|
|`listen 80`|监听 80 端口|
|`server_name`|指定匹配的域名|
|`root`|指定网站文件目录|
|`index`|默认首页文件|

虽然三个网站监听的都是 80 端口，但是 Nginx 可以通过 `server_name` 区分它们。

## 四、VHost 有哪几种类型？

常见的有三种：

### 1. 基于域名的 VHost

多个域名使用同一个 IP 和端口。

例如：

```
a.example.com → 192.168.1.10:80
b.example.com → 192.168.1.10:80
c.example.com → 192.168.1.10:80
```

通过 HTTP 请求中的 Host 字段区分。

最常见

### 2. 基于 IP 的 VHost

一台服务器拥有多个 IP 地址，不同 IP 对应不同网站。

例如：

```
192.168.1.10 → 网站 A
192.168.1.11 → 网站 B
192.168.1.12 → 网站 C
```

### 3. 基于端口的 VHost

同一个 IP，通过不同端口区分网站。

例如：

```
192.168.1.10:8080 → 网站 A
192.168.1.10:8081 → 网站 B
192.168.1.10:8082 → 网站 C
```