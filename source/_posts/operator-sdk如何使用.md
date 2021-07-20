---
layout: post
title: operator-sdk如何使用
date: 2021-07-20 14:30:49
tags: 开发
---
## operator-sdk 如何使用
<!-- more -->

##### 环境
operator-sdk版本：
```sh
operator-sdk version
operator-sdk version: "v1.9.0-2-g9fa13f31", commit: "9fa13f31e2b265253c105315ee12cc439e7b4174", kubernetes version: "v1.20.2", go version: "go1.16.2", GOOS: "darwin", GOARCH: "arm64"
```

开发环境：MacOs
##### 我们首先进入一个项目，对项目进行初始化。
```
operator-sdk init
```

##### 创建一个资源对象 MultiService
```
operator-sdk create api --group kun.multiservice --version v1 --kind MultiService --resource=true --controller=true --namespaced=true
```

##### 对Spec字段进行更改，将MultiService结构体改为
```go
type MultiServiceSpec struct {
    Service string `json:"service,omitempty"`
}
```

##### 更新CRD文件：
```
make manifests
```

##### 开发
Operator SDK 为我们创建了一个快速启动的代码和相关配置，如果我们要开始处理相关的逻辑，我们可以在项目中搜索TODO(user)这个注释来实现我们自己的逻辑，比如在我的 VSCode 环境中，看上去是这样的：
```
func (r *MultiService4Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    _ = log.FromContext(ctx)

    // your logic here
    return ctrl.Result{}, nil
}
```

##### 注意
operator-sdk 不同版本之间的命令行参数有很多不同，如果本文与你的环境有差异，请移步[operator-sdk官网](https://sdk.operatorframework.io/)