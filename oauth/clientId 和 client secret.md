### 它们分别是什么？

可以把它类比成应用程序的“用户名”和“密码”：

- **Client ID（客户端 ID）**：相当于应用程序的**公开用户名**或**身份证号**。它是你在服务商平台注册应用时生成的一串唯一标识符，用于告诉服务器“是哪个应用在请求访问”。[-7](https://docs.oracle.com/en-us/iaas/Content/Identity/api-getstarted/OATOAuthClientWebApp.htm?source=%3Aow%3Ams%3Apt%3A%3A%2C%3Aow%3Ams%3Apt%3A%3A#1)
    
- **Client Secret（客户端密钥）**：相当于应用程序的**私密密码**。它是一串高度机密的字符串，用于向服务器证明“这个应用确实是它声称的那个应用”，而不是别人伪造的。[-7](https://docs.oracle.com/en-us/iaas/Content/Identity/api-getstarted/OATOAuthClientWebApp.htm?source=%3Aow%3Ams%3Apt%3A%3A%2C%3Aow%3Ams%3Apt%3A%3A#1)
    

### 它们是干嘛用的？

主要用在 **OAuth 2.0 授权流程**中。当你的应用需要访问某个受保护的资源（比如读取云存储里的文件、调用 API）时，流程大致如下：

1. 你的应用把 **Client ID** 和 **Client Secret** 一起发给认证服务器。
    
2. 服务器验证这对凭证是否匹配且有效。
    
3. 验证通过后，服务器会返回一个临时的 **Access Token（访问令牌）**。
    
4. 你的应用拿着这个 Token 去请求具体的资源。
    

如果缺少了这对凭证，或者 Client Secret 泄露了，服务器就无法确认请求的合法性，也就不会发放 Token