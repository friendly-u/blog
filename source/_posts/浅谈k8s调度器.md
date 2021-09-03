---
layout: post
title: 浅谈k8s调度器
date: 2021-09-03 23:36:42
tags: 
- k8s
---
kube-scheduler 目前常见的修改方法有extender和framework，其中extender较为简单，framework今后将为主流。
<!-- more -->
# extender

#### 环境说明
通过kubeadm搭建的k8s版本 v1.15.2
```
yum install -y kubelet-1.15.2-0 kubeadm-1.15.2-0 kubectl-1.15.2-0
sudo kubeadm init --kubernetes-version=v1.15.2
```
#### 目标
使k8s在为pod选择节点时经过自己写的extender

#### 创建自定义调度 yaml(/etc/kubernetes/scheduler-config.yaml)
```yml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/etc/kubernetes/scheduler.conf"      # kubeadm 默认将kube-scheduler对config文件放在该路径，不用更改
algorithmSource:
  policy:
    file:
      path: "/etc/kubernetes/scheduler-policy.yaml"  # 指定自定义调度策略文件
```
#### 创建自定义调度文件 (/etc/kubernetes/scheduler-policy.yaml)
```yaml
apiVersion: v1
kind: Policy
extenders:
- urlPrefix: "http://127.0.0.1:8888/" # extender 的监听地址，后面会介绍对应go代码
  filterVerb: "filter"                # extender filterVerb 路径
 # prioritizeVerb: "prioritize"       # 本文只以filterVerb作为示例
  weight: 1
  enableHttps: false
```
#### kube-scheduler 设置
因为是kubeadm搭建，默认kube-scheduler为静态pod创建，我们需要修改其启动参数，对启动参数增加config配置（默认config参数为空。同时修改了原有的挂载路径,将/etc/kubernetes路径挂载到pod（目的是将scheduler.conf、scheduler-config.yaml、scheduler-policy.yaml给pod使用）。
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --bind-address=127.0.0.1
    - --config=/etc/kubernetes/scheduler-config.yaml
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.15.2
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10251
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    volumeMounts:
    - mountPath: /etc/kubernetes
      name: k8s-dir
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes
    name: k8s-dir
status: {}
```
#### golang代码示例
```go
package main

import (
	"net/http"

	log "github.com/gogap/logrus"
	"github.com/julienschmidt/httprouter"
)

func main() {
	router := httprouter.New()
	router.POST("/filter", filter)
	log.Fatal(http.ListenAndServe(":8888", router))
}

func filter(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
	log.Info("=================================")
	log.Info("start filter")
}
```

* 编译：GOOS=linux GOARCH=amd64 CGO_ENABLED=0 GOFLAGS=-mod=vendor go build -o scheduler main.go
在master运行scheduler二进制文件，同时创建pod会看到如下日志。
```go
INFO[0161] =================================
INFO[0161] start filter
```