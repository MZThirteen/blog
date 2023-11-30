---
title: Flutter 常用命令行
date: 2023-11-30 11:45:05
tags: flutter
categories: 前端
---

1. flutter doctor 查看flutter的状态，查看环境配置是否有问题
2. flutter doctor -v 查看flutter 状态的详细信息
3. flutter build apk 打包安卓包
4. flutter build ios 打包苹果ipa
5. flutter run 运行项目 默认--debug
6. flutter run --profile 运行线上测试包
7. flutter run --release 运行线上包
8. flutter channel 查看flutter 的所有分支
9. flutter channel stable 切换到具体的分支
10. flutter upgrade 升级flutter
11. flutter upgrade --force 如果升级flutter出现问题可以尝试 强制更新
12. flutter logs 当链接到某一个设备的时候，通过此命令可以查看到当前设备的log
13. flutter screenshot 可以截取项目当前屏幕展示的图到项目里
14. futter clean 清除缓存
15. flutter analyze Dart默认的linter配置有点弱, 有很多有问题代码也不报错或警告. 通过此命令可以应用dart的最佳代码实践, 对一些不好的代码风格提出警告或者直接报错, 从而提高代码质量
16. flutter attach 混合开发常用命令，1，首先起项目，运行起整个工程；2，到命令行，打开 flutter_lib 目录（Flutter module工程）；3，输入命令：flutter attach
17. flutter test 当前项目的单元测试
18. flutter downgrade 从flutter当前channel 下降到上一个稳定版本
19. flutter install 直接下载apk到手机上，很方便使用，不用重复build or run
20. flutter create 创建一个flutter 新项目 例如：
js复制代码flutter create --org com.yourdomain your_app_name
21. flutter -h 如果忘记具体的命令行 可以通过此命令查找
22. flutter pub get 安装依赖包
23. flutter pub add '' 安装指定依赖