---
layout: post
title: Docker容器/etc/hosts文件被重置Bug
index_img: /images/post/bug-diary-cover.png
date: 2022-08-16 16:13:34
categories:
- Bug日记
tags:
- Docker
- Bug
---

## 1. Bug发现

前些日子，公司有个将Awvs14打包为一个独立的Docker容器并暴露接口和数据库给其他服务做测试的需求

出于测试目的，便在 [🔰雨苁ℒ🔰](https://www.ddosi.org/) 大佬的博客中找了个AWVS14的Linux破解版

具体的破解教程以及破解文件等可以参考：[Awvs破解版14.7.220228146|awvs cracked](https://www.ddosi.org/awvs-14-7/)，本篇日记只专注Bug记录

根据大佬提供的安装教程一路走下来顺顺利利安装完成

接下来自然而然的就是导出容器推送到服务器进行测试

> 小提示：
>
> 容器内安装Awvs需要安装Awvs所需的依赖，可以根据Awvs版本去查看官方的文档安装依赖，在本篇日记中容器使用的是 `ubuntu20.04` 基础镜像
> 
> Awvs的依赖安装如下：
>
> ```shell
> apt install -y libxdamage1 libgtk-3-0 libasound2 libnss3 libxss1 libx11-xcb1 libxcb-dri3-0 libgbm1 libdrm2 libxshmfence1 sudo
> ```

但是推送到服务器后使用 `docker run` 命令启动容器时怪事发生了，扫描任务一直提示启动失败

排除Awvs容器的输出发现每当任务启动时都会提示许可证错误，起初以为是破解文件的问题（因为安装后就直接打包推送到服务器测试，没有在本地启动测试过）

于是重新构建一个容器，在本地测试没问题后推送到服务器，但任务启也是一直失败，错误原因也同样是许可证错误

........

有怀疑过是网络问题，有怀疑过Docker版本问题，有怀疑过是Awvs许可证与Mac地址相关等等，耽搁了一个周，焦头烂额准备放弃之际打算对比下本地容器和测试服务上的容器内部文件有什么不一样

之前从未怀疑过是容器文件不一致导致的Bug，因为在我的认知里，容器导出后再导入再运行内部文件是不会变化的，就像传统虚拟机一样

但是对比过后，发现测试服务器上的 `/etc/hosts` 文件与本地打包的容器不一致

根据大佬的 Awvs14 Linux 版安装教程，在第6步时需要修改 `/etc/hosts` 文件

![](images/post/docker-container-etc-hosts-file-reset-bug/01.png)

这一步在正常的Linux操作系统上操作是没问题的，所以理所当然的以为打包后的 `/etc/hosts` 文件也不会改变

由于 `/etc/hosts` 中与破解相关的内容被清除了，所以AWVS任务因为许可证问题启动失败也是情理之中了


## 2. 为什么 /etc/hosts 会被重置？

参考Docker官方文档：[https://docs.docker.com/engine/reference/run/#managing-etchosts](https://docs.docker.com/engine/reference/run/#managing-etchosts)

![](images/post/docker-container-etc-hosts-file-reset-bug/02.png)

> 你的容器在/etc/hosts中会有几行，它们定义了容器本身的主机名以及localhost和其他一些常见的东西。--add-host标志可以用来向/etc/hosts添加额外的行。
>
> 如果一个容器被连接到默认的桥接网络并与其他容器链接，那么该容器的/etc/hosts文件就会用被链接的容器的名称进行更新。
>
> 由于Docker可能会实时更新容器的/etc/hosts文件，所以可能会出现这样的情况：容器内的进程最终会读取一个空的或不完整的/etc/hosts文件。在大多数情况下，再次重试读取应该可以解决问题。

也就是说Docker会对 `/etc/hosts` 文件进行管理并且会不定时更新

我猜测是我启动测试容器时docker会检查网络并更新 `/etc/hosts` 文件导致破解内容被清除导致


## 3. Bug解决

### 3.1 --add-host

> 如果你只是对已有镜像做简单修改，建议使用此种方式

创建容器时使用 `docker run` 命令的 `--add-host` 参数指定 /etc/hosts 内容

命令格式：

```shell
# host : 主机名称
# ip   : 主机IP地址
docker run --add-host host:ip
```


### 3.2 使用启动脚本

> 如果你使用Dockerfile构建镜像且使用启动脚本做初始化操作时建议使用此种方式

第二种方式是构建镜像时指定一个启动脚本，每次启动容器前检查一次 `/etc/hosts` 文件内容完整性

以AWVS破解教程为基础，以下为一个shell启动脚本例子：

该启动脚本会在容器启动时执行，脚本会检查 `/etc/hosts` 文件中是否存在破解内容

127.0.0.1 updates.acunetix.com
127.0.0.1 erp.acunetix.com

```bash
#!/bin/bash

# 每次启动前都检查 /etc/hosts 文件, 确保屏蔽了AWVS更新检查
# 防止容器迁移后 /etc/hosts 文件内容丢失造成的扫描任务启动失败
host1CheckResult=`cat /etc/hosts | grep updates.acunetix.com | tr -s [:space:]`
host2CheckResult=`cat /etc/hosts | grep erp.acunetix.com | tr -s [:space:]`
if [ -z "$host1CheckResult" ];then
   echo "127.0.0.1 updates.acunetix.com" >> /etc/hosts
fi
if [ -z "$host2CheckResult" ];then
    echo "127.0.0.1 erp.acunetix.com" >> /etc/hosts
fi

# 启动AWVS
su - acunetix -c "cd /home/acunetix/.acunetix/ && ./start.sh"
```

将此脚本保存为 `start.sh`


## 4. 一些参考命令 

### 4.1 将启动脚本附加到已有容器 

将脚本复制到容器中 

```shell
docker cp start.sh 容器名称:脚本存储目录
# 例子
docker cp start.sh awvs-14:/root/
```

给脚本添加执行权限 

```shell
docker exec 容器名称 chmod +x 脚本路径
# 例子
docker exec awvs-14 chmod +x /root/start.sh
```

停止容器

```shell
docker stop 容器名称
# 例子
docker stop awvs-14
```

将容器提交为新镜像

```shell
docker commit 容器名称 镜像名称:tag
# 例子
docker commit awvs-14 awvs:14
```

以提交的镜像作为基础启动一个新镜像，并以 `start.sh` 作为启动脚本

```shell
docker run -itd --name awvs-14-new awvs:14 /root/start.sh
```

### 4.2 使用Dockerfile构建并启动

```dockerfile
FROM ubuntu:20.04

WORKDIR /root/
COPY start.sh ./

RUN chmod +x ./start.sh

CMD ./start.sh
```

构建镜像

```shell
docker build -t awvs:14 .
```

以镜像为基础启动容器

```shell
docker run -itd --name awvs-14-new awvs:14 
```