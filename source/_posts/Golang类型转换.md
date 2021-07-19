---
layout: post
title: Golang类型转换
date: 2021-07-19 18:45:17
tags: golang
---

Golang类型转换
<!-- more -->

#### string 转 int
```go
int,err := strconv.Atoi(string)  
```
#### string 转 int64  
```go
int64, err := strconv.ParseInt(string, 10, 64)  
//第二个参数为基数（2~36），
//第三个参数位大小表示期望转换的结果类型，其值可以为0, 8, 16, 32和64，
//分别对应 int, int8, int16, int32和int64
```
#### int 转 string
```go
string := strconv.Itoa(int) 
//等价于
string := strconv.FormatInt(int64(int),10)
```
#### int64 转 string
```go
string := strconv.FormatInt(int64,10)  
//第二个参数为基数，可选2~36
//对于无符号整形，可以使用FormatUint(i uint64, base int)
```
#### float 转 string
```go
string := strconv.FormatFloat(float32,'E',-1,32)
string := strconv.FormatFloat(float64,'E',-1,64)
// 'b' (-ddddp±ddd，二进制指数)
// 'e' (-d.dddde±dd，十进制指数)
// 'E' (-d.ddddE±dd，十进制指数)
// 'f' (-ddd.dddd，没有指数)
// 'g' ('e':大指数，'f':其它情况)
// 'G' ('E':大指数，'f':其它情况)
```
#### string 转 float64
```go
float,err := strconv.ParseFloat(string,64)
```
#### string 转 float32
```go
float,err := strconv.ParseFloat(string,32)
```

#### int 转 int64
```go
int64_ := int64(1234)
```

#### []byte 转 string
```go
s := string(b)
```
#### interface{} 转 string
```go
m := make(map[string]interface{})
m["one"] = "hahahha"
s := m["one"]
a := s.(string)
fmt.Println("a: ", a)
```