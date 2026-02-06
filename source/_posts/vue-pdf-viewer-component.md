---
title: Vue3 PDF查看器组件实现
date: 2025-02-06 16:40:00
tags: 
  - Vue3
  - PDF
  - 组件开发
  - TypeScript
categories: 前端
---

## 概述

本文介绍一个基于 Vue3 + TypeScript 的高性能 PDF 查看器组件，支持单页/多页模式、懒加载、旋转、下载、打印等功能，适用于企业级应用。

## 核心特性

- ✅ **单页/多页显示模式**：支持翻页模式和全部展示模式
- ✅ **懒加载优化**：多页模式下使用 IntersectionObserver 实现页面懒加载
- ✅ **旋转功能**：支持 90°、180°、270° 旋转
- ✅ **下载功能**：支持各种格式源（URL、Blob、ArrayBuffer）的下载
- ✅ **打印功能**：集成浏览器打印功能
- ✅ **格式校验**：支持多种方式的 PDF 格式验证
- ✅ **中文支持**：配置 CJK 字体支持
- ✅ **灵活配置**：可自定义工具栏、页码、旋转等功能

## 技术栈

```json
{
  "dependencies": {
    "vue": "^3.x",
    "pdfjs-dist": "latest",
    "vue-pdf-embed": "latest",
    "ant-design-vue": "^4.x"
  }
}
```

## 组件接口设计

### Props

```typescript
interface Props {
  /** PDF 文件源：URL 或 Blob/ArrayBuffer */
  source: PdfSource
  /** 容器最小高度 */
  minHeight?: string
  /** 容器固定高度（可选） */
  height?: string
  /** 初始旋转角度（0, 90, 180, 270），已废弃，请使用 toolbar.rotation */
  rotation?: number
  /** 工具栏配置 */
  toolbar?: ToolbarConfig | boolean
}

type PdfSource = string | URL | Blob | ArrayBuffer | Uint8Array | null
```

### 工具栏配置

```typescript
interface ToolbarConfig {
  pagination?: PaginationConfig | boolean
  rotation?: RotationConfig | boolean
  download?: DownloadConfig | boolean
  print?: PrintConfig | boolean
}

interface PaginationConfig {
  enabled?: boolean
  initialPage?: number
  showAllPages?: boolean
}

interface RotationConfig {
  enabled?: boolean
}

interface DownloadConfig {
  enabled?: boolean
  fileName?: string
}

interface PrintConfig {
  enabled?: boolean
}
```

### 事件

```typescript
const emit = defineEmits<{
  loaded: [totalPages: number]
  error: [error: Error]
}>()
```

### 暴露的方法

```typescript
defineExpose({
  reload,           // 重新加载 PDF
  print,            // 打印
  download,         // 下载
  rotateLeft,       // 左转 90°
  rotateRight,      // 右转 90°
  resetRotation,    // 重置旋转
})
```

## 核心功能实现

### 1. PDF.js 配置

#### Worker 配置

```javascript
import * as pdfjsLib from 'pdfjs-dist'

// PDF.js CDN 基础路径
const PDF_CDN_BASE = `//unpkg.com/pdfjs-dist@${pdfjsLib.version}`

// 配置 PDF.js worker
if (typeof window !== 'undefined') {
  pdfjsLib.GlobalWorkerOptions.workerSrc = `${PDF_CDN_BASE}/build/pdf.worker.min.mjs`
}
```

#### 中文字体支持

```typescript
interface PdfFontConfig {
  url: string
  cMapUrl: string
  cMapPacked: boolean
  standardFontDataUrl: string
  isEvalSupported: boolean
  useSystemFonts: boolean
}

const createPdfFontConfig = (url: string): PdfFontConfig => ({
  url,
  cMapUrl: `${PDF_CDN_BASE}/cmaps/`,
  cMapPacked: true,
  standardFontDataUrl: `${PDF_CDN_BASE}/standard_fonts/`,
  isEvalSupported: false,
  useSystemFonts: false,
})
```

### 2. PDF 格式校验

```typescript
const validatePdfFormat = (source: PdfSource): boolean => {
  if (!source) return false

  // 1. 如果是 Blob，检查 MIME 类型
  if (source instanceof Blob) {
    const validTypes = ['application/pdf', 'application/x-pdf']
    return validTypes.includes(source.type)
  }

  // 2. 如果是 URL 字符串，检查文件扩展名
  if (typeof source === 'string') {
    try {
      const url = new URL(source, window.location.href)
      const pathname = url.pathname.toLowerCase()
      return pathname.endsWith('.pdf')
    } catch {
      return source.toLowerCase().endsWith('.pdf')
    }
  }

  // 3. 如果是 URL 对象，检查路径
  if (source instanceof URL) {
    const pathname = source.pathname.toLowerCase()
    return pathname.endsWith('.pdf')
  }

  // 4. 如果是 ArrayBuffer 或 Uint8Array，检查 PDF 文件头（%PDF-）
  if (source instanceof ArrayBuffer || source instanceof Uint8Array) {
    const bytes = source instanceof ArrayBuffer ? new Uint8Array(source) : source
    // PDF 文件头: %PDF- (0x25, 0x50, 0x44, 0x46, 0x2D)
    if (bytes.length >= 5) {
      return (
        bytes[0] === 0x25 && // %
        bytes[1] === 0x50 && // P
        bytes[2] === 0x44 && // D
        bytes[3] === 0x46 && // F
        bytes[4] === 0x2d    // -
      )
    }
    return false
  }

  return true
}
```

### 3. 懒加载实现

#### 配置参数

```typescript
const LAZY_LOADING_CONFIG = {
  rootMargin: '50px',    // 提前 50px 加载
  threshold: 0.1,        // 10% 可见就触发
  initDelay: 100,        // 初始化延迟（ms）
} as const
```

#### IntersectionObserver 初始化

```typescript
const initLazyLoading = () => {
  if (!showAllPages.value || totalPages.value === 0 || !pdfDocument.value) return

  // 清理旧的观察器
  if (observer.value) {
    observer.value.disconnect()
  }

  // 默认渲染第一页
  if (!renderedPages.value.has(1)) {
    renderedPages.value.add(1)
    nextTick(() => {
      renderPage(1)
    })
  }

  // 创建新的观察器
  observer.value = new IntersectionObserver(
    entries => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const pageElement = entry.target as HTMLElement
          const page = Number(pageElement.dataset.page)

          if (page && !renderedPages.value.has(page)) {
            renderedPages.value.add(page)
            nextTick(() => {
              renderPage(page)
            })
          }
        }
      })
    },
    {
      root: null,
      rootMargin: LAZY_LOADING_CONFIG.rootMargin,
      threshold: LAZY_LOADING_CONFIG.threshold,
    },
  )

  // 等待 DOM 更新后观察所有页面容器
  nextTick(() => {
    const pageWrappers = document.querySelectorAll('.pdf-page-wrapper')
    pageWrappers.forEach(wrapper => {
      const pageElement = wrapper as HTMLElement
      const page = Number(pageElement.dataset.page)
      if (!renderedPages.value.has(page)) {
        observer.value?.observe(wrapper)
      }
    })
  })
}
```

#### 页面渲染

```typescript
const renderPage = async (pageNumber: number) => {
  if (!pdfDocument.value) return

  try {
    const canvas = canvasRefs.value.get(pageNumber)
    if (!canvas) return

    const page = await pdfDocument.value.getPage(pageNumber)
    const viewport = page.getViewport({
      scale: 1.5,
      rotation: currentRotation.value,
    })

    const context = canvas.getContext('2d')
    if (!context) return

    canvas.width = viewport.width
    canvas.height = viewport.height

    await page.render({
      canvasContext: context,
      viewport,
    }).promise

    // 页面渲染完成后释放资源
    page.cleanup()
  } catch (error) {
    console.error(`页面 ${pageNumber} 渲染失败:`, error)
  }
}
```

### 4. 旋转功能

```typescript
// 左转 90 度
const rotateLeft = () => {
  currentRotation.value = (currentRotation.value - 90 + 360) % 360
  if (showAllPages.value) {
    reRenderAllPages()
  }
}

// 右转 90 度
const rotateRight = () => {
  currentRotation.value = (currentRotation.value + 90) % 360
  if (showAllPages.value) {
    reRenderAllPages()
  }
}

// 重新渲染所有已显示的页面
const reRenderAllPages = async () => {
  if (!pdfDocument.value || !showAllPages.value) return

  const pagesToRerender = Array.from(renderedPages.value)
  for (const page of pagesToRerender) {
    await nextTick()
    await renderPage(page)
  }
}
```

### 5. 下载功能

```typescript
const handleDownload = async () => {
  try {
    if (!validatePdfFormat(props.source)) {
      message.error('文件格式不正确，无法下载')
      return
    }

    const fileName = toolbarConfig.value.download.fileName

    // 如果是 Blob，直接下载
    if (props.source instanceof Blob) {
      await downloadByBlob(props.source, {
        fileName,
        mimeType: 'application/pdf',
      })
      return
    }

    // 如果是 ArrayBuffer 或 Uint8Array，转换为 Blob 后下载
    if (props.source instanceof ArrayBuffer || props.source instanceof Uint8Array) {
      const blob = new Blob([props.source], { type: 'application/pdf' })
      await downloadByBlob(blob, {
        fileName,
        mimeType: 'application/pdf',
      })
      return
    }

    // 如果是 URL，使用智能下载
    const url = typeof props.source === 'string' ? props.source : props.source?.toString()
    if (!url) {
      message.error('无法获取文件地址')
      return
    }

    await downloadFile(url, { fileName })
    message.success('下载成功')
  } catch (error) {
    console.error('下载失败:', error)
    message.error('下载失败')
  }
}
```

### 6. 资源清理

```typescript
const cleanupPdfResources = () => {
  // 清理 Blob URL
  if (blobUrl.value) {
    URL.revokeObjectURL(blobUrl.value)
    blobUrl.value = undefined
  }

  // 清理 PDF 文档对象
  if (pdfDocument.value) {
    pdfDocument.value.destroy().catch(() => {
      // 忽略销毁错误
    })
    pdfDocument.value = null
  }

  // 清理观察器
  if (observer.value) {
    observer.value.disconnect()
    observer.value = undefined
  }

  // 清理状态
  renderedPages.value.clear()
  canvasRefs.value.clear()
  isLoadingDocument.value = false
}

// 组件卸载时清理
onUnmounted(() => {
  cleanupPdfResources()
})
```

## 使用示例

### 基础使用

```vue
<template>
  <PdfViewer :source="pdfUrl" />
</template>

<script setup lang="ts">
import PdfViewer from '@/components/PdfViewer/index.vue'

const pdfUrl = 'https://example.com/document.pdf'
</script>
```

### 自定义工具栏

```vue
<template>
  <PdfViewer
    :source="pdfUrl"
    :toolbar="{
      pagination: {
        enabled: true,
        initialPage: 1,
        showAllPages: false
      },
      rotation: { enabled: true },
      download: { 
        enabled: true, 
        fileName: 'my-document.pdf' 
      },
      print: { enabled: true }
    }"
  />
</template>
```

### 多页展示模式

```vue
<template>
  <PdfViewer
    :source="pdfUrl"
    :toolbar="{
      pagination: {
        enabled: false,
        showAllPages: true  // 显示所有页面
      }
    }"
    height="800px"
  />
</template>
```

### 使用 Blob 源

```vue
<template>
  <PdfViewer :source="pdfBlob" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import PdfViewer from '@/components/PdfViewer/index.vue'

const pdfBlob = ref<Blob>()

// 从接口获取 PDF
const loadPdf = async () => {
  const response = await fetch('/api/pdf')
  pdfBlob.value = await response.blob()
}

loadPdf()
</script>
```

### 监听事件

```vue
<template>
  <PdfViewer
    :source="pdfUrl"
    @loaded="handleLoaded"
    @error="handleError"
  />
</template>

<script setup lang="ts">
const handleLoaded = (totalPages: number) => {
  console.log(`PDF 加载成功，共 ${totalPages} 页`)
}

const handleError = (error: Error) => {
  console.error('PDF 加载失败:', error)
}
</script>
```

### 使用暴露的方法

```vue
<template>
  <div>
    <a-button @click="handleRotate">旋转</a-button>
    <a-button @click="handleDownload">下载</a-button>
    <PdfViewer ref="pdfViewerRef" :source="pdfUrl" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import PdfViewer from '@/components/PdfViewer/index.vue'

const pdfViewerRef = ref()

const handleRotate = () => {
  pdfViewerRef.value?.rotateRight()
}

const handleDownload = () => {
  pdfViewerRef.value?.download()
}
</script>
```

## 性能优化

### 1. 懒加载策略

- 使用 IntersectionObserver 监听页面可见性
- 只渲染可视区域附近的页面
- 提前 50px 预加载，提升用户体验

### 2. 内存管理

- 及时调用 `page.cleanup()` 释放页面资源
- 使用 `markRaw()` 避免 PDF 对象被 Vue 响应式包装
- 组件卸载时彻底清理资源

### 3. Blob URL 管理

- 监听 source 变化，及时释放旧的 Blob URL
- 避免内存泄漏

## 注意事项

### 1. CORS 跨域问题

如果 PDF 文件在其他域名下，需要服务器配置 CORS 头：

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET
```

### 2. Worker 路径配置

使用 CDN 或本地路径配置 worker：

```javascript
// 使用 CDN（推荐）
pdfjsLib.GlobalWorkerOptions.workerSrc = 
  `//unpkg.com/pdfjs-dist@${pdfjsLib.version}/build/pdf.worker.min.mjs`

// 使用本地文件
pdfjsLib.GlobalWorkerOptions.workerSrc = '/pdf.worker.min.mjs'
```

### 3. 大文件处理

对于超大 PDF 文件：
- 建议使用单页模式
- 或者使用多页模式 + 懒加载
- 避免一次性渲染所有页面

### 4. 移动端适配

移动端需要注意：
- 调整 scale 比例
- 优化触摸手势
- 减少初始加载页数

## 总结

本 PDF 查看器组件具有以下优势：

1. **功能完善**：支持单页/多页、旋转、下载、打印等功能
2. **性能优化**：懒加载、资源清理、响应式优化
3. **灵活配置**：工具栏、页码、旋转等可自定义
4. **类型安全**：完整的 TypeScript 类型定义
5. **易于使用**：简洁的 API，丰富的使用示例

适用于企业级 Web 应用中的文档预览场景。
