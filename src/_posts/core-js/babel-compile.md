---
title: Babel 编译误伤 core-js？
date: 2021-03-26 19:46:28
category: Babel
tags: [Babel, Webpack]
cover: /img/abc/3741616737466_.pic_hd.jpg
---

一个不太常见，但很重要的 core-js bug 讨论：https://github.com/zloirock/core-js/issues/514

## 问题现象

core-js 是 es3 的代码，应当是不需要 babel 编译的，但是，因为某种原因（暂不清楚），babel 会对 core-js 进行处理，并且会影响其中的代码，进而会发现 Symbol 在 Android 6.0 等低版本浏览器的兼容性出问题。

在移动端页面，小伙伴实际遭遇到了这种现象，依赖的第三方包使用了 Symbol 语法，即使引入全量的 core-js，依然无法解决兼容性报错问题，最终小伙伴追踪到了上方的 github issues。

## 解决办法

暂时需要手动将 core-js 添加到 `babel-loader` 的 `exclude` 配置中，指定 babel 不要去处理它。

```js
/**
 * Babel 编译时，会处理 core-js（未来可能会被修复），
 * 导致 polyfill 内部代码发生了变化，产生一些微小的影响，如 Symbol 问题。
 * 暂时我们手动声明略过。
 * https://github.com/zloirock/core-js/issues/514
 * https://github.com/rails/webpacker/pull/2031
 */
exclude: [
  /node_modules[\\/]core-js/,
],
```
