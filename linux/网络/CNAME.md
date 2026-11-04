*.mibads.com 通配记录指向 115.190.121.21。 也就是说，任何没有单独配置的子域名都会解析到这台机器。所以如果你要改的是一个子域，必须先确认列表里有没有同名的具体记录；如果有具体记录，就只改那条，通配不会影响它。

### CNAME
CNAME 是 DNS 中一个非常常见的概念，英文全称是 Canonical Name（规范名称），中文通常叫做别名记录。

你可以简单理解为：

CNAME 就是给一个域名设置一个别名，让这个域名指向另一个域名。

我用一个例子给你讲明白。

## 1. CNAME 是干什么的？

假设你有两个域名：

- `www.example.com`
    
- `example.com`
    

你希望用户访问 `www.example.com` 时，实际上指向 `example.com`。

这时候就可以配置一条 CNAME 记录：

www.example.com

CNAME 别名

指向

example.com

目标域名

这样，DNS 查询 `www.example.com` 时，就会告诉客户端继续查询 `example.com` 的 DNS 记录。

## 2. CNAME 和 A 记录有什么区别？

这是最重要的知识点。

|对比|A 记录|CNAME 记录|
|---|---|---|
|指向目标|IPv4 地址|另一个域名|
|示例|`1.2.3.4`|`example.com`|
|作用|域名解析到 IP|域名解析到另一个域名|

举个例子：

A 记录：

```
www.example.com  A       1.2.3.4
```

意思是：

`www.example.com` 对应的 IP 地址是 `1.2.3.4`。

CNAME 记录：

```
www.example.com  CNAME   example.com
```

意思是：

`www.example.com` 是 `example.com` 的别名。

## 3. 实际工作中 CNAME 有什么用？

比如你购买了一个 CDN 服务，CDN 服务商给你提供了一个域名：

```
abc123.cdn-provider.com
```

你自己的网站域名是：

```
static.example.com
```

你可以在 DNS 中配置：

|主机记录|类型|记录值|
|---|---|---|
|static|CNAME|abc123.cdn-provider.com|

那么用户访问：

```
static.example.com
```

DNS 就会继续解析到：

```
abc123.cdn-provider.com
```

这样就可以通过 CDN 服务商提供的节点访问资源。

## 4. CNAME 有几个重要限制

1. CNAME 记录值必须是域名，不能直接填写 IP 地址。
    
2. 同一个域名节点通常不能同时配置 CNAME 和 A、MX 等其他记录。
    
3. 根域名（例如 `example.com`）通常不能直接配置标准 CNAME，因为它还需要承载 SOA、NS 等 DNS 记录。不过部分 DNS 服务商提供 ALIAS、ANAME 或 CNAME Flattening 来实现类似效果。
    
4. CNAME 可以多级指向，但不建议形成循环。
    

## 5. 最后用一句话记住

A 记录：域名 → IP 地址

CNAME：域名 → 另一个域名

如果你正在学习 DNS，我建议接下来理解一下这三个概念：

- A 记录：域名如何找到服务器 IP。
    
- CNAME：域名如何设置别名。
    
- NS 记录：到底由哪台 DNS 服务器负责管理域名。