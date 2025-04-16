---
title: Vite 疑难杂症
date: 2025-04-15 10:52:55
category:
tags:
cover:
---

在 2022 年初投产了第一个 Vite 应用到产线，陆续处理的问题 Case，挑选一些记录下来。

## 应用中需要支持 `require` 语法

应用代码中，通常不再建议使用 `require`，应当都替换为 `import`.

但是，若为迁移旧代码或其它原因，社区有多款插件可以选用。

简单的场景推荐：https://github.com/wangzongming/vite-plugin-require

如果是三方依赖包的产物中包含了 `require()` 代码呢？

这种场景更加复杂一些，在 esbuild 预构建时可能遇到模块导出的值不符合预期，在调研对比多个社区方案后，推荐考虑使用以下插件：

https://github.com/vite-plugin/vite-plugin-commonjs

## lib 模式编译，如何引入 polyfill？

在打包 App 应用时可以依靠 [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) 处理 Polyfill，而 lib 模式编译类库时，不会引入 Polyfill。

推荐使用 Rollup 插件，通过 Babel 在最后阶段再处理 Polyfill。

```typescript
import { defineConfig } from 'vite';
import { getBabelOutputPlugin } from '@rollup/plugin-babel';

export default defineConfig({
  build: {
    rollupOptions: {
      plugins: [
        /**
         * Running Babel on the generated code:
         *  https://github.com/rollup/plugins/blob/master/packages/babel/README.md#running-babel-on-the-generated-code
         *
         * Transforming ES6+ syntax to ES5 is not supported yet, there are two ways to do:
         *  https://github.com/evanw/esbuild/issues/1010#issuecomment-803865232
         * We choose to run Babel on the output files after esbuild.
         *
         * @vitejs/plugin-legacy does not support library mode:
         *  https://github.com/vitejs/vite/issues/1639
         */
        getBabelOutputPlugin({
          allowAllFormats: true,
          presets: [
            [
              '@babel/preset-env',
              {
                useBuiltIns: false,
                exclude: ['transform-typeof-symbol'],
                modules: false,
              },
            ],
          ],
          plugins: [
            [
              '@babel/plugin-transform-runtime',
              {
                corejs: false,
                // version: require('@babel/runtime').version,
              },
            ],
          ].filter(Boolean),
        }),
      ],
    },
  },
});
```

## Vite lib 模式打包时，静态资源会被内联（[官方解释](https://cn.vite.dev/config/build-options.html#build-assetsinlinelimit)），如何处理？

当我们使用 Vite 编译组件、类库时，此问题暴露尤其明显。例如组件中使用的某几张图片，留待使用该组件的应用打包应属最佳，否则，大量的图片被内联在 js 代码中，会严重影响加载速度。

在我们的场景下此问题十分重要，社区内没有可使用的方案，layne 同学编写了以下插件完美解决问题。

https://github.com/laynezh/vite-plugin-lib-assets

- `iife` 资源路径转拼接指定的 `publicUrl`
- `esm` 转换为相对路径引入
- `commonjs` 转换为相对路径引入

```typescript
import { defineConfig } from 'vite';
import assetsPlugin from '@laynezh/vite-plugin-lib-assets';

export default defineConfig({
  plugins: [
    assetsPlugin({
      name: '[name].[contenthash:8].[ext]',
      limit: 1024 * 8,
      publicUrl: undefined,
    }),
  ],
});
```

## commonjs 模块中引用的资源应原样输出

```typescript
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    commonjsOptions: {
      /**
       * https://github.com/rollup/plugins/tree/master/packages/commonjs#requirereturnsdefault
       *
       * require('./images/123.png') => import Img123 from './images/123.png'
       *
       * and './images/123.png' 会被 vite 处理成 base64 格式实现载入
       */
      requireReturnsDefault(id) {
        const [pureId] = id.split('?', 2);
        return commonjsAssetsFilter(pureId);
      },
    },
  },
});
```

## 修改 lib 模式输出的样式文件名

不同于修改 js 文件输出的名称，样式文件需要不一样的处理方式。

```typescript
import defineConfig from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        assetFileNames(assetInfo) {
          // https://github.com/vitejs/vite/issues/4863
          if (!ignoreStyles && assetInfo.name === 'style.css') {
            return `${fileName}.css`;
          }
          return assetInfo.name;
        },
      },
    },
  },
});
```
