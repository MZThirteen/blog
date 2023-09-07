---
title: nginx-命令
date: 2023-09-07 15:37:40
tags: nginx
categories: nginx
---

## 命令

```
start nginx 开启
nginx -s stop 停止
nginx -s quit 等所有请求结束之后，停止服务器
nginx -s reload 重新加载
nginx -s reopen 重新加载日志文件
nginx -t 检测配置文件是否有错误
nginx -v 版本
```

## 查找Nginx配置文件的位置

```
// 查看nginx进程：可以查看到nginx的进程，但是找不到配置文件
ps -ef | grep nginx

// 检测配置文件位置
nginx -t
```