---
layout: post
title: Golang连接Shell
date: 2021-07-19 18:40:36
tags: golang
---
Golang 连接 Redis
<!-- more -->
```go
func main() {
        command := `./git-back.sh .`
        cmd := exec.Command("/bin/bash", "-c", command)

        err := cmd.Run()
        if err != nil {
                fmt.Println("Execute Command failed:" + err.Error())
                return
        }

        fmt.Println("Execute Command finished.")
}
```