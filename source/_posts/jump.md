---
title: 微信小程序、H5、App互相的跳转
date: 2023-09-05 10:11:31
tags:
  - 微信小程序
  - H5
  - App
categories: 前端
---

## 微信小程序跳转H5、小程序、App

### 跳转H5

1. 在微信公众平台-小程序后台配置业务域名，要将校验文件放置到域名根目录下，才可配置成功，通过 **https://url/校验文件名.txt** 可测试成功与否。
2. 使用 **web-view** 配置路径
3. 如需传参通过 **?key=value** 形式一起传输，在另一端接收 URL 上的参数便可。

```
<web-view :src="`https://url/index.html/#/?timestamp=${timestamp}`" />
```

### 跳转小程序

参考 [navigator](https://developers.weixin.qq.com/miniprogram/dev/component/navigator.html)

```
<navigator target="miniProgram" open-type="navigate" app-id="appid" path="pages/index/index">跳转小程序</navigator>
```

### 跳转App

2021.5 月开始小程序不再支持跳转到 App，那要怎么跳转 App 呢，看了一下腾讯视频的小程序是利用客服消息发送 App 下载地址，感觉此方法体验甚好。源地址：[关于不再提供“小程序打开App技术服务”的通知](https://developers.weixin.qq.com/community/develop/doc/000c04d94c0588744a2cf4d9c5bc09)

```
<button open-type="contact">前方客服会话获取下载链接</button>
```

## H5跳转H5、小程序、App

### 微信h5跳转外部h5

在微信的环境中给出以下头部 
```
header("Content-type:application/pdf");
header("Content-Disposition:attachment;filename='downloaded.pdf'");
```
此时微信会因为头部是下载处理，自动跳转到浏览器中打开这个链接

!! 微信可能会封禁
!! 已停止访问该网页
!! 网页存在安全风险，被多人投诉，为维护绿色上网环境，已停止访问。

### 跳转小程序

#### 微信H5

参考 [微信开放标签](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html#21)

开放对象
* 已认证的服务号，服务号绑定“JS接口安全域名”下的网页可使用此标签跳转任意合法合规的小程序。
* 已认证的非个人主体的小程序，使用小程序云开发的静态网站托管绑定的域名下的网页，可以使用此标签跳转任意合法合规的小程序。

```
<wx-open-launch-weapp
  id="launch-btn"
  appid="wx12345678"
  path="pages/home/index?user=123&action=abc"
>
  <script type="text/wxtag-template">
    <style>.btn { padding: 12px }</style>
    <button class="btn">打开小程序</button>
  </script>
</wx-open-launch-weapp>
<script>
  var btn = document.getElementById('launch-btn');
  btn.addEventListener('launch', function (e) {
    console.log('success');
  });
  btn.addEventListener('error', function (e) {
    console.log('fail', e.detail);
  });
</script>
```
#### H5

##### URL Scheme

可以通过[服务端接口](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/qrcode-link/url-scheme/generateScheme.html)或在[小程序管理后台](https://mp.weixin.qq.com/)「工具」-「生成 URL Scheme」入口可以获取打开小程序任意页面的 URL Scheme。

##### URL Link

通过[服务端接口](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/qrcode-link/url-link/generateUrlLink.html)可以获取打开小程序任意页面的 URL Link

#### 小程序webview

[webview文档](https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html)

```
// <script type="text/javascript" src="https://res.wx.qq.com/open/js/jweixin-1.3.2.js"></script>

// javascript
wx.miniProgram.navigateTo({url: '/path/to/page'})
wx.miniProgram.postMessage({ data: 'foo' })
wx.miniProgram.postMessage({ data: {foo: 'bar'} })
wx.miniProgram.getEnv(function(res) { console.log(res.miniprogram) })

```

### 跳转App

#### 微信H5

参考 [微信开放标签](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html#21)

此功能仅开放给已认证的服务号，服务号绑定“JS接口安全域名”下的网页可使用此标签跳转满足一定条件的App。在使用该标签之前，首先需要前往微信开放平台的管理中心-公众账号或小程序详情-接口信息-网页跳转移动应用-关联设置中绑定所需要跳转的App。详细配置规则参考文档《微信内网页跳转APP功能》。

```
<wx-open-launch-app
  id="launch-btn"
  appid="your-appid"
  extinfo="your-extinfo"
>
  <script type="text/wxtag-template">
    <style>.btn { padding: 12px }</style>
    <button class="btn">App内查看</button>
  </script>
</wx-open-launch-app>
<script>
  var btn = document.getElementById('launch-btn');
  btn.addEventListener('launch', function (e) {
    console.log('success');
  });
  btn.addEventListener('error', function (e) {
    console.log('fail', e.detail);
  });
</script>
```

#### H5

一个**URL Scheme**是由以下部分组成的:
**[scheme]://[host]:[port][path]?[参数]**

说明以及举例

**scheme**是和前端商量好的，一般可以用公司的英文名称，比如淘宝就是**taobao**，这里举例定义为**abc**
**host**是主机，可以写公司域名，可以多级，这里举例定义为**abc.com**
**port**是端口，随便定义，这里举例定义为**8088**
**path**是路径，随便定义，这里举例定义为**/router**
参数是前端带给App的参数，这里举例定义为**token=123&data={这是一个json字符串}**

那么拼出来的URL Scheme就是:
**abc://abc.com:8088/router?token=123&data={这是一个json字符串}**

Universal link方法
Universal link 是苹果公司在2015年推出的新一代Deeplink方法，iOS9及以上的用户可以通过点击一个https链接无缝地跳转到一个App应用内的指定页面，如果用户没有安装这个App，则会跳转到App的下载页面。Universal link方式比URL Scheme方式体验更好,因为中间不需要用户点击确认打开App，也不需要用户在右上角跳转通过safari打开跳转。目前国内微信已经支持Universal link形式的跳转，所以在iOS端采取Universal link的形式唤起App则更为直接方便。

## App跳转H5、小程序、App

### 跳转小程序

[开放平台文档](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Launching_a_Mini_Program/Launching_a_Mini_Program.html)

移动应用(app)拉起小程序功能已向全体开发者开放，开发者在微信开放平台帐号下申请移动应用并通过审核后，即可获得移动应用拉起小程序功能权限。
可在 “管理中心-移动应用-应用详情-关联小程序信息”，为通过审核的移动应用发起关联小程序操作。

在同一开放平台账号下的移动应用及小程序无需关联即可完成跳转，非同一开放平台账号下的小程序需与移动应用（APP）成功关联后才支持跳转。

关联小程序后，将通过APP跳转到小程序。一个移动应用最多只能绑定3个小程序，每月支持绑定3次。同一个小程序最多可被500个移动应用关联。

### 跳转App

通过包名、类名
