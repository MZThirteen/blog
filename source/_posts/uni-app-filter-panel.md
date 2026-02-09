---
title: uni-app 动态筛选组件设计与实现
date: 2025-02-09 20:20:00
tags: 
  - uni-app
  - Vue3
  - 组件设计
  - 筛选组件
  - TypeScript
categories: 前端
---

## 概述

本文介绍一个基于 uni-app + Vue3 + wot-design-uni 的高度可配置筛选组件，适用于移动端列表筛选场景，支持单选、多选、级联选择、时间段等多种筛选类型。

## ✨ 功能特点

- ✅ **四种筛选类型**：单选、多选、多列选择（级联/多选）、时间段
- ✅ **配置化使用**：通过 JSON 配置快速创建筛选条件
- ✅ **联动功能**：支持单选/多选字段联动，支持接口异步联动
- ✅ **多列多选**：支持普通模式和强制联动模式（逐级选择）
- ✅ **时间快捷选择**：内置今天、本周、本月快捷按钮
- ✅ **自动隐藏空选项**：options 为空时自动隐藏筛选项
- ✅ **状态自动重置**：点击重置后，所有内部状态正确清空
- ✅ **完整 TypeScript 支持**：提供完整的类型定义
- ✅ **底部安全区适配**：自动适配 iPhone X 等刘海屏
- ✅ **响应式布局**：自适应不同屏幕尺寸

## 📦 安装依赖

组件依赖 `wot-design-uni` 和 `dayjs`，确保项目已安装：

```bash
pnpm add wot-design-uni dayjs
```

## 🚀 快速开始

### 基础用法

```vue
<template>
  <view>
    <filter-panel
      v-model="filterValue"
      title="筛选条件"
      :filter-items="filterItems"
      @confirm="handleConfirm"
      @reset="handleReset"
    />
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import type { FilterItem } from '@/components/filter-panel/types'

const filterValue = ref({})

const filterItems: FilterItem[] = [
  // 单选
  {
    label: '状态',
    field: 'status',
    type: 'radio',
    options: [
      { label: '待处理', value: '1' },
      { label: '处理中', value: '2' },
      { label: '已完成', value: '3' },
    ],
  },
  // 多选
  {
    label: '标签',
    field: 'tags',
    type: 'checkbox',
    options: [
      { label: '重要', value: 'important' },
      { label: '紧急', value: 'urgent' },
      { label: '普通', value: 'normal' },
    ],
  },
  // 时间段
  {
    label: '创建时间',
    field: 'createTime',
    type: 'daterange',
    format: 'YYYY-MM-DD',
    minDate: new Date(2020, 0, 1),
    maxDate: new Date(),
  },
]

const handleConfirm = (value: any) => {
  console.log('筛选值:', value)
}

const handleReset = () => {
  console.log('重置筛选')
}
</script>
```

## 📖 API 文档

### Props

| 参数 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| title | 弹窗标题 | `string` | `'筛选'` |
| filter-items | 筛选项配置数组 | `FilterItem[]` | `[]` |
| model-value / v-model | 筛选值对象 | `Record<string, any>` | `{}` |

### FilterItem 配置

#### 通用属性

| 参数 | 说明 | 类型 | 必填 |
| --- | --- | --- | --- |
| label | 筛选项显示标题 | `string` | 是 |
| field | 字段名（用于数据绑定） | `string` | 是 |
| type | 筛选类型 | `'radio' \| 'checkbox' \| 'colpicker' \| 'multicolpicker' \| 'daterange'` | 是 |

#### 单选/多选（radio/checkbox）专用

| 参数 | 说明 | 类型 | 必填 |
| --- | --- | --- | --- |
| options | 选项列表 | `FilterOption[]` | 是 |
| linkedFields | 关联的字段名数组（用于联动） | `string[]` | 否 |
| optionsChange | 联动回调函数（支持同步/异步） | `OptionsChangeCallback` | 否 |

**OptionsChangeCallback 类型：**

```typescript
type OptionsChangeCallback = (params: {
  field: string              // 当前字段名
  value: any                 // 当前选中值
  targetField: string        // 目标字段名
  formData: Record<string, any> // 所有表单数据
}) => FilterOption[] | void | Promise<FilterOption[] | void>
```

#### 多列选择器（colpicker）专用

| 参数 | 说明 | 类型 | 必填 |
| --- | --- | --- | --- |
| columns | 多列数据（二维数组） | `any[][]` | 是 |
| columnChange | 列变化回调（用于级联） | `Function` | 否 |

#### 多列多选（multicolpicker）专用

| 参数 | 说明 | 类型 | 必填 |
| --- | --- | --- | --- |
| columns | 多列数据（二维数组） | `any[][]` | 是 |
| columnChange | 列变化回调（支持异步） | `Function` | 否 |
| strictMode | 是否强制联动模式 | `boolean` | 否，默认 `false` |

**强制联动模式（strictMode: true）：**
- 逐级单选（最后一列多选）
- 自动切换到下一列
- 隐藏未激活的列
- 确认时验证所有列

#### 时间段（daterange）专用

| 参数 | 说明 | 类型 | 必填 |
| --- | --- | --- | --- |
| format | 数据格式 | `string \| 'timestamp'` | 否，默认 `'YYYY-MM-DD'` |
| displayFormat | 显示格式（用于UI展示） | `string` | 否，默认同 `format` |
| minDate | 最小可选日期 | `number \| Date` | 否 |
| maxDate | 最大可选日期 | `number \| Date` | 否 |

### Events

| 事件名 | 说明 | 回调参数 |
| --- | --- | --- |
| confirm | 点击确定按钮时触发 | `(value: Record<string, any>)` |
| reset | 点击重置按钮时触发 | `()` |
| change | 筛选项值改变时触发 | `(field: string, value: any)` |

### Slots

| 名称 | 说明 |
| --- | --- |
| trigger | 自定义触发按钮 |

### Methods

| 方法名 | 说明 | 参数 |
| --- | --- | --- |
| open | 打开筛选面板 | `()` |
| close | 关闭筛选面板 | `()` |

## 🎯 使用示例

### 1. 单选/多选联动（静态数据）

```typescript
const filterItems: FilterItem[] = [
  // 主控字段
  {
    label: '类别',
    field: 'category',
    type: 'radio',
    options: [
      { label: '电子产品', value: 'electronics' },
      { label: '图书', value: 'books' },
      { label: '服装', value: 'clothes' },
    ],
    linkedFields: ['subcategory'], // 关联子类别字段
    optionsChange: ({ value, targetField }) => {
      if (targetField === 'subcategory') {
        // 根据不同类别返回不同子类别
        if (value === 'electronics') {
          return [
            { label: '手机', value: 'phone' },
            { label: '电脑', value: 'computer' },
          ]
        }
        if (value === 'books') {
          return [
            { label: '小说', value: 'novel' },
            { label: '技术书籍', value: 'tech' },
          ]
        }
        if (value === 'clothes') {
          return [
            { label: '男装', value: 'men' },
            { label: '女装', value: 'women' },
          ]
        }
      }
    },
  },
  // 被联动字段
  {
    label: '子类别',
    field: 'subcategory',
    type: 'checkbox',
    options: [],
  },
]
```

### 2. 单选/多选联动（接口数据）

```typescript
// 模拟接口
const fetchSubItems = async (mainType: string) => {
  await new Promise(resolve => setTimeout(resolve, 300))
  const data = {
    typeA: [
      { label: '选项A1', value: 'a1' },
      { label: '选项A2', value: 'a2' },
    ],
    typeB: [
      { label: '选项B1', value: 'b1' },
      { label: '选项B2', value: 'b2' },
    ],
  }
  return data[mainType] || []
}

const filterItems: FilterItem[] = [
  {
    label: '主类型',
    field: 'mainType',
    type: 'radio',
    options: [
      { label: '类型A', value: 'typeA' },
      { label: '类型B', value: 'typeB' },
    ],
    linkedFields: ['subItem'],
    optionsChange: async ({ value, targetField }) => {
      if (targetField === 'subItem') {
        if (!value) return []

        try {
          uni.showLoading({ title: '加载中...' })
          const items = await fetchSubItems(value)
          return items
        } catch (error) {
          uni.showToast({ title: '加载失败', icon: 'none' })
          return []
        } finally {
          uni.hideLoading()
        }
      }
    },
  },
  {
    label: '子项',
    field: 'subItem',
    type: 'radio',
    options: [], // 初始为空，通过联动加载
  },
]
```

### 3. 多列选择器（级联单选）

```typescript
const areaDataMap = {
  '110000': [ // 北京市
    { label: '东城区', value: '110101' },
    { label: '西城区', value: '110102' },
    { label: '朝阳区', value: '110105' },
  ],
  '310000': [ // 上海市
    { label: '黄浦区', value: '310101' },
    { label: '徐汇区', value: '310104' },
    { label: '浦东新区', value: '310115' },
  ],
}

const filterItems: FilterItem[] = [
  {
    label: '所在地区',
    field: 'area',
    type: 'colpicker',
    columns: [
      [
        { label: '北京市', value: '110000' },
        { label: '上海市', value: '310000' },
      ],
    ],
    columnChange: ({ selectedItem, resolve, finish }) => {
      const areaData = areaDataMap[selectedItem.value]
      if (areaData) {
        resolve(areaData) // 返回区县数据
      } else {
        finish() // 结束级联
      }
    },
  },
]
```

**数据结构：**
- `modelValue`: `['310000', '310104']` （省市区的 value 数组）
- 显示文本: "上海市/徐汇区"

### 4. 多列多选（普通模式）

```typescript
const filterItems: FilterItem[] = [
  {
    label: '组织架构',
    field: 'organization',
    type: 'multicolpicker',
    columns: [
      // 第一列：部门
      [
        { label: '技术部', value: 'tech' },
        { label: '市场部', value: 'market' },
      ],
      // 第二列：小组
      [
        { label: '技术部/前端组', value: 'tech_fe' },
        { label: '技术部/后端组', value: 'tech_be' },
        { label: '市场部/推广组', value: 'market_promo' },
        { label: '市场部/运营组', value: 'market_ops' },
      ],
    ],
    columnChange: ({ columnIndex, selectedValues }) => {
      if (columnIndex === 0) {
        const groupMap = {
          tech: [
            { label: '技术部/前端组', value: 'tech_fe' },
            { label: '技术部/后端组', value: 'tech_be' },
          ],
          market: [
            { label: '市场部/推广组', value: 'market_promo' },
            { label: '市场部/运营组', value: 'market_ops' },
          ],
        }
        
        if (!selectedValues || selectedValues.length === 0) {
          return [
            ...groupMap.tech,
            ...groupMap.market,
          ]
        }
        
        const groups = []
        selectedValues.forEach(dept => {
          groups.push(...(groupMap[dept] || []))
        })
        return groups
      }
    },
  },
]
```

**数据结构：**
- `modelValue`: `[['tech', 'market'], ['tech_fe', 'market_ops']]` （二维数组）
- 显示文本: "技术部、市场部、技术部/前端组、市场部/运营组"

### 5. 多列多选（强制联动模式）

```typescript
const filterItems: FilterItem[] = [
  {
    label: '层级选择',
    field: 'hierarchy',
    type: 'multicolpicker',
    strictMode: true, // 开启强制联动模式
    columns: [
      [
        { label: '一级分类A', value: 'categoryA' },
        { label: '一级分类B', value: 'categoryB' },
      ],
      [],
    ],
    columnChange: async ({ columnIndex, selectedValues }) => {
      if (columnIndex === 0) {
        await new Promise(resolve => setTimeout(resolve, 300))
        
        const subMap = {
          categoryA: [
            { label: '子分类A1', value: 'sub_a1' },
            { label: '子分类A2', value: 'sub_a2' },
          ],
          categoryB: [
            { label: '子分类B1', value: 'sub_b1' },
            { label: '子分类B2', value: 'sub_b2' },
          ],
        }
        
        return subMap[selectedValues[0]] || []
      }
    },
  },
]
```

**数据结构：**
- `modelValue`: `[['categoryA'], ['sub_a1', 'sub_a2']]` （二维数组）
- 显示文本: "子分类A1、子分类A2" （**只显示最后一列**）

**特点（strictMode: true）：**
- ✅ 逐级单选：第一列只能单选，选中后自动跳转到第二列
- ✅ 最后一列多选：最后一列支持多选
- ✅ 自动隐藏未激活列：只显示当前列和已选列
- ✅ 强制验证：确认时检查所有列是否都有选择
- ✅ 显示优化：只显示最后一列的选中项

### 6. 时间段选择

```typescript
const filterItems: FilterItem[] = [
  // 基础用法
  {
    label: '创建时间',
    field: 'createTime',
    type: 'daterange',
    format: 'YYYY-MM-DD',
    minDate: new Date(2020, 0, 1),
    maxDate: new Date(),
  },
  
  // 返回时间戳 + 自定义显示格式
  {
    label: '更新时间',
    field: 'updateTime',
    type: 'daterange',
    format: 'timestamp',           // 返回时间戳（毫秒）
    displayFormat: 'YYYY年MM月DD日', // 页面显示中文格式
  },
  
  // 自定义格式
  {
    label: '处理时间',
    field: 'processTime',
    type: 'daterange',
    format: 'YYYY-MM-DD HH:mm:ss',
    displayFormat: 'YYYY-MM-DD HH:mm',
  },
]
```

**内置快捷按钮：**
- **今天**：00:00:00 - 23:59:59
- **本周**：本周一 - 今天（中国习惯，周一为一周开始）
- **本月**：本月1号 - 今天

## 💡 核心实现

### 1. 字段联动处理

```typescript
const handleFieldChange = async (field: string, value: any) => {
  const currentItem = props.filterItems.find(item => item.field === field)

  // 如果有联动回调，执行联动逻辑
  if (currentItem?.optionsChange && currentItem?.linkedFields) {
    for (const targetField of currentItem.linkedFields) {
      try {
        const newOptions = await currentItem.optionsChange?.({
          field,
          value,
          targetField,
          formData: formData.value,
        })

        if (newOptions) {
          // 更新目标字段的选项
          dynamicOptions.value[targetField] = newOptions

          // 清空目标字段的值（避免选中无效选项）
          const targetItem = props.filterItems.find(item => item.field === targetField)
          if (targetItem) {
            if (targetItem.type === 'checkbox') {
              formData.value[targetField] = []
            } else {
              formData.value[targetField] = ''
            }
          }
        }
      } catch (error) {
        console.error(`联动更新 ${targetField} 失败:`, error)
      }
    }
  }

  emit('change', field, value)
}
```

### 2. 自动隐藏空选项

```typescript
const shouldShowFilterItem = (item: FilterItem) => {
  // 对于 radio 和 checkbox 类型，如果 options 为空则隐藏
  if (item.type === 'radio' || item.type === 'checkbox') {
    const options = getFieldOptions(item.field)
    return options && options.length > 0
  }
  // 其他类型默认显示
  return true
}
```

### 3. 动态选项获取

```typescript
const getFieldOptions = (field: string) => {
  return (
    dynamicOptions.value[field] ||
    props.filterItems.find(item => item.field === field)?.options ||
    []
  )
}
```

## 📝 类型定义

```typescript
// 筛选选项类型
export interface FilterOption {
  label: string
  value: any
  disabled?: boolean
}

// 选项联动回调类型
export type OptionsChangeCallback = (
  params: {
    field: string
    value: any
    targetField: string
    formData: Record<string, any>
  },
) => FilterOption[] | void | Promise<FilterOption[] | void>

// 筛选项配置类型
export interface FilterItem {
  label: string
  field: string
  type: 'radio' | 'checkbox' | 'colpicker' | 'multicolpicker' | 'daterange'
  options?: FilterOption[]
  optionsChange?: OptionsChangeCallback
  linkedFields?: string[]
  columns?: any[]
  columnChange?: any
  strictMode?: boolean
  minDate?: number | Date
  maxDate?: number | Date
  format?: string
  displayFormat?: string
}

// 筛选值类型
export type FilterValue = Record<string, any>
```

## 💡 设计亮点

### 1. 配置驱动

通过 JSON 配置快速构建筛选条件，无需编写大量模板代码。

### 2. 异步联动支持

`optionsChange` 回调支持返回 Promise，可以调用接口动态加载选项。

### 3. 智能状态管理

- 联动时自动清空相关字段的值
- 重置时正确清空所有组件的内部状态
- 空选项自动隐藏

### 4. 灵活的多列多选

提供普通模式和强制联动模式两种方式，适应不同的业务场景。

### 5. 底部安全区适配

```scss
.safe-area-bottom {
  padding-bottom: calc(24rpx + constant(safe-area-inset-bottom));
  padding-bottom: calc(24rpx + env(safe-area-inset-bottom));
}
```

## ❓ 常见问题

### 1. 如何设置默认筛选值？

```typescript
const filterValue = ref({
  status: '1',           // 单选默认值
  tags: ['tag1', 'tag2'], // 多选默认值
  area: ['110000', '110101'], // colpicker 默认值
  organization: [['tech'], ['tech_fe']], // multicolpicker 默认值
  createTime: ['2024-01-01', '2024-01-31'], // daterange 默认值
})
```

### 2. 如何判断是否有筛选条件？

```typescript
const hasActiveFilter = computed(() => {
  return Object.values(filterValue.value).some(value => {
    if (Array.isArray(value)) {
      if (value.length === 0) return false
      return value.some(v => {
        if (Array.isArray(v)) {
          return v.length > 0 && v.some(vv => vv !== '')
        }
        return v !== ''
      })
    }
    return value !== '' && value !== null && value !== undefined
  })
})
```

### 3. 如何获取筛选后的参数用于API请求？

```typescript
const handleFilterConfirm = (value: any) => {
  // 构建 API 参数
  const params = {
    ...value,
    // 如果需要特殊处理某些字段
    area: value.area?.join(','), // 多列选择转为字符串
  }
  
  // 调用 API
  api.getList(params)
}
```

## 📂 组件结构

```
filter-panel/
├── index.vue                    # 主组件
├── types.ts                     # TypeScript 类型定义
├── components/                  # 子组件
│   ├── FilterOptions.vue        # 单选/多选组件
│   ├── FilterDateRange.vue      # 时间段组件
│   ├── FilterColPicker.vue      # 多列选择器（级联单选）
│   └── FilterMultiColPicker.vue # 多列多选组件
└── index.ts                     # 导出入口
```

## 总结

这个筛选组件具有以下优势：

1. **灵活性**：支持多种筛选类型，配置化使用
2. **联动性**：支持同步/异步联动，适应复杂业务场景
3. **易用性**：简洁的 API，完善的类型定义
4. **健壮性**：智能状态管理，自动处理边界情况
5. **移动端优化**：底部安全区适配，响应式布局

适用于 uni-app 开发的各种移动端列表筛选场景，大大提升开发效率。
