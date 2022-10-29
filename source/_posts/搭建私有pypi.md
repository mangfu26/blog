---
layout: post
title: 搭建私有pypi
index_img: /images/post/pypi-cover.png
date: 2022-10-29 11:42:41
categories:
- 建设
tags: 
- python
- pypi
---
> 环境：
> 操作系统: ubuntu:20.04

## 1、安装 Python3 + pip

```shell
apt update
apt install -y python3 python3-pip
```


## 2、安装 pypiserver

```shell
python3 -m pip install pypiserver
```

如果网络错误，无法访问官方 pypi ，可以使用国内镜像 pypi 站点

例如 阿里云 pypi：[https://mirrors.aliyun.com/pypi/simple/](https://mirrors.aliyun.com/pypi/simple/)

```shell
python3 -m pip install pypiserver -i https://mirrors.aliyun.com/pypi/simple/
```

## 3、无认证方式启动

安装完成后可以使用以下命令启动一个无需 用户/密码 认证的 pypi 服务
```shell
pypi-server
```

pypiserver 默认监听 8080 端口，访问 http://youip:8080 ，如果可以看到欢迎页面，表示服务启动成功

![](images/post/ce25d5b1573d11ed95612cfda1215af5.png)

> 如果你想自定义端口，请使用 -p 参数指定

```shell
pypi-server -p 8000
```

## 4、带认证方式启动

### 4.1、使用 htpasswd 生成密码文件

> htpasswd 命令是 Apache 的 Web 服务器内置工具，用于创建和更新储存用户名、域和用户基本认证的密码文件。

如果提示命令不存在，使用以下命令安装 htpasswd

```shell
apt install -y apache2-utils
```

使用以下命令在当前用户家目录生成密码文件 `.pypipasswd`
认证用户名为 `pypi`

```shell
htpasswd -c ~/.pypipasswd pypi
```

生成过程中提示输入用户密码，请为 `pypi` 用户设置一个访问密码
```shell
root@98fabb4b0b64:~# htpasswd -c ~/.pypipasswd pypi
New password: 
Re-type new password: 
Adding password for user pypi
```

### 4.2、使用密码文件启动带验证的 pypi 服务

```shell
pypi-server -P ~/.pypipasswd 
```

如果报以下错误：

```shell
apache.passlib library is not available. You must install pypiserver with the optional 'passlib' dependency (`pip install pypiserver['passlib']`) in order to use password authentication
```

执行以下命令安装 `pypiserver['passlib']` 后尝试再次启动 pypi 服务

```shell
python3 -m pip install pypiserver['passlib']
```
