---
title: GeoJson-数据获取
date: 2023-09-07 17:15:22
tags: GeoJson
categories: 前端
---

### 省市区获取

[阿里云地址](https://datav.aliyun.com/portal/school/atlas/area_selector)

所需数据过多时，可考虑脚本爬取。

### geojson.io

使用geojson.io添加对应的属性
[https://geojson.io/](https://geojson.io/)

### 天地图

行政区划服务获取
[api地址](http://lbs.tianditu.gov.cn/server/administrative.html)

### 村镇级别获取

#### 准备工作

* 需要转换村镇的png/svg图
* Vector Magin用于将png转换为svg工具
* Svg2geojson工具，git地址：[Svg2geojson](https://github.com/yuhonyon/svg2geojson)
* geojson添加属性工具：[geojson.io](http://geojson.io/)（需要T子）
* geojson压缩工具：[mapshaper](https://mapshaper.org/)（需要T子）

#### 整体思路

1. PNG转SVG
2. SVG转换为GeoJson

[可参考-待测试](https://juejin.cn/post/7257333598468997178?from=search-suggest)