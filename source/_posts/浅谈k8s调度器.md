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
  nodeCacheCapable: true
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
[完成代码地址](https://github.com/friendly-u/scheduler)
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"

	log "github.com/gogap/logrus"
	"github.com/julienschmidt/httprouter"
	v1 "k8s.io/api/core/v1"
	schedulerapi "k8s.io/kubernetes/pkg/scheduler/api"
)

func main() {
	router := httprouter.New()
	router.POST("/filter", filter)
	log.Fatal(http.ListenAndServe(":8888", router))
}

func filter(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
	log.Info("start filter")
	var buf bytes.Buffer
	body := io.TeeReader(r.Body, &buf)
	var extenderArgs schedulerapi.ExtenderArgs
	var result schedulerapi.ExtenderFilterResult
	defer func() {
		if err := recover(); err != nil {
			result.Error = fmt.Sprintf("%v", err)
		}
		w.Header().Set("Content-Type", "application/json")
		if response, err := json.Marshal(&result); err != nil {
			log.Error(err)
			w.WriteHeader(http.StatusInternalServerError)
		} else {
			w.WriteHeader(http.StatusOK)
			log.Info("response: ", string(response))
			w.Write(response)
		}
	}()
	if err := json.NewDecoder(body).Decode(&extenderArgs); err != nil {
		result = schedulerapi.ExtenderFilterResult{
			Error: err.Error(),
		}
		return
	}

	result = filterNodes(&extenderArgs, fitNodes)
	log.WithFields(log.Fields{
		"result": result}).Debug("Filter done")

}

func filterNodes(args *schedulerapi.ExtenderArgs, f func(*v1.Pod, []string) ([]string, error)) schedulerapi.ExtenderFilterResult {
	var nodes []string
	if args.NodeNames != nil && len(*args.NodeNames) > 0 {
		nodes = *args.NodeNames
	} else if args.Nodes != nil && len(args.Nodes.Items) > 0 {
		for _, node := range args.Nodes.Items {
			log.Info(node.Name)
			nodes = append(nodes, node.GetName())
		}
	}

	var result schedulerapi.ExtenderFilterResult
	fitnodes, err := f(args.Pod, nodes)
	if err != nil {
		result.Error = err.Error()
	}
	// 1. 跟自定义scheduler的策略有关，如果 nodeCacheCapable: true,则响应可省略 result.Nodes。
	// 2. 如果响应需要result.Nodes，设置nodeCacheCapable: false。
	// 第二种参考：https://www.qikqiak.com/post/custom-kube-scheduler/
	result.NodeNames = &fitnodes
	return result
}

func fitNodes(pod *v1.Pod, nodes []string) (out []string, err error) {
	if pod.Labels["want"] == "you" {
		for _, node := range nodes {
			if node == "10-9-101-66" {
				out = append(out, node)
			}
		}
	}
	return out, nil
}
```