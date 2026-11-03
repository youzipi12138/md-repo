Linux 中的 NFS 是 Network File System（网络文件系统） 的缩写。

简单来说，NFS 就是让一台 Linux 服务器把自己的目录共享出来，其他 Linux 服务器通过网络访问这个目录，就像访问本地磁盘上的目录一样。

你可以把它理解为：Linux 系统之间的网络共享文件夹。

## 1. NFS 是干什么的？

假设你有两台 Linux 服务器：

- 服务器 A：IP 地址 `192.168.1.100`
    
- 服务器 B：IP 地址 `192.168.1.200`
    

服务器 A 上有一个目录：

bash

```
/data/share
```

现在你希望服务器 B 也能访问这个目录中的文件。

如果使用 NFS，就可以实现：

![NFS Server and client configuration](https://images.openai.com/static-rsc-4/6pNokIIe-w69AiWoUGU9WsT5Ch1b75_OaTTME-icosUT92ozHYBEi176Ea1ltqIkh1iaECpdg2W1Db1yw8ZjTIJnKZkv2fDTgu5vmwf1fNPSUKKWQGNlgukTrkIYxK1VpzyLiJxePyS3Rx22yp7A94Jc0t_zt53J6gUUWaBeuuo?purpose=inline)

服务器 A 将 `/data/share` 目录共享出来。

服务器 B 通过 NFS 将它挂载到自己的 `/mnt/nfs` 目录。

这样，服务器 B 执行：

bash

```
cd /mnt/nfs
ls
```

看到的就是服务器 A 的 `/data/share` 目录中的文件。

甚至在服务器 B 上创建文件：

bash

```
touch /mnt/nfs/test.txt
```

这个文件实际上就存储在服务器 A 的 `/data/share` 目录中。

## 2. NFS 中的两个重要角色

NFS 主要分为两个角色：

|角色|含义|作用|
|---|---|---|
|NFS Server|NFS 服务端|提供共享目录|
|NFS Client|NFS 客户端|挂载并访问共享目录|

记住：

- Server 负责共享文件。
    
- Client 负责访问文件。
    

## 3. Linux 中如何使用 NFS？

我们通过一个简单的例子来理解。

假设：

- NFS 服务端 IP：`192.168.1.100`
    
- NFS 客户端 IP：`192.168.1.200`
    
- 服务端共享目录：`/data/share`
    
- 客户端挂载目录：`/mnt/nfs`
    

### 第一步：服务端安装 NFS

以 CentOS/RHEL 为例：

bash

```
yum install -y nfs-utils
```

Ubuntu/Debian 可以使用：

bash

```
apt install -y nfs-kernel-server
```

### 第二步：创建共享目录

bash

```
mkdir -p /data/share
```

### 第三步：配置 NFS 共享

编辑配置文件：

bash

```
vim /etc/exports
```

添加：

bash

```
/data/share 192.168.1.200(rw,sync,no_subtree_check)
```

简单解释：

|参数|含义|
|---|---|
|`/data/share`|要共享的目录|
|`192.168.1.200`|允许访问的客户端 IP|
|`rw`|允许读写|
|`sync`|同步写入|
|`no_subtree_check`|不进行子目录检查|

### 第四步：启动 NFS 服务

bash

```
systemctl enable --now nfs-server
```

使配置生效：

bash

```
exportfs -rav
```

### 第五步：客户端挂载 NFS

客户端安装：

bash

```
yum install -y nfs-utils
```

创建挂载目录：

bash

```
mkdir -p /mnt/nfs
```

执行挂载：

bash

```
mount -t nfs 192.168.1.100:/data/share /mnt/nfs
```

查看挂载情况：

bash

```
df -h
```

此时客户端就可以访问服务端共享的文件了。

## 4. NFS 常用命令

|命令|作用|
|---|---|
|`showmount -e IP`|查看 NFS 服务端共享目录|
|`mount -t nfs`|挂载 NFS 目录|
|`umount`|卸载 NFS 目录|
|`exportfs -rav`|重新加载共享配置|
|`exportfs -v`|查看当前共享配置|
|`df -h`|查看文件系统挂载情况|
|`systemctl status nfs-server`|查看 NFS 服务状态|

## 5. NFS 有什么实际应用场景？

在实际 Linux 运维和企业服务器环境中，NFS 很常见。

例如：

1. 多台服务器共享文件：多台服务器访问同一份文件，不需要每台服务器都保存一份。
    
2. 共享应用程序数据：多台 Web 服务器共享上传文件、图片等。
    
3. 容器存储：Kubernetes 可以通过 NFS 提供共享存储。
    
4. 备份存储：将备份文件统一存放在 NFS 服务器上。
    

## 6. 最后，理解 NFS 最重要的一点

我用一句话帮你记住：

NFS 就是通过网络，把远程 Linux 服务器上的目录挂载到本地 Linux 系统中，让远程目录看起来像本地目录一样使用。

再记住下面三个概念：

- 共享（Export）：服务端把目录共享出来。
    
- 挂载（Mount）：客户端把远程目录挂载到本地目录。
    
- 卸载（Umount）：客户端断开这个目录的挂载。