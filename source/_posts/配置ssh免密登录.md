---
layout: post
title: 配置ssh免密登录
index_img: /images/post/ssh-cover.png
date: 2022-11-05 22:54:46
categories:
- 开发
- 常用配置
tags:
- 运维
- ssh
---

> **环境**
> 
> 操作系统:
>  - Windows11: 开发者笔记本（后续使用 `Client` 代替）
>  - Ubuntu Server 20.04: 开发环境（后续使用 `Server` 代替）

> **实现目标**
>
> `Client` ssh免密登录 `Server`，免密登录用户为 `mf`


## 1、`Client` 创建 ssh 密钥对

启动一个 powershell 并使用以下命令生成密钥对

```shell
ssh-keygen -t rsa
```

`-t` 参数指定密算法，本例使用 RSA 算法

生成过程中会提示输入 密钥文件名称，如果不指定则使用默认名称 `id_rsa`，存储路径为 `{家目录}/.ssh/id_rsa`


```shell
Enter file in which to save the key (C:\Users\Rebel/.ssh/id_rsa):
```

不建议使用默认名称，可能会覆盖掉已有的密钥文件

本例将使用 `oksec_development_environment_rsa` 作为密钥文件名称，你可以根据自己的喜好命名任何一个名称

**注意：** 请使用绝对路径指定密钥文件路径

手动输入密钥文件名称后一路回车即可生成一对密钥文件，你可以看到类似以下的输出：

```shell
PS C:\Users\Rebel\.ssh> ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Rebel/.ssh/id_rsa): C:\Users\Rebel/.ssh/oksec_development_environment_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in C:\Users\Rebel/.ssh/oksec_development_environment_rsa.
Your public key has been saved in C:\Users\Rebel/.ssh/oksec_development_environment_rsa.pub.
The key fingerprint is:
SHA256:81v1bxsms7YIJ3omeFyf9q1657U5GOufP+7zjblQnIs rebel@mangfu
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|                 |
|             . . |
|        S     =  |
|         +   = o |
|      o .oo.Eo=oo|
|     . +.o+*.**BX|
|      ..+ oo**%&%|
+----[SHA256]-----+
```

此时会得到两个密钥文件：

- oksec_development_environment_rsa：私钥文件
- oksec_development_environment_rsa.pub：公钥文件


## 2、将 `Client` 公钥添加到 `Server`

使用用户 `mf` 登录 `Server`

将 `Client` 生成的公钥（也就是 `oksec_development_environment_rsa.pub` 文件的内容）添加到 `Server` 的 `{家目录}/.ssh/authorized_keys` 文件中（如果没有 `authorized_keys` 文件，请创建）

这是本例添加后的结果：

```shell
mf@mf:~/.ssh$ pwd
/home/mf/.ssh
mf@mf:~/.ssh$ cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDL+7Yg4Ztb1AkZ321rXbtP1lBWeJG+LUNFalyE9ElVAQhwNNQLuUuiyHbVuSFQlEGZPyvG4+CtFOVnD2mINZIx7dnOVkhYFyZvq5cMEvzr7gFFDApyvk2roOPkrAskGILtfdIDs30qZTDvrrMGztdcALSK1qhLvj3WAHIhUsjfrlBYR5kts3iVCMwvL/a7J5yIVXUvZe3aNgw83A7vYF3oPoTqdMjJUSbx14kyWQ0Mthc8YfzM1i+EOe3YFwydxpXK5JTVXpgucm0SdFEBevsgWMKguxITN6ELRNBGihhgwQEiiGOnRBFpoXti38gPHoWJXwtOuHrE/lv2DjDqDsyE3jWX6Z6WGSPAZROCxwx7dyP0BGuo2MhTCCiBd2MkQ207ItLu2BqrsS2RUaaZ1OjV5UdOURBe2bxqzxs0toE91qwL9+o3mllWKGD3Rfd1Nl6d5i4u3Ec1UqVtmEvnzfmVaYew/GKtoMfrONYknhkVQqjCwOKd8hya+MyvFOcnIHU= rebel@mangfu

mf@mf:~/.ssh$ 
```



## 3、`Client` 配置登录

修改 `Client` 的 `{家目录}/.ssh/config` 文件（如果没有请手动创建）

写入以下内容：

```yaml
Host {连接名称}
    HostName {Server的IP地址}
    PreferredAuthentications publickey
    IdentityFile {Client私钥文件路径}
    User {登录用户}
```

- {连接名称}：连接名称为 ssh 连接时的名称，可以任意命名，但不建议包含特殊符号
- {Server的IP地址}：Server的IP地址
- {Client私钥文件路径}：步骤一中生成的私钥文件的路径
- {登录用户}：免密登录的用户

以下是本例的配置：
```yaml
# 内网开发环境
Host 公司内网开发环境
    HostName 192.168.0.15
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/oksec_development_environment_rsa
    User mf
```


## 4、测试

在 `Client` 起一个 powershell ，尝试使用 ssh 免密登录 `Server`

```shell
ssh {连接名称}
```

本例中的连接名称为 `公司内网开发环境`

```
ssh 公司内网开发环境
```

不出意外的话，你已经成功进入 `Server` 的控制台了

![](images/post/335ef5dc5d2211ed9c9b9335f43ccaf2.png)