---
title: vscode 调试技巧
date: 2022-02-23 14:34:15
category: VSCode
tags: [调试，vscode， nodejs]
cover: 
---

Web 开发中，大多数人都会习惯在浏览器端调试代码，而借助 vscode，可以带来不一样的调试体验，试想在代码编写处，直接打一个断点，实时 debugger，体验是不是更顺畅了一点？

以下简单介绍日常 SPA 单页应用、nodejs 应用的调试。

## 调试 SPA 应用

### launch 模式

1、项目中添加调试配置

操作“运行 - 添加配置”，下拉选择 chrome 自动添加即可，根据需要修改参数值

示例如下：

```JSON
{
  // 使用 IntelliSense 了解相关属性。
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Chrome against localhost",
      "type": "pwa-chrome",
      "request": "launch",
      // 默认启动 url
      "url": "http://localhost:3000",
      // 指定终端
      "console": "integratedTerminal",
      "webRoot": "${workspaceFolder}",
      "sourceMaps": true
    }
  ]
}
```

2、启动应用

3、按 `F5` 启动 vscode 调试，即可自动拉起 chrome 访问 url，开始调试。

### attach 模式

attach 模式不会自动拉起 chrome，可以自己选择访问，相比 launch 模式而言，自由度更高一点，并且已安装的浏览器插件可以使用（launch 模式不能）。

1、通过终端启动 chrome，指定 vscode 和 chrome 调试连接的端口（`--remote-debugging-port`），与下方配置中的 `port` 保持一致。

> 命令行启动时，如果 chrome 已经在运行，需要 command + Q 先退出。

```Bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

启动成功如下图：

![](https://secure2.wostatic.cn/static/nk9F5eqwcRjTMKueAJkLj3/image.png)

若未退出已有的 chrome，直接启动会失败，需要先退出再启动，如下图：

![](https://secure2.wostatic.cn/static/kqKnn5saQ96LEWz3b4tXfU/image.png)

2、项目中添加调试配置

操作“运行 - 添加配置”，下拉选择 chrome 自动添加即可，根据需要修改参数值

示例如下：

```JSON
{
  // 使用 IntelliSense 了解相关属性。
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Attach to Chrome",
      "type": "pwa-chrome",
      "request": "attach",
      // vscode 与 chrome 调试的 websocket 端口
      "port": 9222,
      // 指定终端
      "console": "integratedTerminal",
      "webRoot": "${workspaceFolder}",
      "sourceMaps": true
    }
  ]
}
```

3、启动应用

4、按`F5` 启动 vscode 调试

5、在浏览器正常访问应用，即可在 vscode 中开始调试

## 调试 nodejs 应用

1、项目中添加调试配置

操作“运行 - 添加配置”，下拉选择 node 自动添加即可，根据需要修改参数值

示例如下：

```JSON
{
  // 使用 IntelliSense 了解相关属性。
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "pwa-node",
      "request": "launch",
      "name": "create-component-app",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"],
      // 运行当前文件开始调试
      "program": "${file}"
    }
  ]
}
```

2、按`F5` 开始调试

## 备注

1. 如果应用中同时配置了多个调试模式，建议通过点击“运行和调试”，先选择启动模式，再去启动调试。

## 参考文档

1. [这几个 JavaScript 断点调试技能你有必要掌握！](https://mp.weixin.qq.com/s/9gERuxNiJaWYUeie910ALA)
2. [Node.js 深度调试指南](https://juejin.cn/post/6844904199805730823#heading-0)
