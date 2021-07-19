---
layout: post
title: Golang连接Email
date: 2021-07-19 18:21:56
tags: golang
---

Golang 连接 Email
<!-- more -->

```go
package main

import (
    "crypto/tls"
    "fmt"
    "strconv"

    "git.ucloudadmin.com/leesin/email/html"
    log "github.com/gogap/logrus"
    "gopkg.in/gomail.v2"
)

func SendMail(mailTo []string, subject string, body string) error {
    //定义邮箱服务器连接信息，如果是阿里邮箱 pass填密码，qq邮箱填授权码
    mailConn := map[string]string{
        "user": "806459794@qq.com",
        "pass": "授权码",
        "host": "smtp.qq.com",
        "port": "25",
    }
    port, _ := strconv.Atoi(mailConn["port"]) //转换端口类型为int
    m := gomail.NewMessage()
    m.SetHeader("From", "yw"+"<"+mailConn["user"]+">") //这种方式可以添加别名，即“XD Game”， 也可以直接用<code>m.SetHeader("From",mailConn["user"])</code> 读者可以自行实验下效果
    m.SetHeader("To", mailTo...)                       //发送给多个用户
    m.SetHeader("Subject", subject)                    //设置邮件主题
    m.SetBody("text/html", body)                       //设置邮件正文
    d := gomail.NewDialer(mailConn["host"], port, mailConn["user"], mailConn["pass"])
    d.TLSConfig = &tls.Config{InsecureSkipVerify: true}
    err := d.DialAndSend(m)
    fmt.Println(err)
    return err
}

func main() {
    subject := "Dobrother 运营日报"
    body := html.LoadHtml() // html.LoadHtml load html, return type string

    err := SendMail(html.MailTo, subject, body)
    if err != nil {
        log.Error("邮件发送失败： ", err)
        return
    }
    for _, v := range html.MailTo {
        log.Info("邮件成功发送至管理员: ", v)
    }
}

```