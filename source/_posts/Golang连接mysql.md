---
layout: post
title: Golang连接Mysql
date: 2021-07-19 17:47:04
tags: golang
---

Golang 连接 Mysql
<!-- more -->

注意：如果不使用 dn.Ping()，无法验证是否可以 ping 通 mysql
```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/go-sql-driver/mysql"
)

const (
    username = "root"
    passward = "123456"
    network  = "tcp"
    host     = "113.31.106.132"
    port     = 3306
    database = "lee"
)

func main() {
    dsn := fmt.Sprintf("%s:%s@%s(%s:%d)/%s", username, passward, network, host, port, database)

    db, err := sql.Open("mysql", dsn)

    if err != nil {
        fmt.Printf("Open mysql failed,err:%v\n", err)
        return
    }
    defer db.Close()
    err = db.Ping()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Open mysql succeed")
    return
}
```