---
title: Vue3 骨架屏组件库设计与实现
date: 2025-02-06 16:45:00
tags: 
  - Vue3
  - 骨架屏
  - 组件库
  - TypeScript
  - UI组件
categories: 前端
---

## 概述

骨架屏（Skeleton Screen）是一种优化用户体验的 UI 技术，在内容加载前展示页面的大致结构，避免白屏等待，提升感知性能。本文介绍一套基于 Vue3 + TypeScript 的企业级骨架屏组件库的设计与实现。

## 组件库特性

- ✅ **组件丰富**：涵盖表格、表单、列表、详情页等常见场景
- ✅ **灵活配置**：支持自定义行数、列数、布局方式等
- ✅ **嵌入模式**：支持组件嵌套，构建复杂页面骨架
- ✅ **动画效果**：渐变流动动画，提升视觉体验
- ✅ **TypeScript**：完整的类型定义，类型安全
- ✅ **易于扩展**：模块化设计，易于定制和扩展

## 组件列表

| 组件名称 | 用途 | 主要配置 |
|---------|------|---------|
| `TableSkeleton` | 表格骨架屏 | 行数、列数、表头、分页 |
| `FormSkeleton` | 表单骨架屏 | 字段数、布局方式、按钮 |
| `ListSkeleton` | 列表骨架屏 | 项数、头像、网格布局 |
| `DetailSkeleton` | 详情页骨架屏 | 头部、卡片配置数组 |
| `CardContentSkeleton` | 卡片内容骨架屏 | 字段数、布局、列数 |
| `FilePreviewSkeleton` | 文件预览骨架屏 | 标题、按钮、边框 |

## 架构设计

### 目录结构

```
skeleton/
├── index.ts                      # 统一导出入口
├── types.ts                      # TypeScript 类型定义
├── skeleton-common.scss          # 公共样式和 mixin
├── table-skeleton.vue            # 表格骨架屏
├── form-skeleton.vue             # 表单骨架屏
├── list-skeleton.vue             # 列表骨架屏
├── detail-skeleton.vue           # 详情页骨架屏
├── card-content-skeleton.vue     # 卡片内容骨架屏
└── file-preview-skeleton.vue     # 文件预览骨架屏
```

### 类型系统设计

#### 基础类型

```typescript
// 骨架屏基础 Props
interface BaseSkeletonProps {
  /** 是否激活动画 */
  active?: boolean
}

// 嵌入式骨架屏 Props（可嵌入到其他组件中）
interface EmbeddableSkeletonProps extends BaseSkeletonProps {
  /** 是否为嵌入模式（去掉 padding 和 background） */
  embedded?: boolean
}

// 布局类型
type SkeletonLayout = 'vertical' | 'horizontal'

// 头像形状类型
type AvatarShape = 'circle' | 'square'
```

#### 组件 Props 类型

```typescript
// 表格骨架屏
interface TableSkeletonProps extends BaseSkeletonProps {
  rows?: number              // 显示的行数
  showHeader?: boolean       // 是否显示表头
  columns?: number           // 列数
  showPagination?: boolean   // 是否显示分页
}

// 表单骨架屏
interface FormSkeletonProps extends EmbeddableSkeletonProps {
  fields?: number            // 表单项数量
  showButton?: boolean       // 是否显示提交按钮
  layout?: SkeletonLayout    // 布局方式
}

// 列表骨架屏
interface ListSkeletonProps extends EmbeddableSkeletonProps {
  count?: number             // 列表项数量
  showAvatar?: boolean       // 是否显示头像
  avatarShape?: AvatarShape  // 头像形状
  columns?: number           // 列数（网格布局）
  rows?: number              // 段落行数
}
```

#### 详情页卡片配置

```typescript
// 卡片配置 - 表单类型
interface CardConfigForm {
  type: 'form'
  showTitle?: boolean
  fields?: number
  layout?: SkeletonLayout
  showButton?: boolean
}

// 卡片配置 - 表格类型
interface CardConfigTable {
  type: 'table'
  showTitle?: boolean
  rows?: number
  columns?: number
  showHeader?: boolean
  showPagination?: boolean
}

// 卡片配置 - 列表类型
interface CardConfigList {
  type: 'list'
  showTitle?: boolean
  count?: number
  showAvatar?: boolean
  avatarShape?: AvatarShape
  columns?: number
  rows?: number
}

// 卡片配置联合类型
type CardConfig =
  | CardConfigForm
  | CardConfigTable
  | CardConfigList
  | CardConfigFilePreview
  | CardConfigFields
```

### 公共样式设计

#### SCSS Mixin

```scss
// 骨架屏渐变背景 mixin
@mixin gradient {
  background: linear-gradient(90deg, #f0f0f0 25%, #e8e8e8 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  border-radius: 2px;
}

// 嵌入模式 mixin
@mixin embedded {
  padding: 0;
  background: transparent;
  border-radius: 0;
}
```

#### 动画效果

```scss
// 骨架屏动画效果
.skeleton-animated {
  animation: skeleton-loading 1.5s ease-in-out infinite;
}

@keyframes skeleton-loading {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}
```

## 核心组件实现

### 1. TableSkeleton - 表格骨架屏

#### 功能特性

- ✅ 支持自定义行数和列数
- ✅ 可选择是否显示表头
- ✅ 可选择是否显示分页器
- ✅ 自适应列宽

#### 实现代码

```vue
<template>
  <div class="table-skeleton">
    <!-- 表头骨架屏 -->
    <div v-if="showHeader" class="table-skeleton-header">
      <div
        v-for="col in columns"
        :key="col"
        class="table-skeleton-header-cell"
        :class="{ 'skeleton-animated': active }"
      ></div>
    </div>

    <!-- 表格行骨架屏 -->
    <div class="table-skeleton-body">
      <div v-for="row in rows" :key="row" class="table-skeleton-row">
        <div
          v-for="col in columns"
          :key="col"
          class="table-skeleton-cell"
          :class="{ 'skeleton-animated': active }"
        ></div>
      </div>
    </div>

    <!-- 分页骨架屏 -->
    <div v-if="showPagination" class="table-skeleton-pagination">
      <!-- 分页组件... -->
    </div>
  </div>
</template>

<script setup lang="ts">
import type { TableSkeletonProps } from './types'

defineOptions({ name: 'TableSkeleton' })

withDefaults(defineProps<TableSkeletonProps>(), {
  rows: 10,
  showHeader: true,
  columns: 5,
  showPagination: false,
  active: true,
})
</script>
```

#### 样式实现

```scss
.table-skeleton {
  width: 100%;
  background: #fff;
  border-radius: 4px;

  &-header {
    display: flex;
    gap: 16px;
    padding: 16px;
    border-bottom: 1px solid #f0f0f0;

    &-cell {
      flex: 1;
      height: 16px;
      @include skeleton.gradient;

      &:first-child {
        max-width: 60px;
      }
    }
  }

  &-row {
    display: flex;
    gap: 16px;
    padding: 12px 16px;
    border-bottom: 1px solid #f0f0f0;
  }

  &-cell {
    flex: 1;
    height: 14px;
    @include skeleton.gradient;
  }
}
```

### 2. FormSkeleton - 表单骨架屏

#### 功能特性

- ✅ 支持水平/垂直布局
- ✅ 可自定义表单项数量
- ✅ 可选择是否显示提交按钮
- ✅ 支持嵌入模式

#### 实现代码

```vue
<template>
  <div class="form-skeleton" :class="{ 'is-embedded': embedded }">
    <div class="form-skeleton-content" :class="[`form-skeleton-${layout}`]">
      <!-- 表单项骨架屏 -->
      <div v-for="field in fields" :key="field" class="form-skeleton-field">
        <div class="form-skeleton-label" :class="{ 'skeleton-animated': active }"></div>
        <div class="form-skeleton-input" :class="{ 'skeleton-animated': active }"></div>
      </div>
    </div>

    <!-- 提交按钮骨架屏 -->
    <div v-if="showButton" class="form-skeleton-buttons">
      <div class="form-skeleton-button-primary" :class="{ 'skeleton-animated': active }"></div>
      <div class="form-skeleton-button-default" :class="{ 'skeleton-animated': active }"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { FormSkeletonProps } from './types'

withDefaults(defineProps<FormSkeletonProps>(), {
  fields: 6,
  showButton: false,
  layout: 'horizontal',
  active: true,
  embedded: false,
})
</script>
```

#### 布局样式

```scss
// 水平布局
.form-skeleton-horizontal {
  .form-skeleton-field {
    display: flex;
    gap: 16px;
    align-items: center;
  }

  .form-skeleton-label {
    flex-shrink: 0;
    width: 100px;
    height: 14px;
  }

  .form-skeleton-input {
    flex: 1;
    height: 32px;
  }
}

// 垂直布局
.form-skeleton-vertical {
  .form-skeleton-field {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .form-skeleton-label {
    width: 80px;
    height: 14px;
  }

  .form-skeleton-input {
    width: 100%;
    height: 32px;
  }
}
```

### 3. ListSkeleton - 列表骨架屏

#### 功能特性

- ✅ 支持单列/网格布局
- ✅ 可选择是否显示头像
- ✅ 支持圆形/方形头像
- ✅ 可自定义列表项数量和段落行数

#### 实现代码

```vue
<template>
  <div
    class="list-skeleton"
    :class="{ 'list-skeleton-grid': columns > 1, 'is-embedded': embedded }"
    :style="columns > 1 ? { gridTemplateColumns: `repeat(${columns}, 1fr)` } : undefined"
  >
    <div v-for="item in count" :key="item" class="list-skeleton-item">
      <!-- 头像 -->
      <div
        v-if="showAvatar"
        class="list-skeleton-avatar"
        :class="[
          { 'skeleton-animated': active },
          avatarShape === 'circle' ? 'list-skeleton-avatar-circle' : 'list-skeleton-avatar-square',
        ]"
      ></div>

      <!-- 内容区域 -->
      <div class="list-skeleton-content">
        <div class="list-skeleton-title" :class="{ 'skeleton-animated': active }"></div>
        <div class="list-skeleton-rows">
          <div
            v-for="row in rows"
            :key="row"
            class="list-skeleton-row"
            :class="{ 'skeleton-animated': active }"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>
```

### 4. DetailSkeleton - 详情页骨架屏

#### 功能特性

- ✅ 支持配置多个卡片
- ✅ 每个卡片可以是不同类型（表单、表格、列表等）
- ✅ 支持显示页面头部
- ✅ 灵活的组合式设计

#### 实现代码

```vue
<template>
  <div class="detail-skeleton">
    <!-- 头部骨架屏 -->
    <div v-if="showHeader" class="detail-skeleton-header">
      <div class="detail-skeleton-title" :class="{ 'skeleton-animated': active }"></div>
      <div class="detail-skeleton-subtitle" :class="{ 'skeleton-animated': active }"></div>
    </div>

    <!-- 卡片骨架屏 -->
    <div class="detail-skeleton-cards">
      <div v-for="(card, index) in cards" :key="index" class="detail-skeleton-card">
        <!-- 卡片标题 -->
        <div
          v-if="card.showTitle !== false"
          class="detail-skeleton-card-title"
          :class="{ 'skeleton-animated': active }"
        ></div>

        <!-- 根据卡片类型渲染不同的骨架屏组件 -->
        <FormSkeleton v-if="card.type === 'form'" v-bind="card" :active="active" embedded />
        <TableSkeleton v-else-if="card.type === 'table'" v-bind="card" :active="active" />
        <ListSkeleton v-else-if="card.type === 'list'" v-bind="card" :active="active" embedded />
        <FilePreviewSkeleton v-else-if="card.type === 'filePreview'" v-bind="card" :active="active" embedded />
        <CardContentSkeleton v-else-if="card.type === 'fields'" v-bind="card" :active="active" embedded />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { DetailSkeletonProps } from './types'

withDefaults(defineProps<DetailSkeletonProps>(), {
  showHeader: true,
  cards: () => [
    { type: 'fields', fields: 6 },
    { type: 'fields', fields: 6 },
    { type: 'fields', fields: 6 },
  ],
  active: true,
})
</script>
```

## 使用示例

### 基础使用

#### 表格骨架屏

```vue
<template>
  <TableSkeleton 
    v-if="loading"
    :rows="10"
    :columns="5"
    :show-header="true"
    :show-pagination="true"
  />
  <a-table v-else :dataSource="data" :columns="columns" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { TableSkeleton } from '@/components/skeleton'

const loading = ref(true)
const data = ref([])

// 加载数据...
</script>
```

#### 表单骨架屏

```vue
<template>
  <FormSkeleton
    v-if="loading"
    :fields="8"
    layout="horizontal"
    :show-button="true"
  />
  <a-form v-else v-model="formData">
    <!-- 表单内容 -->
  </a-form>
</template>
```

#### 列表骨架屏

```vue
<template>
  <ListSkeleton
    v-if="loading"
    :count="6"
    :columns="2"
    :show-avatar="true"
    avatar-shape="circle"
    :rows="2"
  />
  <div v-else class="list">
    <!-- 列表内容 -->
  </div>
</template>
```

### 详情页骨架屏

```vue
<template>
  <DetailSkeleton
    v-if="loading"
    :show-header="true"
    :cards="[
      { type: 'fields', fields: 8, columns: 2, showTitle: true },
      { type: 'table', rows: 5, columns: 4, showTitle: true },
      { type: 'list', count: 3, showAvatar: true, showTitle: true },
      { type: 'filePreview', showTitle: true, showButton: true },
    ]"
  />
  <div v-else>
    <!-- 详情页内容 -->
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { DetailSkeleton } from '@/components/skeleton'

const loading = ref(true)

// 加载数据...
</script>
```

### 自定义配置示例

```vue
<template>
  <!-- 垂直布局的表单骨架屏 -->
  <FormSkeleton
    :fields="6"
    layout="vertical"
    :show-button="true"
    :active="true"
  />

  <!-- 网格布局的列表骨架屏 -->
  <ListSkeleton
    :count="9"
    :columns="3"
    :show-avatar="true"
    avatar-shape="square"
    :rows="3"
  />

  <!-- 嵌入模式的卡片内容骨架屏 -->
  <div class="my-card">
    <h3>卡片标题</h3>
    <CardContentSkeleton
      :fields="6"
      :columns="2"
      layout="horizontal"
      embedded
    />
  </div>
</template>
```

## 设计要点

### 1. 嵌入模式设计

嵌入模式允许骨架屏组件嵌入到其他容器中，自动去除内边距和背景色：

```typescript
interface EmbeddableSkeletonProps extends BaseSkeletonProps {
  embedded?: boolean  // 嵌入模式标志
}
```

```scss
// 嵌入模式样式
.is-embedded {
  padding: 0;
  background: transparent;
  border-radius: 0;
}
```

### 2. 组合式设计

`DetailSkeleton` 通过组合其他骨架屏组件，实现复杂页面的骨架：

```typescript
type CardConfig = 
  | CardConfigForm 
  | CardConfigTable 
  | CardConfigList 
  | CardConfigFilePreview 
  | CardConfigFields

interface DetailSkeletonProps {
  cards?: CardConfig[]  // 卡片配置数组
}
```

### 3. 动画性能优化

使用 CSS `animation` 而非 JavaScript 实现动画，性能更好：

```scss
.skeleton-animated {
  animation: skeleton-loading 1.5s ease-in-out infinite;
}

@keyframes skeleton-loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

### 4. 类型安全

完整的 TypeScript 类型定义，确保类型安全：

```typescript
// 联合类型确保卡片配置的类型安全
type CardConfig =
  | CardConfigForm
  | CardConfigTable
  | CardConfigList
  | CardConfigFilePreview
  | CardConfigFields

// 每种卡片类型都有明确的 type 字段
interface CardConfigForm {
  type: 'form'  // 类型判别
  // ...其他配置
}
```

## 最佳实践

### 1. 匹配真实内容结构

骨架屏应尽量匹配真实内容的结构：

```vue
<!-- ✅ 好的做法 -->
<TableSkeleton :rows="10" :columns="5" />  <!-- 与实际表格列数一致 -->

<!-- ❌ 不好的做法 -->
<TableSkeleton :rows="3" :columns="3" />   <!-- 与实际表格差异太大 -->
```

### 2. 合理使用动画

根据场景选择是否开启动画：

```vue
<!-- 快速加载场景，可关闭动画 -->
<TableSkeleton :active="false" />

<!-- 慢速加载场景，开启动画提升体验 -->
<TableSkeleton :active="true" />
```

### 3. 避免过度使用

不是所有加载场景都需要骨架屏，简单场景可以使用 Spin 组件：

```vue
<!-- 简单场景：使用 Spin -->
<a-spin :spinning="loading">
  <div>内容</div>
</a-spin>

<!-- 复杂场景：使用骨架屏 -->
<DetailSkeleton v-if="loading" />
<div v-else>内容</div>
```

### 4. 响应式设计

考虑移动端适配：

```vue
<ListSkeleton
  :columns="isMobile ? 1 : 2"
  :count="isMobile ? 5 : 10"
/>
```

## 性能优化

### 1. 减少 DOM 节点

对于大量重复元素，使用合理的数量：

```vue
<!-- ✅ 合理 -->
<TableSkeleton :rows="10" />

<!-- ❌ 过多 -->
<TableSkeleton :rows="100" />
```

### 2. 使用 CSS 而非 JS

动画效果使用 CSS 实现，避免 JS 计算：

```scss
// ✅ CSS 动画
.skeleton-animated {
  animation: skeleton-loading 1.5s ease-in-out infinite;
}

// ❌ 避免使用 JS 动画
```

### 3. 懒加载骨架屏

对于不在视口内的骨架屏，可以延迟渲染：

```vue
<template>
  <div v-if="isVisible">
    <TableSkeleton />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useIntersectionObserver } from '@vueuse/core'

const isVisible = ref(false)
// 使用 IntersectionObserver 判断是否可见
</script>
```

## 总结

本骨架屏组件库具有以下特点：

1. **设计合理**：类型系统完善，支持嵌入和组合
2. **功能丰富**：涵盖表格、表单、列表等常见场景
3. **易于使用**：简洁的 API，灵活的配置
4. **性能优化**：CSS 动画，合理的 DOM 结构
5. **可维护性**：模块化设计，易于扩展

通过合理使用骨架屏，可以显著提升用户体验，特别是在网络较慢或数据加载较久的场景下。
