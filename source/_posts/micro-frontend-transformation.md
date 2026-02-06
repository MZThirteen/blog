---
title: 微前端架构改造实践
date: 2025-02-06 16:35:00
tags: 
  - 微前端
  - Micro-app
  - Vue
  - 前端架构
categories: 前端
---

## 概述

本文档基于 Micro-app 框架，详细说明微前端架构中主应用与内嵌子应用的改造要点、配置步骤及注意事项，确保两者实现无缝集成、隔离兼容。

## 主应用改造内容

主应用核心职责：提供微应用容器、管理路由分发、处理跨应用通信、保障样式与沙箱隔离。

### 基础环境配置

#### 安装依赖

主应用需安装 Micro-app 核心包：

```bash
pnpm add @micro-zoe/micro-app
```

#### 初始化 Micro-app

在主应用入口文件中初始化：

```javascript
import microApp from '@micro-zoe/micro-app'

microApp.start()
```

### 路由适配改造

#### 配置子应用路由规则

主应用需为每个子应用配置独立路由：

```typescript
const routes: RouteRecordRaw[] = [
  {
    path: '/sub-app-a/:page*',
    name: 'SubAppA',
    component: () => import('@/views/sub-app/detail-a.vue'),
    meta: {
      title: '子应用A详情',
      platform: 'web',
      requiresAuth: false, // 需要登录
    },
  },
  {
    path: '/sub-app-b/:page*',
    name: 'SubAppB',
    component: () => import('@/views/sub-app/detail-b.vue'),
    meta: {
      title: '子应用B详情',
      platform: 'web',
      requiresAuth: false, // 需要登录
    },
  },
]
```

#### 子应用容器组件实现

每个子应用对应一个容器组件，通过 `<micro-app>` 标签嵌入：

```vue
<template>
  <div class="micro-app-container">
    <!-- 子应用容器 -->
    <micro-app
      name="sub-app"
      :url="URL_CONFIG.subApp.baseUrl"
      iframe
      baseroute="/sub-app-b"
    ></micro-app>
  </div>
</template>

<script setup lang="ts">
import { URL_CONFIG } from '@/constants/url-config'
import { redirectToLogin } from '@/utils/auth'

defineOptions({ name: 'SubAppDetail' })

// 监听子应用消息
window.addEventListener('message', event => {
  if (event.data.type === 'MICRO_APP_TO_LOGIN') {
    redirectToLogin()
  }
})
</script>

<style scoped lang="scss">
.micro-app-container {
  width: 100%;
  min-height: 400px;
  position: relative;
}
</style>
```

### 其他改造要点

#### 回调地址编码处理

```javascript
// 改造前
url.searchParams.set('redirectUrl', redirectUrl)

// 改造后
url.searchParams.set('redirectUrl', encodeURI(redirectUrl))
```

#### 登录回调后参数丢失修复

```javascript
// 改造前
next({ path: cleanPath, replace: true })

// 修复：直接使用 window.history.replaceState 保留完整路径
window.history.replaceState(null, '', cleanPath)
next()
```

#### 子应用路由菜单对应显示

```javascript
// 根据路由名称判断激活的菜单项
if (['SubApp', 'SubAppList', 'SubAppA', 'SubAppB'].includes(String(name))) {
    return '/sub-app'
}
```

#### 跳转子应用方法修改

```javascript
// 改造前
// 直接跳转到子应用URL
redirectSelf(URL_CONFIG.subApp.introduce, {
    productId: props.item.productId,
    ticket,
})

// 改造后
// 构建子应用路径
const subAppPath = `/web/detail.html#/apply/introduce?productId=${props.item.productId}`
router.push({
    path: `/sub-app-b`,
    query: {
      'sub-app': subAppPath,
    },
})
```

#### 标签警告修复

在 Vite 配置中添加自定义元素配置：

```javascript
vue({
  template: {
    compilerOptions: {
      isCustomElement: tag => tag === 'micro-app',
    },
  },
}),
```

## 子应用改造内容

子应用核心职责：适配微前端环境、修改路由基准路径（如果基座应用是 history 路由，子应用是 hash 路由，则不需要设置 baseroute）、处理样式隔离、实现跨应用通信。

### 样式隔离改造

避免子应用 `:root` 变量污染主应用，需修改样式作用域。替换 `:root` 为子应用根容器选择器：

```css
/* 子应用全局样式（改造前） */
:root {
  --spacing: 0.2667vw;
}

/* 改造后 */
#app { /* 子应用根容器 ID */
  --spacing: 0.2667vw; /* 仅作用于子应用内部 */
}
```

### 适配微前端环境

#### 登录处理（由主应用处理）

```javascript
/* 改造前 */
export const navigateToSSO = () => {
  if (isWechatIOS()) {
    window.location.replace(ssoUrl())
  } else {
    window.location.href = ssoUrl()
  }
}

/* 改造后 */
export const navigateToSSO = () => {
  // 在微前端环境中，使用主应用的导航
  if (window.__MICRO_APP_ENVIRONMENT__) {
    // 通知主应用进行导航
    window.parent?.postMessage(
      {
        type: 'MICRO_APP_TO_LOGIN',
      },
      '*',
    )
  } else if (isWechatIOS()) {
    window.location.replace(ssoUrl())
  } else {
    window.location.href = ssoUrl()
  }
}
```

#### 跳转处理（window 代理）

```javascript
// 统一的页面跳转方法
export const navigateToUrl = (url: string, method: 'replace' | 'href' = 'href') => {
  // 微前端环境：使用原生 window 跳转（绕过沙箱代理）
  if (window.__MICRO_APP_ENVIRONMENT__) {
    const rawWindow = (window as any).rawWindow || window.parent || window.top
    if (method === 'replace') {
      rawWindow?.location.replace(url)
    } else {
      rawWindow.location.href = url
    }
    return
  }

  // 独立环境：直接使用指定的跳转方式
  if (method === 'replace') {
    window.location.replace(url)
  } else {
    window.location.href = url
  }
}
```

### TypeScript 类型声明

```typescript
/**
 * 微前端相关类型声明
 */
interface Window {
    /**
     * 微前端环境标识
     */
    __MICRO_APP_ENVIRONMENT__?: boolean
}
```

## 待优化项

- [ ] 代理与跨域配置优化
- [ ] 子应用预加载策略
- [ ] 性能监控与优化
- [ ] 错误边界处理

## 总结

通过 Micro-app 框架实现微前端架构改造，主要关注以下几点：

1. **主应用改造**：路由配置、容器组件、通信机制
2. **子应用改造**：样式隔离、环境适配、跨应用通信
3. **注意事项**：参数编码、路径保留、沙箱代理绕过

合理的微前端架构可以实现应用间的解耦，提高开发效率和可维护性。
