---
title: Charles 抓包/代理 websockt 请求
date: 2021-05-29 19:13:25
category: Charles
tags: websocket
cover:
---

Charles 中如何抓包、代理 websockt 请求？过程有些复杂、坑点颇多，这里提供一份可用的方案。

请先确保你的 charles 能够抓到浏览器的请求。

通常你可能需要注意以下几类问题：

- 确保已勾选 charles 的 `Proxy - macOS Proxy` 选项
- 浏览器推荐设置为“系统代理”，如果使用了某些浏览器的代理插件，请记得切换
- 是否已安装好 https 证书？
- 如果有使用某些 VPN 代理工具，可能会存在代理冲突。如果遇到了抓不到请求的问题，推荐先关掉 VPN 工具，随后重启 charles（重启比较重要，它会重置系统的网络代理配置）

如果还有问题，推荐阅读这篇使用指南 ☞ [传送门](https://blog.csdn.net/mxw2552261/article/details/78645118)。

保证能正常抓到请求，再继续向下去看怎么抓取 websockt，当前使用的 charles 版本是 v4.6.1。

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
