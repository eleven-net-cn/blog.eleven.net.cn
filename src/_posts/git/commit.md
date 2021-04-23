---
title: 规范 git commit
date: 2020-03-25 10:17:53
category: Git
tags: [Git]
---

如何让 `git commit` 提交时更加规范？例如：vue、angular，如下图。规范化地提交记录，会让将来的回溯查找更容易，也让其他人阅读起来更加简便。

![](https://user-gold-cdn.xitu.io/2020/5/12/172081ff77566188?w=996&h=1021&f=png&s=251160)

最近读到一篇不错的文章（[你可能已经忽略的 git commit 规范](https://juejin.im/post/5e0c82a15188253a907111dc)），吸取下养分，顺便做个记录。文章介绍得很清楚，建议阅读原文，不做额外赘述，以下为集成到项目的快速指南。

## 快速指南

使用到的工具 [commitizen](https://github.com/commitizen/cz-cli)、[husky](https://github.com/typicode/husky)、[gitmoji-cli](https://github.com/carloscuesta/gitmoji-cli)，cz-conventional-changelog 是 angular 的 commit message 格式。

所有包不推荐 global 安装，而仅项目本地安装，方便多人开发时，减少其他人的额外操作。

1. 安装工具

    ```bash
    yarn add commitizen cz-conventional-changelog -D
    ```
    
2. 在项目根目录的 package.json 中添加配置

    ```json
    {
      "scripts": {
        "commit": "git-cz"
      },
      "config": {
        "commitizen": {
          "path": "./node_modules/cz-conventional-changelog"
        }
      }
    }
    ```
    
    官方推荐的是 global 安装 commitizen，然后执行 `commitizen init cz-conventional-changelog --yarn --dev --exact` 去自动添加 cz-conventional-changelog，自动在 package.json 中添加 config 配置，不太推荐这种方式。
    
3. 使用

    - `git commit` 仍然是普通的 git 提交模式
    
    - `yarn commit` 会执行交互式 commit 提交，在终端跟着提示一步步输入即可。

4. 限制每一次 `git commit` 都执行交互式提交

    如果想要更暴力一点，限制每一次 `git commit` 都自动执行规范化地提交，可以配置 git 提交的钩子，借助 husky 更方便一点（不用 husky 当然也可以）。
    
    先将 scripts 中配置的 commit 命令删除，不再需要了。

    安装 husky
    
    ```bash
    yarn add husky -D
    ```
    
    在 package.json 中增加配置
    
    ```json
    {
      "husky": {
        "hooks": {
          "prepare-commit-msg": "exec < /dev/tty && git cz --hook || true"
        }
      },
    }
    ```
    
    有些时候可能不太需要所有的 commit 都执行规范化的提交流程，因此，推荐不要这么暴力限制，而是仅在关键性的提交步骤才执行。
    
## 在提交中支持表情符号

如果想要在提交中使用一些表情符号，如下图：
    
![](https://user-gold-cdn.xitu.io/2020/5/12/172086dbf52c646f?w=2016&h=1570&f=png&s=405106)
    
可以借助 [gitmoji-cli](https://github.com/carloscuesta/gitmoji-cli)
    
安装
    
```bash
yarn add gitmoji-cli -D
```
    
使用方法：在提交时按照约定格式输入表情字符即可（左右两边英文冒号夹着字符，例如bug ☞ `:bug:`），提交后会自动被显示，示例：

```bash
git commit -m "fix(src): :bug: 修复列表显示问题"
```
    
如果想要查看所有的表情符号及介绍，可以[去官方文档查阅](https://gitmoji.carloscuesta.me/)。

或者，`npx gitmoji-cli -l` 查看。

如果全局安装 `npm i -g gitmoji-cli`，则执行 `gitmoji -l` 命令查看。
    
    
## 常见的 commit 类型

- feat: 新增feature
- fix: 修复bug
- docs: 仅仅修改了文档，如readme.md
- style: 仅仅是对格式进行修改，如逗号、缩进、空格等。不改变代码逻辑。
- refactor: 代码重构，没有新增功能或修复bug
- perf: 优化相关，如提升性能、用户体验等。
- test: 测试用例，包括单元测试、集成测试。
- chore: 改变构建流程、或者增加依赖库、工具等。
- revert: 版本回滚

## 手写 commit 的推荐写法
如果通过简单的 git commit -m "" 提交，你大概可以这样写：

```bash
git commit -m "feat(player): 播放功能开发完成"
```

引号内即 commit 的 message:

- feat 表明本次提交的类型
- 括号内容是本次代码的影响目录/文件
- 冒号后面是本次提交的简短描述（冒号后面推荐来个空格）  

加点表情符号（当然，你要先安装 gitmoji-cli）：

```bash
git commit -m "feat(player): :rocket: 播放功能开发完成"
```
