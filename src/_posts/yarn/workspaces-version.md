---
title: yarn workspaces 依赖安装报错
date: 2021-04-07 15:29:52
category: Yarn
tags: [Yarn, Yarn Workspaces]
---

yarn workspaces 结构的项目，在执行命令安装子包依赖时，可能会看到如下错误：

```zsh
Failed to install dependencies in workspace: expected workspace package to exist
```

yarn issues 下的相关问题讨论：https://github.com/yarnpkg/yarn/issues/7807

目前比较简单的处理办法是降级 yarn 到 v1.18.0

```zsh
yarn policies set-version 1.18.0
```

后续还需要等待 yarn 官方来解决此问题。