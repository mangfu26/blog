---
layout: post
title: Docker 引擎安装
index_img: /images/post/docker-cover.png
date: 2022-10-08 11:52:18
categories:
- 建设
tags: 
- Docker
---

## 各操作系统的 Docker 引擎安装

### Ubuntu

旧版本的Docker称为 `docker`, `docker.io` 或 `docker-engine` 如果已安装，请卸载它们：

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

在新主机上首次安装 Docker 引擎之前，需要设置Docker存储库。之后可以从存储库安装和更新Docker。
```bash
# 更新apt包索引并安装包，以允许apt通过HTTPS使用存储库
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release 
```

```bash
# 添加Docker的官方GPG密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
# 设置存储库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安装 Docker 引擎
```bash
sudo apt-get update
# 如果不需要 docker compose 编排工具, 可将 docker-compose-plugin 移除
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 非 root 用户使用 docker 

每次使用 Docker 都需要 `sudo` root 权限，有些许不方便，可以配置常用用户使用 Docker

```bash
# 创建 docker 组
sudo groupadd docker

# 将当前用户加入 docker 组, $USER 保存的是当前用户的名称, 需要添加其他用户的将 $USER 
# 替换为目标用户名称
sudo usermod -G docker $USER

# 重启 Docker 服务
sudo service docker restart
```