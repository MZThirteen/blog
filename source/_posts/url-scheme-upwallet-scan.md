---
title: 云闪付的扫一扫 URL Scheme 到底是啥？
date: 2023-11-30 13:24:57
tags:
  - URL Scheme
  - 云闪付
categories: 前端
---

1. 云闪付扫一扫添加至桌面的链接是：https://openysf.cup.com.cn/s/open/addDesktop/createHTMLbase64.html?params=ewogICJhcHBJY29uIiA6ICJodHRwczpcL1wvYmFzZS5jdXAuY29tLmNuXC9zXC93bFwvaWNvblwvbGFyZ2VcL3BheW1lbnRjb2RlX3Nob3J0Y3V0LnBuZyIsCiAgImRlc3QiIDogInNjYW5Db2RlIiwKICAiYXBwTmFtZSIgOiAi5omr5LiA5omrIiwKICAiaG9zdCIgOiAibmF0aXZlIgp9

2. 参数解析出来是：
```
{
  "appIcon" : "https:\/\/wallet.95516.com\/s\/wl\/icon\/large\/paymentcode_shortcut.png",
  "dest" : "scanCode",
  "appName" : "扫一扫",
  "host" : "native"
}
```

3. 获取链接
[upwallet://native/scanCode](upwallet://native/scanCode)

