---
layout: post
title: Golang字符串操作
date: 2021-07-19 18:41:40
tags: golang
---
Golang字符串操作
<!-- more -->

#### 包含
```go
strings.Contains("I'll chop up this carrot", "carrot")
```
包含返回true

#### 替换
```go
strings.ReplaceAll("one night of love", "one", "all")
```

#### 保留两位小数
```go
func decimal(value float64) float64 {
	return math.Trunc(value*1e2+0.5) * 1e-2
}

```

#### 分割字符串
```go
s1 := "Shello-worldP"
s2 := s1[1 : len(s1)-1]
echo: hello-world
```

#### 字符串大小写
```go
// ToUpper 将 s 中的所有字符修改为其大写格式
us := strings.ToUpper(s)
// ToLower 将 s 中的所有字符修改为其小写格式
ls := strings.ToLower(s)
```

#### 切割字符串
```go
# strings.Split()函数用于将指定的分隔符切割字符串，并返回切割后的字符串切片。

demo := "I&love&Go,&and&I&also&love&Python."
string_slice := strings.Split(demo, "&")
fmt.Println("result:",string_slice)
# result: [I love Go, and I also love Python.]
```