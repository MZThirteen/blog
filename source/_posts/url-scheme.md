---
title: 快捷指令常用 URL Scheme
date: 2023-11-30 13:19:27
tags: URL Scheme
categories: 前端
---

### 前言

由于苹果的各应用都是在沙盒中，不能够互相之间访问或共享数据。但是苹果还是给出了一个可以在 APP 之间跳转的方法：URL Scheme。简单的说，URL Scheme 就是一个可以让 APP 相互之间可以跳转的协议。每个 APP 的URL Scheme 都是不一样的，如果存在一样的 URL Scheme，那么系统就会响应先安装那个 APP 的URL Scheme，因为后安装的 APP 的 URL Scheme 被覆盖掉了，是不能被调用的。

### iOS

```
// 呼叫电话
tel://电话号码

// 打开相机
camera://

// 打开计算器
calc://

// 蜂窝网络设置
prefs:root=MOBILE_DATA_SETTINGS_ID

// 无线局域网设置
prefs:root=WIFI

// 自动锁定时间修改页面
prefs:root=General&path=AUTOLOCK

// 电池电量
prefs:root=BATTERY_USAGE

// 隐私与安全性
Prefs:root=Privacy

// 定位服务
prefs:root=Privacy&path=LOCATION

// 通知设置
prefs:root=NOTIFICATIONS_ID

// 壁纸设置
prefs:root=Wallpaper

// 语言与地区
prefs:root=General&path=INTERNATIONAL

// 恢复出厂设置
prefs:root=General&path=Reset

// 设置的蓝牙页面
prefs:root=Bluetooth

// 设置的通用页面
prefs:root=General&path=ACCESSIBILITY

// VPN与设备管理
prefs:root=General&path=ManagedConfigurationList

// 关于本机
Prefs:root=General&path=About

// 面容 ID 与密码
prefs:root=TOUCHID_PASSCODE
```

### 微信

```
// 打开微信
weixin://

// 微信扫一扫
weixin://scanqrcode

// 微信付款码 v8.0.42 可用
weixin://widget/pay
```

### 钉钉

```
// 打开钉钉
dingtalk://

// 钉钉扫一扫
dingtalk://dingtalkclient/page/scan
```

### QQ音乐

```
// QQ音乐启动
qqmusic://

// QQ音乐听歌识曲
qqmusic://http://qq.com/ui/recognize
```

### 支付宝

```
// 支付宝小程序 全国住房公积金
alipays://platformapi/startapp?appId=2019060365493242

// 支付宝小程序 货拉拉
alipays://platformapi/startapp?appId=2018061360359342

// 支付宝蚂蚁森林
alipays://platformapi/startapp?appId=60000002

// 支付宝神奇物种
alipays://platformapi/startapp?appId=68687886

// 支付宝神奇海洋
alipays://platformapi/startapp?appId=2021003115672468

// 支付宝蚂蚁新村
alipays://platformapi/startapp?appId=68687809

// 支付宝芭芭农场
alipays://platformapi/startapp?appId=68687599

// 支付宝蚂蚁庄园
alipays://platformapi/startapp?appId=66666674

// 支付宝账单
alipays://platformapi/startapp?appId=20000003

// 支付宝生活缴费
alipays://platformapi/startapp?appId=20000193

// 支付宝手机充值
alipays://platformapi/startapp?appId=10000003

// 支付宝扫一扫
alipays://platformapi/startapp?saId=10000007

// 支付宝付款码
alipayqr://platformapi/startapp?saId=20000056

// 支付宝收款码
alipays://platformapi/startapp?appId=20000123

// 支付宝余额宝
alipays://platformapi/startapp?appId=20000032

// 支付宝芝麻信用
alipayqr://platformapi/startapp?saId=20000118

// 支付宝花呗
alipays://platformapi/startapp?appId=20000199

// 支付宝转账
alipayqr://platformapi/startapp?saId=20000116

// 支付宝提现
alipayqr://platformapi/startapp?saId=20000033

// 支付宝运动
alipayqr://platformapi/startapp?saId=20000869

// 支付宝出行
alipayqr://platformapi/startapp?saId=200011235

// 支付宝滴滴出行
alipay://platformapi/startapp?appId=20000778
```

### 云闪付

```
// 云闪付扫一扫
upwallet://native/scanCode

// 云闪付付款码
upwallet://pay

// 云闪付乘车码
upwallet://rn/rnhtmlridingcode
```

### 美团

```
// 美团扫一扫
imeituan://www.meituan.com/scanQRCode?openAR=1

// 美团付款码
imeituan://www.meituan.com/web?url=https://awp.meituan.com/payfe/union-pay-qrcode/index.html
```

### 知乎

```
// 知乎扫一扫
zhihu://codereader
```

### 高德地图

```
// 打开高德地图
iosamap://

// 高德导航 (里面的坐标可根据需要自行修改)
// t 为出行方式：0 驾车，1 公交，2 步行
// m 为出行要求 
// 当 t 为驾车时：0 速度最快，1 费用最少，2 距离最短，3 不走高速，4 躲避拥堵，
// 5 不走高速且避免收费，6 不走高速且躲避拥堵，7 躲避收费和拥堵，8 不走高速躲避收费和拥堵
// 当 t 为公交时：0 最快捷，2 最少换乘，3 最少步行，5 不乘地铁 ，7 只坐地铁 ，8 时间短
iosamap://path?sourceApplication=日历&sid=BGVIS1&sname=$我的位置&did=BGVIS2&dname=目的地&dlat=纬度&dlon=经度&dev=0&m=0&t=0
```
