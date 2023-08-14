---
title: nginx
tags: nginx
---

## 命令

```
start nginx 开启
nginx -s stop 停止
nginx -s reload 重新加载
```

## location的匹配规则

* = 表示精确匹配。只有请求的url路径与后面的字符串完全相等时，才会命中。
* ^~ 表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找。
* ~ 表示该规则是使用正则定义的，区分大小写。
* ~* 表示该规则是使用正则定义的，不区分大小写。

注意的是，nginx的匹配优先顺序按照上面的顺序进行优先匹配，而且注意的是**一旦某一个匹配命中直接退出，不再进行往下的匹配**

剩下的普通匹配会按照**最长匹配长度优先级来匹配**，就是谁匹配的越多就用谁。

## history模式、跨域、缓存、反向代理

```
# html设置history模式
location / {
    index index.html index.htm;
    proxy_set_header Host $host;
    # history模式最重要就是这里
    try_files $uri $uri/ /index.html;
    # index.html文件不可以设置强缓存 设置协商缓存即可
    add_header Cache-Control 'no-cache, must-revalidate, proxy-revalidate, max-age=0';
}

# 接口反向代理
location ^~ /api/ {
    # 跨域处理 设置头部域名
    add_header Access-Control-Allow-Origin *;
    # 跨域处理 设置头部方法
    add_header Access-Control-Allow-Methods 'GET,POST,DELETE,OPTIONS,HEAD';
    # 改写路径
    rewrite ^/api/(.*)$ /$1 break;
    # 反向代理
    proxy_pass http://static_env;
    proxy_set_header Host $http_host;
}

location ~* \.(?:css(\.map)?|js(\.map)?|gif|svg|jfif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
    # 静态资源设置七天强缓存
    expires 7d;
    access_log off;
}
```

[Nginx反向代理解决前端跨域问题](https://juejin.cn/post/7008446203562033188)

[前端解决跨域，nginx 反向代理](https://juejin.cn/post/7021433082691452936)