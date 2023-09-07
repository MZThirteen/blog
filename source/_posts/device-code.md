---
title: 设备唯一标识
date: 2023-09-05 10:06:15
tags: flutter
categories: 前端
---

## H5

在H5开发中，获取手机设备的唯一标识是很难的，因为Web浏览器不允许访问底层硬件信息。但是，可以使用一些前端技巧来生成一个伪唯一标识。

一种方法是使用网页存储（localStorage或sessionStorage）来存储一个生成的UUID，如果该UUID不存在，则生成一个新的并存储在网页存储中，否则就直接使用存储的UUID。这种方法的缺点是如果用户清除了浏览器数据，UUID也将被清除。

另一种方法是使用网页指纹（fingerprint）技术，该技术可以通过收集浏览器、设备和操作系统等信息生成一个经过哈希处理的唯一识别码。但是，网页指纹也不是100%唯一且易受恶意攻击，因此不能完全依赖网页指纹。

[指纹测试地址](https://fingerprintjs.github.io/fingerprintjs/)

## Android

### 常见的几个解决方案的弊端

* IMEI
  Android 10 中官方明确说明**第三方应用无法获取到IMEI码**：[地址](https://developer.android.google.cn/about/versions/10/privacy/changes?hl=zh-cn)
  Android 10 以下的版本，需要申请READ_PHONE_STATE权限。
* Android ID
  **Android ID 不具有真正的唯一性，**
  ROOT、刷机、恢复出厂设置、不同签名的应用等都会导致获取的 Android ID 发生改变，
  并且不同厂商定制的系统的BUG会导致不同的设备可能会产生相同的 Android ID。
* MAC地址
  **Android 10 中 MAC地址具有随机化的特征：**[地址](https://developer.android.google.cn/about/versions/10/privacy/changes?hl=zh-cn#randomized-mac-addresses)
  虽然目前大部分手机还不支持这个特性，但是随着厂商的跟进，这个方案就会逐渐作废
* OAID
  APP类广告效果追踪需要使用到用户的设备标识进行广告点击和转化效果的匹配，而安卓系统当前强依赖于IMEI的获取，上面提到Andorid Q（10.0）版本后，将无法获取IMEI。基于此背景，进行广告投放的效果追踪，需要能够替代及补充IMEI的设备标识。
  用户可以手动在设置中通过重置广告标识符更换OAID或者“限制广告跟踪”。

### uuid + 本地文件，实现一个通用解决方案

启动APP时，检查并读取根目录下保存有uuid的文件，若没有该文件，则视为一台新设备，创建文件并写入uuid。

并且要确保卸载应用时，该文件不会被系统携带着删除（这也是为什么要在根目录下创建的原因）。

## IOS

* IDFA

  IDFA 是苹果 iOS 6 开始新增的广告标识符，用于给开发者跟踪广告效果用的，可以简单理解为 iPhone 的设备临时身份证，说是临时身份证是因为它允许用户更换，IDFA 存储在用户 iOS 系统上，同一设备上的应用获取到的 IDFA 是相同的。iOS 用户可以通过（设置程序 -> 通用 -> 还原 -> 还原位置与隐私）更换 IDFA，iOS 10 系统开始提供禁止广告跟踪功能，用户勾选这个功能后，应用程序将无法读取到设备的 IDFA。（在统计唯一用户的时候，IDFA 的可变性会造成部分用户的重复统计。）

  适用于对外：例如广告推广，换量等跨应用的用户追踪等。

  总结：iOS 6 时面世，可以监控广告效果，同时保证用户设备不被APP追踪的折中方案。可能发生变化，如系统重置、在设置里还原广告标识符。用户可以在设置里打开“限制广告跟踪”。

* UDID

  用来标示设备的唯一性，由40个字符的字母和数字组成 。

  iOS 6 之后被禁止获取系统原生的UDID，但可以通过uuid，写入到钥匙串中，从而获得自定义的UDID（非系统原生），即使用户重装APP，只要每次都取这个钥匙串返回，就是不变的。

  OpenUDID：是一个替代 UDID 的第三发解决方案。缺点是如果你完全删除全部带有 OpenUDID SDK 包的 App（比如恢复系统等），那么 OpenUDID 会重新生成，而且和之前的值会不同，相当于新设备。

### IDFV + Keychain

```
import 'package:device_info/device_info.dart';
import 'package:flutter_keychain/flutter_keychain.dart';

var iDFVKey = 'IDFV';
String? oldIDFV = await FlutterKeychain.get(key: iDFVKey);
print('oldIDFV---$oldIDFV');
if (!isEmpty(oldIDFV)) {
  return oldIDFV!;
}
IosDeviceInfo iosInfo = await deviceInfo.iosInfo;
String newIDFV = iosInfo.identifierForVendor;
await FlutterKeychain.put(key: iDFVKey, value: newIDFV);
print('newIDFV---$newIDFV');
return newIDFV;
```

[参考1](https://zhuanlan.zhihu.com/p/359752003)
[参考2](https://zhuanlan.zhihu.com/p/395387972)
[参考3](https://www.zhihu.com/question/397322621/answer/1247919471)
[参考4](https://juejin.cn/post/7049234772937146399)