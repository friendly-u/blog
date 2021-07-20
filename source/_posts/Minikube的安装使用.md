---
layout: post
title: Minikube的安装使用
date: 2021-07-20 14:14:42
tags: k8s
---

# Minikube安装使用
在做k8s开发的时候受限于本地的性能以及复杂度不能搭建一个完整的k8s集群，这个时候需要minikube来搭建k8s开发环境

#### 下载安装
阿里云版本地址,官方版本地址,推荐阿里云版本
#### 下载阿里云版本二进制文件
Macos
```
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.13.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
Linux
```
curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.14.2/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
#### 验证安装
执行minikube version验证安装

#### 启动Minikube
```
minikube start --driver=docker --image-mirror-country cn
```
这样就启动一个使用docker作为驱动的minikube，稍等一会就会启动成功，并且将kubectl设置为minikube
再次启动是只需要执行minikube start即可

#### 常用命令
```
minikube start 启动集群

minikube stop 停止集群

minikube delete 删除集群

minikube dashboard 打开k8s报表

minikube status 查看minikube状态

minikube ssh 登录到minikube节点上
```

#### 常见问题
如果在 Mac 上安装 minikube 不成功，可以：

1. stop docker(退出应用即可)

2. rm ~/Library/Containers/com.docker.docker

3. 启动docker后运行minikube start