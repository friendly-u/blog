---
layout: post
title: 如何写一个systemd服务
date: 2021-07-19 17:40:06
tags: linux
---
如何写一个systemd服务
<!-- more -->
```sh
[Unit]
Description=yuru-web   		# 简单描述服务
After=network.target    # 描述服务类别，表示本服务需要在network服务启动后在启动

[Service]
WorkingDirectory=/root/xiaoxie/web
ExecStart=/root/xiaoxie/web/yuru-web    	# 服务启动命令，命令需要绝对路径（采用sh脚本启动其他进程时Type须为forking）

[Install]
```

#### 模版
```sh
[Unit]
Description=test   		# 简单描述服务
After=network.target    # 描述服务类别，表示本服务需要在network服务启动后在启动
Before=xxx.service      # 表示需要在某些服务启动之前启动，After和Before字段只涉及启动顺序，不涉及依赖关系。

[Service]
Type=forking     		# 设置服务的启动方式
User=USER        		# 设置服务运行的用户
Group=USER       		# 设置服务运行的用户组
WorkingDirectory=/PATH	# 设置服务运行的路径(cwd)
KillMode=control-group  # 定义systemd如何停止服务
Restart=no        		# 定义服务进程退出后，systemd的重启方式，默认是不重启
ExecStart=/start.sh    	# 服务启动命令，命令需要绝对路径（采用sh脚本启动其他进程时Type须为forking）

[Install]
WantedBy=multi-user.target  # 多用户
```
#### 保存目录
```sh
/etc/systemd/system
/usr/lib/systemd/system/vsftpd.service
```

#### 重载
```sh
systemctl daemon-reload
```