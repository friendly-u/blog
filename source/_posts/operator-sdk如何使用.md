---
layout: post
title: operator-sdk如何使用
date: 2021-07-20 14:30:49
tags: 开发
---

快速生成operator框架
<!-- more -->

## 环境

operator-sdk版本：

```sh
operator-sdk version
operator-sdk version: "v1.9.0-2-g9fa13f31", commit: "9fa13f31e2b265253c105315ee12cc439e7b4174", kubernetes version: "v1.20.2", go version: "go1.16.2", GOOS: "darwin", GOARCH: "arm64"
```

开发环境：MacOs

## 我们首先进入一个项目，对项目进行初始化

```sh
operator-sdk init --domain example.com
```

## 创建一个资源对象 MultiService

```sh
operator-sdk create api --group multiservice --version v1 --kind MultiService --resource=true --controller=true --namespaced=true
```

## 对Spec字段进行更改，将MultiService结构体改为

```go
type MultiServiceSpec struct {
    Service string `json:"service,omitempty"`
}
```

## 更新CRD文件

```sh
make manifests
```

## 开发

Operator SDK 为我们创建了一个快速启动的代码和相关配置，如果我们要开始处理相关的逻辑，我们可以在项目中搜索TODO(user)这个注释来实现我们自己的逻辑，比如在我的 VSCode 环境中，看上去是这样的：
```go
func (r *MultiService4Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    _ = log.FromContext(ctx)

    // your logic here
    return ctrl.Result{}, nil
}
```

## 部署 crd

```sh
kubectl apply -f config/crd/bases/xx.yaml
```

## 编写 cr

```yaml
# 这里的 multiservice.example.com/v1
# multiservice 是根据 --group multiservice 而来,
# example.com 是根据 operator-sdk init --domain example.com 而来。
# v1 是根据 --version v1 而来。
apiVersion: multiservice.example.com/v1
kind: MultiService
metadata:
  name: service
spec:
  service: hello-world
```

## 监听的 cr 范围

在 main.go 中的 ctrl.NewManage 处可以对其 NameSpace参数进行修改。""代表监听所有。（如果没有可以之间新增NameSpace字段）

## 获取对应事件

```go
// 部署后才会触发Reconcile
func (r *MultiService4Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	reqLogger := ctrl.Log.WithName("reconcile")
    reqLogger.Info("Reconciling MultiService4")

	var err error
	ms4 := &multi4.MultiService4{}

	if err = r.Get(ctx, req.NamespacedName, ms4); err != nil {
		if errors.IsNotFound(err) {
			fmt.Println("Delete")
			return reconciled()
		}
		return requeueWithError(reqLogger, err.Error(), err)
	}
	if ms4.Generation != 1 {
		fmt.Println("Update")
	} else {
		fmt.Println("Add")
	}

	return ctrl.Result{}, nil
}
```

##### 注意

operator-sdk 不同版本之间的命令行参数有很多不同，如果本文与你的环境有差异，请移步[operator-sdk官网](https://sdk.operatorframework.io/)。