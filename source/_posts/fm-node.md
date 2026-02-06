---
title: Node.js 实现音频网站数据批量下载
date: 2023-09-07 17:39:07
tags: 
  - node
  - 爬虫
  - 数据备份
categories: node
---

## 概述

本文介绍如何使用 Node.js 实现音频网站的个人数据批量下载，包括音频文件、弹幕、评论等内容的备份。

## 功能实现

### 核心代码

```javascript
import axios from 'axios'
import * as fs from 'fs'
import qs from 'qs'
import {
  downloadFile,
  requestAfterDownloadFile,
  mkdirSync,
  appendFileSync,
  errorAppendFileSync
} from './utils/fs.js'

// 配置项
const baseApi = 'https://example.com/'
const USER_ID = 'YOUR_USER_ID'  // 替换为你的用户ID
const TOKEN = 'YOUR_TOKEN'      // 替换为你的认证Token
const config = {
  baseJson: true,   // 是否保存基础JSON信息
  sound: true,      // 是否下载音频文件
  DM: true,         // 是否下载弹幕
  CV: true,         // 是否下载CV信息
  comment: true     // 是否下载评论
}

const request = axios.create({
  timeout: 1000 * 60
})

const fileSavePath = 'AudioData/'
// 创建存放文件夹
await mkdirSync(fileSavePath)
errorAppendFileSync(`\n***** 开始下载 ${new Date()} *****\n`)

// 获取收藏列表
async function getSubscriptions({ page, json }) {
  try {
    const res = await request
      .get(
        `${baseApi}api/getusersubscriptions?page_size=20&page=${page}&user_id=${USER_ID}`,
        {
          headers: {
            Cookie: `token=${TOKEN}`
          }
        }
      )
      .catch(() => {
        throw new Error(`收藏第 ${page} 页 ==> 请求接口报错`)
      })
    const { Datas: lists, pagination } = res?.data?.info
    const { maxpage: maxPage } = pagination
    console.log(`***************  收藏第 ${page} 页  ***************`)
    // 创建基础信息json
    if (config.baseJson && page === maxPage) {
      const fileNameJson = `${fileSavePath}index.json`
      const _lists = [...json, ...lists].map((val) => {
        return {
          id: val.id,
          name: val.name,
          type: val.type,
          abstract: val.abstract,
          cover: val.cover
        }
      })
      await appendFileSync(fileNameJson, JSON.stringify(_lists))
    }
    for (const list of lists) {
      await getList({ dramaId: list.id, dramaName: list.name })
    }
    if (page < maxPage) {
      await getSubscriptions({ page: page + 1, json: [...json, ...lists] })
    }
  } catch (error) {
    console.log(`收藏第 ${page} 页 ==> 报错`)
    errorAppendFileSync(`${error?.message || JSON.stringify(error)}\n`)
  }
}

// 获取集数列表
async function getList({ dramaId, dramaName }) {
  try {
    const res = await request
      .get(`${baseApi}api/getdrama?drama_id=${dramaId}`)
      .catch(() => {
        throw new Error(`${dramaName}获取集数列表 ==> 请求接口报错`)
      })
    const { cvs, drama, episodes } = res?.data?.info
    console.log(`------  ${dramaName}获取集数  ------`)
    const path = `${fileSavePath}${dramaName}-${dramaId}`
    // 创建集数文件夹
    await mkdirSync(path)
    // 创建集数弹幕文件夹
    if (config.DM) {
      await mkdirSync(`${path}/DM`)
    }
    // 创建集数热评文件夹
    if (config.comment) {
      await mkdirSync(`${path}/commentHot`)
    }
    // CV
    if (config.CV) {
      await mkdirSync(`${path}/CV`)
    }
    // 创建集数信息json
    if (config.baseJson) {
      const fileNameJson = `${path}/index.json`
      const json = {
        cvs,
        drama,
        episodes
      }
      await appendFileSync(fileNameJson, JSON.stringify(json))
      // 封面图
      await requestAfterDownloadFile(drama.cover, `${path}/cover.jpg`)
    }
    // CV
    if (config.CV) {
      for (const cv of cvs) {
        const { icon, id } = cv?.cv_info
        await requestAfterDownloadFile(icon, `${path}/CV/${id}.jpg`)
      }
    }
    const lists = [...episodes.episode, ...episodes.ft, ...episodes.music]
    for (const list of lists) {
      if (config.sound) {
        await getSound({
          soundId: list.sound_id,
          fileName: `${path}/${list.name}-${list.sound_id}.m4a`
        })
      }
      if (config.DM) {
        await getSoundDM({
          soundId: list.sound_id,
          fileName: `${path}/DM/${list.sound_id}.xml`
        })
      }
      if (config.comment) {
        await getCommentHot({
          page: 1,
          soundId: list.sound_id,
          fileName: `${path}/commentHot/${list.sound_id}.json`,
          json: []
        })
      }
    }
  } catch (error) {
    console.log(`${dramaName}获取集数 ===> 报错`)
    errorAppendFileSync(`${error?.message || JSON.stringify(error)}\n`)
  }
}

// 获取详情
async function getSound({ soundId, fileName }) {
  try {
    const checkDir = fs.existsSync(fileName)
    if (checkDir) {
      console.log(`${fileName} ==> 已下载`)
    } else {
      const res = await request
        .get(`${baseApi}sound/getsound?soundid=${soundId}`, {
          headers: {
            Cookie: `token=${TOKEN}`
          }
        })
        .catch(() => {
          throw new Error(`${fileName} ==> 请求接口报错`)
        })
      const { sound } = res?.data?.info
      if (sound?.soundurl) {
        console.log(`${fileName} ==> 下载中...`)
        await downloadFile(sound.soundurl, fileName)
      } else {
        throw new Error(`${fileName} ==> 未获取到资源`)
      }
    }
  } catch (error) {
    console.log(`${fileName} ===> 报错`)
    errorAppendFileSync(`${error?.message || JSON.stringify(error)}\n`)
  }
}

// 获取弹幕
async function getSoundDM({ soundId, fileName }) {
  try {
    const checkDirJson = fs.existsSync(fileName)
    if (checkDirJson) {
      console.log(`${fileName}---弹幕 ==> 已下载`)
    } else {
      console.log(`${fileName}---弹幕 ==> 下载中...`)
      const res = await request
        .get(`${baseApi}api/getdm?soundid=${soundId}`)
        .catch(() => {
          throw new Error(`${fileName}---弹幕 ==> 请求接口报错`)
        })
      const json = res?.data
      fs.appendFileSync(fileName, json, 'UTF-8', { flags: 'a' })
      console.log(`${fileName}---弹幕 ==> 下载成功`)
    }
  } catch (error) {
    console.log(`${fileName}---弹幕 ===> 报错`)
    errorAppendFileSync(`${error?.message || JSON.stringify(error)}\n`)
  }
}

// 获取评论
async function getCommentHot({ page, soundId, fileName, json }) {
  try {
    const checkDirJson = fs.existsSync(fileName)
    if (checkDirJson) {
      console.log(`${fileName}---评论 ==> 已下载`)
    } else {
      const res = await request
        .post(
          `${baseApi}api/getcomment`,
          qs.stringify({
            type: 1,
            eId: soundId,
            order: 1,
            p: page,
            pagesize: 100
          }), // order 1-最热 3-最新
          {
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded',
              origin: baseApi
            }
          }
        )
        .catch(() => {
          throw new Error(`${fileName} 评论第 ${page} 页 ==> 请求接口报错`)
        })
      const { Datas: lists, pagination } = res?.data?.info?.comment
      const { maxpage: maxPage } = pagination
      console.log(
        `***************  ${fileName} 评论第 ${page} 页  ***************`
      )
      if (page === maxPage || page == 30) {
        await appendFileSync(fileName, JSON.stringify(json))
      }
      if (page < maxPage) {
        await getCommentHot({
          page: page + 1,
          soundId,
          fileName,
          json: [...json, ...lists]
        })
      }
    }
  } catch (error) {
    console.log(`${fileName}---评论 ===> 报错`)
    if (fs.existsSync(fileName)) {
      fs.rmdirSync(fileName)
    }
    errorAppendFileSync(`${error?.message || JSON.stringify(error)}\n`)
  }
}
```

## 主要功能模块

### 1. 收藏列表获取

- 支持分页获取用户收藏列表
- 自动保存基础信息 JSON
- 递归处理所有页面

### 2. 音频文件下载

- 批量下载音频文件（M4A 格式）
- 自动检测已下载文件，避免重复下载
- 错误处理和日志记录

### 3. 弹幕下载

- 下载音频对应的弹幕数据（XML 格式）
- 保存到独立的 DM 文件夹

### 4. 评论下载

- 获取热门评论
- 支持分页，最多获取 30 页
- 保存为 JSON 格式

### 5. CV 信息下载

- 下载配音演员头像
- 保存 CV 相关信息

## 配置说明

```javascript
const config = {
  baseJson: true,   // 是否保存基础JSON信息
  sound: true,      // 是否下载音频文件
  DM: true,         // 是否下载弹幕
  CV: true,         // 是否下载CV信息
  comment: true     // 是否下载评论
}
```

## 文件结构

```
AudioData/
├── index.json                    // 收藏列表汇总
├── 剧集名称-ID/
│   ├── index.json               // 剧集详情
│   ├── cover.jpg                // 封面图
│   ├── 第1集-ID.m4a             // 音频文件
│   ├── 第2集-ID.m4a
│   ├── DM/                      // 弹幕文件夹
│   │   ├── ID.xml
│   │   └── ...
│   ├── commentHot/              // 热门评论文件夹
│   │   ├── ID.json
│   │   └── ...
│   └── CV/                      // CV头像文件夹
│       ├── CV_ID.jpg
│       └── ...
└── error.log                     // 错误日志
```

## 使用方法

1. 安装依赖：
```bash
npm install axios qs
```

2. 配置用户信息：
```javascript
const USER_ID = 'YOUR_USER_ID'
const TOKEN = 'YOUR_TOKEN'
```

3. 运行脚本：
```bash
node index.js
```

## 注意事项

1. **认证信息**：需要替换 `USER_ID` 和 `TOKEN` 为实际的认证信息
2. **接口频率**：建议添加请求延迟，避免频繁请求被封禁
3. **存储空间**：音频文件较大，注意硬盘空间
4. **错误处理**：所有错误都会记录到 `error.log` 文件
5. **断点续传**：已下载的文件会自动跳过

## 扩展功能

可以根据需求添加以下功能：

- 添加请求延迟（防止频繁请求）
- 支持断点续传
- 多线程下载
- 进度条显示
- 自动重试机制

## 总结

这个脚本实现了音频网站个人数据的批量备份功能，通过模块化设计，可以灵活配置需要下载的内容类型，适合用于个人数据备份和归档。
