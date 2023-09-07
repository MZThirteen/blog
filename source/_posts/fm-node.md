---
title: 猫耳个人数据获取
date: 2023-09-07 17:39:07
tags: 
  - node
  - FM
categories: node
---

功能实现--代码

```
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

// FM
const baseApi = 'https://www.missevan.com/'
const USER_ID = '****'
const TOKEN = '****'
const config = {
  baseJson: true,
  sound: true,
  DM: true,
  CV: true,
  comment: true
}

const request = axios.create({
  timeout: 1000 * 60
})

const fileSavePath = 'FM/'
// 创建存放文件夹
await mkdirSync(fileSavePath)
errorAppendFileSync(`\n*****猫*****${new Date()}*****耳*****\n`)

// 获取收藏列表
async function getSubscriptions({ page, json }) {
  try {
    const res = await request
      .get(
        `${baseApi}dramaapi/getusersubscriptions?page_size=20&page=${page}&user_id=${USER_ID}`,
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
      .get(`${baseApi}dramaapi/getdrama?drama_id=${dramaId}`)
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
        .get(`${baseApi}sound/getdm?soundid=${soundId}`)
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
          `${baseApi}site/getcomment`,
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
              origin: 'https://www.missevan.com'
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
