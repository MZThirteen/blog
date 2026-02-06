---
title: Vue3 动态筛选表单组件设计
date: 2025-02-06 17:45:00
tags: 
  - Vue3
  - 组件设计
  - 表单
  - TypeScript
  - 响应式布局
categories: 前端
---

## 概述

本文介绍一个基于 Vue3 + TypeScript + Ant Design Vue 的高度灵活的筛选表单组件，支持动态布局、展开收起、响应式设计等功能，适用于后台管理系统的列表筛选场景。

## 核心特性

- ✅ **动态布局**：支持自定义每行显示项目数（1-n 列）
- ✅ **智能计算**：自动计算表单项和按钮的栅格布局
- ✅ **展开收起**：超过指定行数自动显示展开/收起功能
- ✅ **插槽驱动**：完全自定义表单项内容
- ✅ **响应式设计**：移动端自适应
- ✅ **类型安全**：完整的 TypeScript 类型定义

## 组件接口设计

### Props

```typescript
interface Props {
  /** 表单数据 */
  formData: Record<string, any>
  /** 表单项配置 */
  formItems: FormItem[]
  /** 每行显示的项目数，默认3 */
  itemsPerRow?: number
  /** 搜索按钮文本，默认'查询' */
  searchText?: string
  /** 标签宽度，默认72px */
  labelWidth?: number
  /** 是否启用展开收起功能 */
  enableExpand?: boolean
  /** 超过多少行时显示展开收起按钮，默认3 */
  expandRowThreshold?: number
}
```

### FormItem 类型

```typescript
export interface FormItem {
  slotName: string  // 插槽名称
  /**
   * 表单项占用的份数
   * - 建议值：1 = 1/3宽度，2 = 2/3宽度，3 = 整行宽度（基于默认 itemsPerRow=3）
   * - 实际宽度 = (24 / itemsPerRow) * span
   * - 如果 span > itemsPerRow，会自动限制为整行宽度
   */
  span?: number
  [key: string]: any
}
```

### Events

```typescript
interface Emits {
  (e: 'reset'): void   // 重置事件
  (e: 'search'): void  // 搜索事件
}
```

## 核心功能实现

### 1. 栅格布局计算

基于 Ant Design 的 24 栅格系统，根据 `itemsPerRow` 和 `span` 动态计算列宽：

```typescript
/**
 * 根据 span 计算栅格列数（Ant Design 24 栅格系统）
 * 
 * @param span 占用份数（建议值：1-itemsPerRow）
 * @returns 栅格列数（24/itemsPerRow * span，最大24）
 */
const getColSpan = (span: number) => {
  const colsPerItem = 24 / props.itemsPerRow
  const calculatedSpan = colsPerItem * span
  // 限制最大值为24，避免超出栅格系统范围导致布局溢出
  return Math.min(calculatedSpan, 24)
}
```

**示例**：
- `itemsPerRow = 3`（默认）
  - `span = 1` → 8 列（1/3 宽度）
  - `span = 2` → 16 列（2/3 宽度）
  - `span = 3` → 24 列（整行）

### 2. 布局智能计算

计算表单项累计占用的行空间，用于判断是否显示展开按钮：

```typescript
/**
 * 计算表单项累计占用的行空间
 * 
 * @param items 表单项列表
 * @returns 返回布局信息，包含最后一行占用的空间和总行数
 */
const calculateLayout = (items: FormItem[]) => {
  let currentRowSpan = 0
  let rows = 0

  for (const item of items) {
    const itemSpan = item.span || 1
    // 限制 itemSpan 不超过 itemsPerRow，避免逻辑错误
    const normalizedSpan = Math.min(itemSpan, props.itemsPerRow)

    // 如果当前行放不下这个项,则开始新行
    if (currentRowSpan + normalizedSpan > props.itemsPerRow) {
      rows++
      currentRowSpan = normalizedSpan
    } else {
      currentRowSpan += normalizedSpan
      if (currentRowSpan === props.itemsPerRow) {
        rows++
        currentRowSpan = 0
      }
    }
  }

  // 如果最后还有未满的行
  if (currentRowSpan > 0) {
    rows++
  }

  return {
    lastRowSpan: currentRowSpan,
    totalRows: rows,
  }
}
```

### 3. 展开收起逻辑

根据配置和实际布局，智能判断是否显示展开按钮：

```typescript
/**
 * 是否显示展开按钮
 * 根据 enableExpand 和 expandRowThreshold 配置判断
 */
const showExpandButton = computed(() => {
  // 如果未启用展开收起功能，直接返回false
  if (!props.enableExpand) return false

  const items = props.formItems || []
  if (items.length === 0) return false

  // 计算表单项的布局
  const { lastRowSpan, totalRows } = calculateLayout(items)

  // 计算按钮是否会占用新的一行
  const buttonOccupiesNewRow = lastRowSpan === 0 || lastRowSpan === props.itemsPerRow
  const totalRowsWithButton = buttonOccupiesNewRow ? totalRows + 1 : totalRows

  // 如果按钮独占一行，且总行数（不含按钮行）不超过阈值，则不显示展开按钮
  if (buttonOccupiesNewRow && totalRows <= props.expandRowThreshold) {
    return false
  }

  // 根据配置的行数阈值判断是否显示展开按钮
  return totalRowsWithButton > props.expandRowThreshold
})
```

### 4. 可见表单项计算

收起状态时，只显示阈值行数的项目，并为按钮预留空间：

```typescript
/**
 * 可见的表单项
 * - 展开状态或无展开按钮时：显示所有项
 * - 收起状态：只显示阈值行数的项目，并为按钮预留空间
 */
const visibleFormItems = computed(() => {
  // 如果展开或不显示展开按钮，显示所有项
  if (expanded.value || !showExpandButton.value) {
    return props.formItems
  }

  // 收起状态：只显示阈值行数的项目
  const items = props.formItems || []
  const collapsedItems: FormItem[] = []
  let currentRowSpan = 0
  let currentRow = 1

  for (const item of items) {
    const itemSpan = item.span || 1
    const normalizedSpan = Math.min(itemSpan, props.itemsPerRow)

    // 如果当前行放不下，需要换行
    if (currentRowSpan + normalizedSpan > props.itemsPerRow) {
      currentRow++
      currentRowSpan = 0

      // 如果已经到达最后一行，需要为按钮预留空间
      if (currentRow > props.expandRowThreshold) {
        break
      }
    }

    // 如果是最后一行，需要预留按钮空间（至少1个位置）
    if (currentRow === props.expandRowThreshold) {
      const availableSpace = props.itemsPerRow - 1 // 为按钮预留空间
      if (currentRowSpan + normalizedSpan > availableSpace) {
        break
      }
    }

    collapsedItems.push(item)
    currentRowSpan += normalizedSpan
  }

  return collapsedItems
})
```

### 5. 按钮位置计算

智能计算按钮应占据的栅格数：

```typescript
/**
 * 计算按钮所占份数
 * 如果最后一行有剩余空间，按钮占据剩余空间；否则独占一行
 */
const buttonSpan = computed(() => {
  const items = visibleFormItems.value || []
  const { lastRowSpan } = calculateLayout(items)

  // 如果最后一行有剩余空间，按钮占据剩余空间
  if (lastRowSpan > 0 && lastRowSpan < props.itemsPerRow) {
    return props.itemsPerRow - lastRowSpan
  }

  // 否则按钮独占一行
  return props.itemsPerRow
})
```

## 使用示例

### 基础使用

```vue
<template>
  <FilterForm
    :form-data="formData"
    :form-items="formItems"
    :items-per-row="3"
    @search="handleSearch"
    @reset="handleReset"
  >
    <!-- 使用插槽自定义表单项 -->
    <template #name="{ item }">
      <a-form-item label="姓名">
        <a-input v-model:value="formData.name" placeholder="请输入姓名" />
      </a-form-item>
    </template>

    <template #status="{ item }">
      <a-form-item label="状态">
        <a-select v-model:value="formData.status" placeholder="请选择状态">
          <a-select-option value="1">启用</a-select-option>
          <a-select-option value="0">禁用</a-select-option>
        </a-select>
      </a-form-item>
    </template>

    <template #date="{ item }">
      <a-form-item label="日期">
        <a-range-picker v-model:value="formData.dateRange" />
      </a-form-item>
    </template>
  </FilterForm>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { FilterForm } from '@/components/FilterForm'
import type { FormItem } from '@/components/FilterForm'

const formData = ref({
  name: '',
  status: undefined,
  dateRange: [],
})

const formItems: FormItem[] = [
  { slotName: 'name', span: 1 },
  { slotName: 'status', span: 1 },
  { slotName: 'date', span: 1 },
]

const handleSearch = () => {
  console.log('搜索', formData.value)
}

const handleReset = () => {
  formData.value = {
    name: '',
    status: undefined,
    dateRange: [],
  }
}
</script>
```

### 自定义布局

#### 示例1：两列布局

```typescript
const formItems: FormItem[] = [
  { slotName: 'name', span: 1 },      // 1/2 宽度
  { slotName: 'status', span: 1 },    // 1/2 宽度
  { slotName: 'date', span: 2 },      // 整行宽度
]

// itemsPerRow = 2
```

#### 示例2：不等宽布局

```typescript
const formItems: FormItem[] = [
  { slotName: 'name', span: 1 },      // 1/3 宽度
  { slotName: 'status', span: 2 },    // 2/3 宽度
  { slotName: 'date', span: 3 },      // 整行宽度
]

// itemsPerRow = 3（默认）
```

### 配置展开收起

```vue
<FilterForm
  :form-data="formData"
  :form-items="formItems"
  :enable-expand="true"
  :expand-row-threshold="2"
  @search="handleSearch"
  @reset="handleReset"
>
  <!-- 表单项插槽 -->
</FilterForm>
```

**说明**：
- `enable-expand`：是否启用展开收起功能
- `expand-row-threshold`：超过多少行时显示展开按钮（默认3）

### 自定义按钮

```vue
<FilterForm
  :form-data="formData"
  :form-items="formItems"
  search-text="搜索"
  @search="handleSearch"
  @reset="handleReset"
>
  <!-- 表单项插槽 -->
  
  <!-- 自定义按钮插槽 -->
  <template #actions>
    <a-button @click="handleExport">导出</a-button>
    <a-button type="primary" @click="handleAdd">新增</a-button>
  </template>
</FilterForm>
```

## 样式定制

### CSS 变量

组件使用 CSS 变量控制标签宽度：

```scss
.filter-form-container {
  --label-width: 72px;  // 可通过 props 动态设置
}
```

### 自定义样式

```scss
// 修改标签宽度
<FilterForm :label-width="100" ... />

// 覆盖样式
:deep(.filter-form) {
  .ant-form-item-label {
    // 自定义标签样式
  }
  
  .form-buttons {
    // 自定义按钮样式
  }
}
```

### 响应式断点

组件内置响应式设计，移动端自动调整：

```scss
@media (width <= 768px) {
  // 小屏幕：按钮靠右对齐
  .form-buttons {
    width: 100%;
    justify-content: flex-end;
  }
}
```

## 设计亮点

### 1. 智能布局算法

- 自动计算表单项换行
- 智能预留按钮空间
- 避免布局溢出

### 2. 插槽驱动设计

- 完全自定义表单项内容
- 支持任意复杂的表单控件
- 不侵入业务逻辑

### 3. 性能优化

- 使用 `computed` 缓存计算结果
- 只渲染可见的表单项
- 最小化 DOM 操作

### 4. 类型安全

- 完整的 TypeScript 类型定义
- Props 和 Emits 类型推导
- 开发时类型提示

## 最佳实践

### 1. 合理设置 span

```typescript
// ✅ 推荐：根据表单项内容设置合理的宽度
const formItems: FormItem[] = [
  { slotName: 'name', span: 1 },        // 短输入框
  { slotName: 'address', span: 2 },     // 长输入框
  { slotName: 'description', span: 3 }, // 多行文本，占满一行
]

// ❌ 避免：所有项都设置相同宽度
```

### 2. 控制表单项数量

```typescript
// ✅ 推荐：筛选项不超过 6-8 个
const formItems: FormItem[] = [
  // ... 6-8 个常用筛选项
]

// ❌ 避免：过多筛选项影响用户体验
```

### 3. 合理使用展开收起

```vue
<!-- ✅ 推荐：筛选项较多时启用展开收起 -->
<FilterForm
  :form-items="formItems"
  :enable-expand="formItems.length > 6"
  :expand-row-threshold="2"
/>
```

### 4. 响应式设计

```typescript
// 根据屏幕宽度动态调整每行显示数
import { useWindowSize } from '@vueuse/core'

const { width } = useWindowSize()
const itemsPerRow = computed(() => {
  if (width.value < 768) return 1   // 移动端：1列
  if (width.value < 1200) return 2  // 平板：2列
  return 3                           // 桌面：3列
})
```

## 总结

这个筛选表单组件具有以下优势：

1. **灵活性**：插槽驱动，支持任意表单控件
2. **智能性**：自动计算布局，无需手动配置
3. **可维护性**：代码结构清晰，注释完善
4. **性能**：使用 computed 优化，按需渲染
5. **类型安全**：完整的 TypeScript 支持

适用于各种后台管理系统的列表筛选场景，大大提升开发效率和用户体验。
