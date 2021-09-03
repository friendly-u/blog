
---
layout: post
title: Golang web
date: 2021-08-16 17:40:06
tags: golang
---
关于 gin 包的有效使用
<!-- more -->

#### 关于 Post 的使用
```go
package main

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.New()
	r.POST("/posts", func(c *gin.Context) {
		id := c.Query("id")
		page := c.DefaultQuery("page", "0")
		username := c.PostForm("username")
		password := c.DefaultPostForm("username", "000000") // 可设置默认值
		fmt.Println("-------")
		c.JSON(http.StatusOK, gin.H{
			"id":       id,
			"page":     page,
			"username": username,
			"password": password,
		})
	})
	r.Run()
}
```

```sh
$ curl "http://localhost:9999/posts?id=9876&page=7"  -X POST -d 'username=geektutu&password=1234'
{"id":"9876","page":"7","password":"1234","username":"geektutu"}
```