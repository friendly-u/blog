---
layout: post
title: Golang连接 HTML
date: 2021-07-19 18:20:52
tags: golang
---

Golang 打通 HTML5
<!-- more -->

```go
func main() {
        fs := http.FileServer(http.Dir("yuru/html"))
        http.Handle("/", fs)

        err := http.ListenAndServe(":80", nil)
        if err != nil {
                log.Fatal("ListenAndServe: ", err)
        }
}
```