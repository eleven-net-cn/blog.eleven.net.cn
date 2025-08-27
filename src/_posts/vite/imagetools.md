---
title: ImageTools 指南
date: 2025-08-27 03:11:02
category: Vite
tags: [Vite, 性能优化, 图片优化]
cover: 
---

> 基于 [vite-imagetools](https://github.com/JonasKruckenberg/imagetools) 的图片优化解决方案，官方的文档太过于简单，因此，借助 AI 整理了完整有效的使用指南。
> 
> 当前文档适用版本 vite-imagetools 6.2.9

## 📖 简介

ImageTools 是一个强大的 Vite 插件，专门负责重要图片的格式转换（PNG/JPG → WebP/AVIF），通过查询参数的方式在构建时对图片进行优化处理。

**⚠️ 重要提醒**：图片必须通过 `import` 导入，才能让 imagetools 插件工作生效。

## ⚙️ 插件配置参数

### `include` - 文件包含规则

**作用**：决定哪些文件会被 imagetools 插件处理  
**类型**：`string | RegExp | Array<string | RegExp>`  
**默认值**：`'**/*.(heif,avif,jpeg,jpg,png,tiff,webp,gif)?*'`

```typescript
imagetools({
  // 只处理 assets 目录下的图片
  include: 'src/assets/**/*.{jpg,jpeg,png,webp}',

  // 或使用正则表达式
  include: /^.*\.(jpg|jpeg|png|webp)$/,

  // 或使用数组处理多个规则
  include: ['src/assets/**/*.{jpg,jpeg,png}', 'src/images/**/*.webp'],
})
```

**编译结果影响**：

- ✅ 匹配的文件：会被 imagetools 处理，支持查询参数
- ❌ 不匹配的文件：直接复制到 dist，不支持查询参数

### `exclude` - 文件排除规则

**作用**：指定哪些文件不被 imagetools 处理，即使匹配了 include  
**类型**：`string | RegExp | Array<string | RegExp>`  
**默认值**：`'public/**/*'`

```typescript
imagetools({
  exclude: [
    'public/**/*', // 排除 public 目录
    'src/assets/icons/**/*', // 排除图标目录
    /.*\.svg$/, // 排除所有 SVG 文件
  ],
})
```

**编译结果影响**：

- 被排除的文件不会被处理，即使路径匹配 `include` 规则
- 这些文件会按照 Vite 的默认规则处理

### `defaultDirectives` - 默认处理指令

**作用**：为所有图片设置默认的处理参数  
**类型**：`URLSearchParams | ((url: URL, metadata: () => Promise<Metadata>) => URLSearchParams)`

#### 静态配置方式：

```typescript
imagetools({
  defaultDirectives: new URLSearchParams({
    format: 'webp', // 默认转换为 WebP
    quality: '85', // 默认质量 85%
    progressive: 'true', // 默认启用渐进式加载
  }),
})
```

#### 动态配置方式：

```typescript
imagetools({
  defaultDirectives: (url, metadata) => {
    const params = new URLSearchParams()

    // 根据文件路径设置不同默认值
    if (url.pathname.includes('/hero/')) {
      params.set('format', 'webp')
      params.set('quality', '90')
      params.set('w', '1920')
    } else if (url.pathname.includes('/thumb/')) {
      params.set('format', 'webp')
      params.set('quality', '75')
      params.set('w', '300')
      params.set('h', '300')
      params.set('fit', 'cover')
    } else {
      params.set('format', 'webp')
      params.set('quality', '85')
    }

    return params
  },
})
```

**编译结果影响**：

- 默认指令会与查询参数合并，查询参数优先级更高
- 如果没有查询参数，会使用默认指令处理

### `removeMetadata` - 移除元数据

**作用**：是否移除图片的 EXIF、GPS、作者等隐私信息  
**类型**：`boolean`  
**默认值**：`true`

```typescript
imagetools({
  removeMetadata: true, // 移除元数据，减小文件大小
})
```

**编译结果影响**：

- `true`：移除所有元数据，文件更小，保护隐私
- `false`：保留元数据，文件稍大，保留原始信息

### `namedExports` - 命名导出

**作用**：是否生成命名导出（如 srcset、metadata）  
**类型**：`boolean`  
**默认值**：`undefined`

```typescript
imagetools({
  namedExports: true,
})
```

**编译结果影响**：

```typescript
// namedExports: true 时
import img, { srcset, metadata } from './pic.jpg?w=400;800&as=srcset'

// namedExports: false 时
import img from './pic.jpg?w=400;800&as=srcset' // 只有默认导出
```

## 🎯 查询参数详解

### 尺寸控制参数

#### `w` - 宽度设置

**作用**：设置输出图片的宽度（像素）

```typescript
// 单一宽度
import img from './pic.jpg?w=400'
// 编译结果：生成 400px 宽的图片

// 多个宽度（响应式）
import img from './pic.jpg?w=400;800;1200'
// 编译结果：生成 3 个不同宽度的图片文件
```

#### `h` - 高度设置

**作用**：设置输出图片的高度（像素）

```typescript
import img from './pic.jpg?h=300'
// 编译结果：生成 300px 高的图片，宽度按比例缩放

import img from './pic.jpg?w=400&h=300'
// 编译结果：生成 400x300 的图片，可能会裁剪
```

#### `aspect` - 宽高比

**作用**：强制设置图片的宽高比  
**格式**：`16:9`、`4:3`、`1:1` 等

```typescript
import img from './pic.jpg?w=400&aspect=16:9'
// 编译结果：生成 400x225 的图片（16:9 比例）
```

#### `fit` - 适配模式

**作用**：当指定了宽度和高度时，决定如何处理图片以适应目标尺寸

```typescript
// cover：裁剪填满（默认）
import cover from './pic.jpg?w=400&h=300&fit=cover'
// 编译结果：400x300，可能裁剪图片边缘

// contain：完整显示
import contain from './pic.jpg?w=400&h=300&fit=contain'
// 编译结果：图片完整显示在 400x300 区域内，可能有空白

// fill：拉伸填满
import fill from './pic.jpg?w=400&h=300&fit=fill'
// 编译结果：强制拉伸到 400x300，可能变形

// inside：缩小到区域内
import inside from './pic.jpg?w=400&h=300&fit=inside'
// 编译结果：只缩小，不放大

// outside：放大到覆盖区域
import outside from './pic.jpg?w=400&h=300&fit=outside'
// 编译结果：放大到完全覆盖区域
```

#### `position` - 裁剪位置

**作用**：当使用 `fit=cover` 时，指定从图片的哪个部分开始裁剪

```typescript
import img from './pic.jpg?w=400&h=300&fit=cover&position=top'
// 编译结果：从图片顶部开始裁剪

// 可选值：top, right, bottom, left, center
// 也可以组合：top left, bottom right 等
```

### 格式转换参数

#### `format` - 输出格式

**作用**：转换图片格式  
**支持格式**：`webp`、`avif`、`jpeg`、`jpg`、`png`、`tiff`、`gif`、`heif`

```typescript
// 单一格式
import webp from './pic.jpg?format=webp'
// 编译结果：生成 .webp 文件

// 多格式输出
import multi from './pic.jpg?format=webp;avif;jpg'
// 编译结果：生成 3 个不同格式的文件
```

#### `quality` - 图片质量

**作用**：设置有损压缩的质量  
**范围**：0-100  
**适用格式**：JPEG、WebP（有损模式）、AVIF

```typescript
import high from './pic.jpg?format=webp&quality=95'
// 编译结果：高质量 WebP（文件较大）

import low from './pic.jpg?format=webp&quality=60'
// 编译结果：低质量 WebP（文件较小）
```

**推荐质量设置**：

- **WebP**: `quality=85` 是视觉无损的最佳平衡点
- **AVIF**: `quality=80` 提供更好的压缩率

#### `progressive` - 渐进式加载

**作用**：生成支持渐进式加载的图片  
**支持格式**：只有 JPEG

```typescript
import prog from './pic.jpg?format=jpeg&progressive=true'
// 编译结果：生成渐进式 JPEG，支持逐步加载显示
```

#### `lossless` - 无损压缩

**作用**：对支持的格式使用无损压缩  
**支持格式**：WebP、PNG

```typescript
import lossless from './pic.png?format=webp&lossless=true'
// 编译结果：生成无损 WebP，质量完美但文件较大
```

### 视觉效果参数

#### `blur` - 模糊效果

**作用**：应用高斯模糊  
**范围**：0.3-1000（实际常用 1-20）

```typescript
import blurred from './pic.jpg?blur=5'
// 编译结果：应用 5px 半径的高斯模糊

// 常用值：1-10，数值越大越模糊
```

#### `grayscale` - 灰度转换

**作用**：转换为灰度图片

```typescript
import gray from './pic.jpg?grayscale=true'
// 编译结果：转换为灰度图片
```

#### `invert` - 颜色反转

**作用**：反转所有颜色（类似底片效果）

```typescript
import inverted from './pic.jpg?invert=true'
// 编译结果：反转所有颜色
```

#### `tint` - 色调调整

**作用**：给图片添加颜色色调  
**格式**：十六进制颜色值（不需要 # 前缀）

```typescript
import tinted from './pic.jpg?tint=ff0000'
// 编译结果：添加红色色调

// 使用十六进制颜色值，不需要 # 前缀
```

#### `rotate` - 旋转

**作用**：旋转图片  
**范围**：任意角度

```typescript
import rotated from './pic.jpg?rotate=90'
// 编译结果：顺时针旋转 90 度

// 支持任意角度，负数表示逆时针
```

#### `flip` / `flop` - 翻转

**作用**：翻转图片

```typescript
import flipped from './pic.jpg?flip=true'
// 编译结果：水平翻转（左右翻转）

import flopped from './pic.jpg?flop=true'
// 编译结果：垂直翻转（上下翻转）
```

### 高级参数

#### `background` - 背景色

**作用**：为透明区域或空白区域设置背景色

```typescript
import withBg from './pic.png?w=400&h=300&fit=contain&background=ffffff'
// 编译结果：透明区域或空白区域填充白色

// 用于 PNG 转 JPEG 时移除透明度
```

#### `flatten` - 图层合并

**作用**：将透明图层合并到背景上

```typescript
import flattened from './pic.png?flatten=true&background=ffffff'
// 编译结果：将透明图层合并到白色背景上
```

#### `normalize` - 归一化

**作用**：增强图片对比度

```typescript
import normalized from './pic.jpg?normalize=true'
// 编译结果：增强图片对比度，拉伸亮度分布
```

#### `median` - 中值滤波

**作用**：去除图片噪点

```typescript
import filtered from './pic.jpg?median=3'
// 编译结果：应用 3x3 中值滤波，去除噪点
```

## 📤 输出格式详解

### 默认输出（URL 字符串）

```typescript
import img from './pic.jpg?w=400&format=webp'
// 类型：string
// 值："/assets/pic.hash.webp"
// 用途：直接用作 img src
```

### `as=srcset` - 响应式图片集

```typescript
import { srcset } from './pic.jpg?w=400;800;1200&format=webp&as=srcset'
// 类型：string
// 值："/assets/pic-400.hash.webp 400w, /assets/pic-800.hash.webp 800w, /assets/pic-1200.hash.webp 1200w"
// 用途：img 标签的 srcset 属性
```

### `as=metadata` - 图片元信息

```typescript
import { metadata } from './pic.jpg?w=400&as=metadata'
// 类型：{ src: string, width: number, height: number, format: string }
// 值：{ src: "/assets/pic.hash.webp", width: 400, height: 300, format: "webp" }
// 用途：获取处理后的图片信息
```

### `as=img` - 图片对象

```typescript
import img from './pic.jpg?w=400&as=img'
// 类型：{ src: string, w: number, h: number }
// 值：{ src: "/assets/pic.hash.webp", w: 400, h: 300 }
// 用途：同时获取 URL 和尺寸信息
```

### `as=picture` - Picture 元素数据

```typescript
import picture from './pic.jpg?format=webp;avif;jpg&w=800&as=picture'
// 类型：{ sources: Record<string, string>, img: { src: string, w: number, h: number } }
// 值：{
//   sources: {
//     webp: "/assets/pic.hash.webp 800w",
//     avif: "/assets/pic.hash.avif 800w"
//   },
//   img: { src: "/assets/pic.hash.jpg", w: 800, h: 600 }
// }
// 用途：生成 <picture> 元素
```

## 🚀 实际使用示例

### 1. 基础 WebP 转换

```vue
<template>
  <img
    :src="heroImage"
    alt="Hero image"
  />
</template>

<script setup lang="ts">
// 将 PNG 转换为 WebP 格式
import heroImage from '@/assets/images/hero.png?format=webp&quality=85'
</script>
```

### 2. 响应式图像

```vue
<template>
  <img
    :srcset="srcset"
    sizes="(max-width: 600px) 300px, 600px"
    alt="Responsive image"
  />
</template>

<script setup lang="ts">
// 生成多种尺寸的 WebP 图像
import { srcset } from '@/assets/images/banner.png?w=300;600;900&format=webp&quality=85'
</script>
```

### 3. 多格式支持

```vue
<template>
  <picture>
    <source
      :srcset="avifSrcset"
      type="image/avif"
    />
    <source
      :srcset="webpSrcset"
      type="image/webp"
    />
    <img
      :src="fallbackSrc"
      alt="Multi-format image"
    />
  </picture>
</template>

<script setup lang="ts">
// 生成 WebP 和 AVIF 格式
import { srcset as webpSrcset } from '@/assets/images/photo.png?format=webp&quality=85&w=300;600'
import { srcset as avifSrcset } from '@/assets/images/photo.png?format=avif&quality=80&w=300;600'
import fallbackSrc from '@/assets/images/photo.png?w=600'
</script>
```

### 4. 渐进式 WebP 图像

```vue
<template>
  <img
    :src="progressiveImage"
    alt="Progressive WebP"
  />
</template>

<script setup lang="ts">
// 生成渐进式 WebP 图像
import progressiveImage from '@/assets/images/large-image.png?format=webp&quality=85&progressive=true'
</script>
```

### 5. 背景图像优化

```vue
<template>
  <div
    class="hero-bg"
    :style="{ backgroundImage: `url(${bgImage})` }"
  ></div>
</template>

<script setup lang="ts">
// 优化背景图像
import bgImage from '@/assets/images/background.png?format=webp&quality=80&w=1920'
</script>
```

### 6. 高级用法：Picture 元素

```vue
<template>
  <picture>
    <source
      v-for="(srcset, format) in modernSources"
      :key="format"
      :srcset="srcset"
      :type="`image/${format}`"
    />
    <img
      :src="img.src"
      :width="img.w"
      :height="img.h"
      alt="Banner"
    />
  </picture>
</template>

<script setup lang="ts">
import { sources as modernSources, img } from '@/assets/hero.jpg?format=webp;avif&w=800&as=picture'
</script>
```

## 📊 性能影响对比

### 文件大小对比（以 1MB 原始 JPEG 为例）

| 配置                            | 输出大小 | 质量 | 用途       |
| ------------------------------- | -------- | ---- | ---------- |
| `?format=webp&quality=90`       | ~400KB   | 优秀 | 高质量展示 |
| `?format=webp&quality=85`       | ~300KB   | 很好 | 通用场景   |
| `?format=webp&quality=75`       | ~200KB   | 良好 | 缩略图     |
| `?format=avif&quality=85`       | ~250KB   | 很好 | 现代浏览器 |
| `?w=400&format=webp&quality=85` | ~50KB    | 很好 | 小尺寸图片 |

### 构建时间影响

| 操作       | 相对耗时 | 说明           |
| ---------- | -------- | -------------- |
| 格式转换   | 1x       | 基础操作       |
| 尺寸调整   | 1.2x     | 轻微增加       |
| 多尺寸生成 | 2-3x     | 按尺寸数量倍增 |
| 多格式输出 | 2-4x     | 按格式数量倍增 |
| 复杂滤镜   | 1.5-2x   | 模糊、色调等   |

## 💡 最佳实践

### 1. 选择合适的图片

- **重要图片**：使用 imagetools 进行格式转换
- **普通图片**：依赖 vite-plugin-image-optimizer 进行压缩

### 2. 质量设置

- **WebP**: `quality=85` 是视觉无损的最佳平衡点
- **AVIF**: `quality=80` 提供更好的压缩率

### 3. 响应式图像

- 为不同设备生成合适的尺寸
- 使用 `sizes` 属性配合 `srcset`

### 4. 降级方案

- 使用 `<picture>` 标签提供多格式支持
- 确保有 PNG/JPG 格式的降级方案

### 5. 基础配置推荐

```typescript
// vite.config.mts
imagetools({
  defaultDirectives: new URLSearchParams({
    format: 'webp',
    quality: '85',
    progressive: 'true',
  }),
  removeMetadata: true,
})
```

### 6. 按用途分类处理

```typescript
// 英雄图片 - 高质量
import hero from '@/assets/hero.jpg?format=webp&quality=90&w=1920'

// 缩略图 - 小尺寸，低质量
import thumb from '@/assets/photo.jpg?w=200&h=200&fit=cover&quality=75'

// 背景图 - 模糊，低质量
import bgBlur from '@/assets/bg.jpg?blur=10&quality=60&w=1200'

// 响应式图片
import { srcset } from '@/assets/banner.jpg?w=400;800;1200&format=webp&as=srcset'
```

## 🔧 项目配置

### 当前项目配置

```typescript
// vite.config.mts
imagetools({
  // 每张图片默认生效的指令，拼接默认参数
  defaultDirectives: new URLSearchParams({
    progressive: 'true',
  }),
}),
```

### TypeScript 类型声明

项目已在 `env.d.ts` 中配置了完整的 ImageTools 类型声明，支持所有查询参数的类型检查。

```ts
// Imagetools 类型声明
declare module '*.png?format=webp' {
  const src: string
  export default src
}

declare module '*.jpg?format=webp' {
  const src: string
  export default src
}

declare module '*.jpeg?format=webp' {
  const src: string
  export default src
}

declare module '*.png?format=avif' {
  const src: string
  export default src
}

declare module '*.jpg?format=avif' {
  const src: string
  export default src
}

declare module '*.jpeg?format=avif' {
  const src: string
  export default src
}

declare module '*.png?format=webp,avif' {
  const src: string
  export default src
}

declare module '*.jpg?format=webp,avif' {
  const src: string
  export default src
}

declare module '*.jpeg?format=webp,avif' {
  const src: string
  export default src
}

// 带尺寸参数的 WebP
declare module '*.png?w=*&format=webp' {
  const srcset: string
  export { srcset }
}

declare module '*.jpg?w=*&format=webp' {
  const srcset: string
  export { srcset }
}

declare module '*.jpeg?w=*&format=webp' {
  const srcset: string
  export { srcset }
}

// 带尺寸参数的 AVIF
declare module '*.png?w=*&format=avif' {
  const srcset: string
  export { srcset }
}

declare module '*.jpg?w=*&format=avif' {
  const srcset: string
  export { srcset }
}

declare module '*.jpeg?w=*&format=avif' {
  const srcset: string
  export { srcset }
}

// 通用查询参数支持
declare module '*.png?*' {
  const src: string
  export default src
  export const srcset: string
  export const metadata: any
}

declare module '*.jpg?*' {
  const src: string
  export default src
  export const srcset: string
  export const metadata: any
}

declare module '*.jpeg?*' {
  const src: string
  export default src
  export const srcset: string
  export const metadata: any
}

declare module '*.webp?*' {
  const src: string
  export default src
  export const srcset: string
  export const metadata: any
}

declare module '*.avif?*' {
  const src: string
  export default src
  export const srcset: string
  export const metadata: any
}
```

## ⚠️ 注意事项

1. **图片必须通过 import 导入**：只有通过 ES6 import 导入的图片才会被 imagetools 处理
2. **Node.js 版本要求**：最新版本的 imagetools 需要 Node.js 18+，当前项目使用兼容版本
3. **构建时间**：复杂的图片处理会增加构建时间，建议合理使用
4. **文件大小权衡**：过度优化可能导致质量损失，需要在文件大小和质量之间找到平衡

## 🌐 WebP/AVIF 浏览器兼容性

### WebP 支持

- **Chrome**: 23+
- **Firefox**: 65+
- **Safari**: 14+
- **Edge**: 18+

### AVIF 支持

- **Chrome**: 85+
- **Firefox**: 93+
- **Safari**: 16.1+
- **Edge**: 85+

### 移动端支持

- **iOS Safari**: iOS 14+ 支持 WebP
- **Android Chrome**: 原生支持 WebP
- **微信浏览器**: 支持 WebP

### 文件大小对比

- **WebP**: 通常比 PNG 小 25-35%
- **AVIF**: 通常比 WebP 小 20-30%

## 📚 相关链接

- [ImageTools 官方文档](https://github.com/JonasKruckenberg/imagetools)
- [Sharp 文档](https://sharp.pixelplumbing.com/)
- [WebP 格式说明](https://developers.google.com/speed/webp)
- [AVIF 格式说明](https://web.dev/compress-images-avif/)
