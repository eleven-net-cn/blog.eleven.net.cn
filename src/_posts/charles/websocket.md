---
title: Charles 抓包/代理 websockt 请求
date: 2021-05-29 19:13:25
category: Charles
tags: websocket
cover:
---

Charles 中想要代理 websockt 请求，过程有些复杂、坑点颇多。好不容易调通，记录一波。

## charles 需要开启若干设置

1、打开 Proxy - Proxy Settings，勾选 Enable SOCKS proxy，默认配置不必修改，如下图：

![](/imgs/charles_01.png)

2、在 Proxy - Proxy Settings - macos 面板中，确保勾选 Use SOCKS proxy，如下图：

![](/imgs/charles_02.png)

## Mac 的网络设置中勾选若干项

确保勾选了 SOCKS 选项，如下图：

![](/imgs/mac_01.png)

![](/imgs/mac_02.png)

SOCKS 代理的端口 8889 是 charles 代理 websockt 的默认端口，一般情况下不要去修改它。

以上工作都完成，那么你应该可以愉快地抓取、代理 websockt 请求了。
