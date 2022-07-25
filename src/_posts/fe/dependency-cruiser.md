---
title: 前端依赖治理
date: 2022-07-22 10:58:02
category:
tags:
cover:
---

## 简介

> Validate and visualise dependencies. With your rules. JavaScript. TypeScript. CoffeeScript. ES6, CommonJS, AMD.

dependency-cruiser，用于前端依赖模块的可视化和依赖的校验，支持使用定制的规则，支持前端常用的 JS、TS 语言以及 ES Module、CommonJS 等模块规范。

GitHub：https://github.com/sverweij/dependency-cruiser

## 模块依赖可视化

推荐作为阅读项目或开源库源码的辅助工具，更易理清项目内部的结构、依赖关系。

官方示例图

![](https://static.eleven.net.cn/images/dependency-cruiser-archi-graph.svg)

官方示例图（React）

![](https://static.eleven.net.cn/images/react-high-level-dependencies.svg)

更多示例：https://github.com/sverweij/dependency-cruiser/blob/develop/doc/real-world-samples.md

### 使用方式

推荐最简单的查看方式，安装 VS Code 插件 ☞ Dependency Cruiser Extension

![](/imgs/dependency_cruiser_vscode_plugin.png)

安装好插件后，在 vscode 中目标项目目录上，点击右键 - “View Dependencies”，即可自动生成相关依赖分析图，示例如下。

### 更多方式

如果有更加具体的需求场景，也可以通过工具生成 svg 依赖关系图片，详细参照官方说明 https://github.com/sverweij/dependency-cruiser#show-stuff-to-your-grandma 。

一般需要做这些事情：

1. 安装 graphviz，用来生成依赖关系图

   ```bash
   brew install graphviz
   ```

2. 在项目根目录下安装 dependency-cruiser

   ```bash
   npm i dependency-cruiser -D
   ```

3. 运行命令，生成 svg 依赖关系图片

   ```bash
   npx depcruise --include-only "^src" —output-type dot src | dot -T svg > dependency-graph.svg
   ```

相关参数：

- `-output-type dot` ：表示输出格式为 dot，意味着使用 Graphviz 来输出

- `dot -T svg > dependency-graph.svg` ：为 Graphviz 的命令行语法，表示输出名为 dependency-graph 的 svg 文件

- `-exclude`：用于过滤掉图上不关心的依赖

- `-include-only`：与 --exclude 相反，只保留范围内的依赖

- `-do-not-follow`：会过滤掉某个依赖的后续依赖

- `-max-depth`：指定依赖树的深度

## 模块依赖校验

借助 dependency-cruiser 对项目的依赖关系做校验，根据指定的规则，或自定义的规则，规避项目中可能产生的问题风险。

在某些复杂场景，或有较多模块使用约定的项目，非常适合使用 dependency-cruiser 去做规则校验，限制不规范或危险的行为。

### 使用方式

1. 在项目中初始化配置文件

   ```bash
   npx -p dependency-cruiser depcruise --init
   ```

   ![](/imgs/dependency_cruiser_init.gif)

   会在根目录下生成 .dependency-cruiser.js 配置文件

2. 在项目中安装依赖

   ```bash
   npm i dependency-cruiser -D
   ```

3. 在 package.json 中添加命令

   ```json
   {
     "scripts": {
       "start": "vite --port 4000",
       "build": "vue-tsc --noEmit && vite build",
       "preview": "vite preview",
     + "lint": "depcruise src --config .dependency-cruiser.cjs"
     },
     +"gitHooks": {
     + "pre-commit": "lint-staged --allow-empty"
     +},
     + "lint-staged": {
     +   "src/**/*.{js?(x),ts?(x)}": [
     +    "yarn lint"
     +  ]
     +}
   }
   ```

   添加命令 lint，随时运行校验。

   推荐加入到 Git Hooks 中，例如：放到 pre-commit 阶段执行校验，未通过禁止提交代码。

   ![](/imgs/dependency_cruiser_lint.png)

   这里为了方便演示，Git Hooks 用的 yorkie，若使用 husky 则更换对应配置。

### 规则文件

来看一下规则文件都有什么？

文件结构

```js
// .dependency-cruiser.cjs
module.exports = {
  forbidden: [ // 被禁止的规则列表
    {...}, // 规则1
    {...}, // 规则2
    {...}, // 规则3
    // ...
  ],
  options: { // 工具层面的若干配置
   // ...
  }
}
```

### 规则定义

```json
{
  // ----- 规则基本配置 -----
  name: 'no-non-package-json', // 规则名称
  severity: 'error', // 严重等级
  comment: "This module depends on an npm package that isn't in the 'dependencies' section of your package.json. " +
        "That's problematic as the package either (1) won't be available on live (2 - worse) will be " +
        'available on live with an non-guaranteed version. Fix it by adding the package to the dependencies ' +
        'in your package.json.', // 规则描述
  // ----- 规则内容 -----
  from: {}, // 不填则表示所有引用 ☞ 哪些模块、目录会被限制
  to: { // 命中哪些规则限制
    dependencyTypes: [ // 命中的规则
      'npm-no-pkg',
      'npm-unknown',
    ],
    pathNot: [], // 排除掉哪些
  },
},
```

from 和 to 描述规则的具体内容，from 表示「依赖方」，to 表示「被依赖方」。

### 常用规则

常用规则基本都已默认内置在生成的配置文件中，根据需要去开启、关闭或修改匹配范围、规则即可。

- 禁止循环引用 no-circular

- 禁止引用不存在的模块（幽灵依赖） not-to-unresolvable

- 检测未被使用的模块 no-orphans

- 禁止生产环境代码使用开发依赖 not-to-dev-dep

- ......

### 自定义规则

借助强大的可配置能力，几乎可以制定出任何想要的依赖限制规则，以下举几个栗子。

1. 指定范围禁止引用某些三方模块

   ```json
   {
     name: 'not-to-import-src-node',
     comment: 'src/node 禁止引用的模块',
     severity: 'error',
     // from: {}, // 不填，所有引用
     from: {
       path: '^src/node',
     },
     to: {
       path: [
         'antd',
         'react-dnd',
         'react-dnd-html5-backend',
         'moment',
       ],
     },
   };
   ```

2. 禁止跨模块引用

   限制 src 下某个大模块，只能引用模块内部依赖或公共模块

   ```json
   {
     // ----- 规则基本配置 -----
     name: 'no-cross-module-import',
     severity: 'error', // 严重等级
     comment: '禁止跨模块引用',
     // ----- 规则内容 -----
     from: {
       path: '^src/pageA',
     },
     to: {
       pathNot: [
         // 只能引入自己或公共的模块
         '^src/pageA',
         '^src/utils',
       ]
     },
   },
   ```

3. 叶子依赖禁止再依赖其它模块

   例如，项目封装了某个模块，完全使用浏览器原生 API，禁止使用其它相关的社区依赖，可以做如下限制。

   ```json
   {
     // ----- 规则基本配置 -----
     name: 'cookies-leaf',
     comment: 'cookies 库不应该有其它依赖',
     severity: 'error',
     // ----- 规则内容 -----
     from: {
       path: '^src/libs/cookies',
     },
     to: {}, // 不能引用任何其它依赖
   },
   ```

### 集成到 NodeJS 脚本

https://github.com/sverweij/dependency-cruiser/blob/develop/doc/api.md

对于一些较为复杂的项目，尤其有诸多规则限制的项目，考虑将其集成到脚本中做默认校验会，将是一个好选择。

```typescript
import { cruise } from "dependency-cruiser";
import type { IReporterOutput, ICruiseOptions } from "dependency-cruiser";

const ARRAY_OF_FILES_AND_DIRS_TO_CRUISE: string[] = ["src"];
const cruiseOptions: ICruiseOptions = {
  includeOnly: "src",
};
try {
  const cruiseResult: IReporterOutput = cruise(
    ARRAY_OF_FILES_AND_DIRS_TO_CRUISE,
    cruiseOptions
  );
  console.dir(cruiseResult.output, { depth: 10 });
} catch (error) {
  console.error(error);
}
```

## 其它

1. 跟 ESLint 有什么区别么？

   A：二者没有直接联系，一般推荐搭配起来一起使用。eslint 负责代码检查，dependency-cruiser 负责依赖检查。

2. 能集成到 eslint 中，作为 eslint plugin 使用么？

   A：暂时不能

   - 相关讨论：https://github.com/sverweij/dependency-cruiser/issues/529

   - 有人做了尝试：https://github.com/sverweij/eslint-plugin-budapestian

3. 能集成到 webpack 等编译工具中，达到编码中自动 watch 反馈的体验么？

   A：暂时不能。主要是性能方面的考虑，不太适合提供这样的能力。

更多问题的官方 FAQ：https://github.com/sverweij/dependency-cruiser/blob/develop/doc/faq.md
