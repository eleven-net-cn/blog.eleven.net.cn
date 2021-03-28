---
title: 「转」H5直播流原理与实践方案
date: 2021-03-28 12:22:05
category: Video
tags: [直播, Video, Live]
---

> 转自 DuPei 同学的文档，这是比较全面的方案调研，转载分享给更多人，待将来需要时细细研究。

## HLS 方案

HLS 全称 HTTP Live Streaming，顾名思义是一个基于 HTTP 的流媒体传输协议。是由苹果公司提出，在苹果家族的整个产品都得到了比较好的支持，后来谷歌在 Chrome 浏览器和移动端浏览器也进行了原生支持，所以目前无论你是在 PC 还是移动端的浏览器基本都原生支持 HLS 协议进行播放视频，算是一个在移动端比较好的跨平台方案，同时微信内嵌的浏览器也都是原生支持的。

它允许用户从不同的备用源以不同的速率下载同样的资源片段。允许流媒体会话适应不同的数据速率。

### 工作原理

它的工作原理是把整个流切分成一片片小的基于 HTTP 的文件来下载。
浏览器使用的是 m3u8 文件，可以简单的认为 m3u8 就是包含多个 ts 文件的播放列表。播放器按顺序逐个播放，全部放完再请求一下 m3u8 文件，获得包含最新 ts 文件的播放列表继续播，周而复始。整个直播过程就是依靠一个不断更新的 m3u8 和一堆小的 ts 文件组成，m3u8 必须动态更新，ts 可以走 CDN。

![](/imgs/video_live_hls.png)

### 流说明

切片流，需要不断去请求新的 ts 文件才能续播。

- 视频的封装格式是 TS。
- 视频的编码格式为 H264,音频编码格式为 MP3、AAC 或者 AC-3。
- 除了 TS 视频文件本身，还定义了用来控制播放的 m3u8 文件（文本文件）

m3u8 文件包含的内容如下：

- EXTM3U
  每一个 m3u8 文件开头必须为这个 tag，用作标示。
- EXT-X-VERSION
  用于标示版本，当前版本为 3，有且只能有一个，默认为 1。
- EXT-X-TARGETDURATION
  每一片的最大长度，部分 iphone 设备此参数不正确会导致无法播放。
- EXT-X-MEDIA-SEQUENCE
  切片的开始序号，且每一片的序号具有唯一性，相邻序号递增以保持流的连续性。
- EXTINF
  切片的实际时长。
- EXT-X-PLAYLIST-TYPE
  类型
- EXT-X-ENDLIST
  结束符号。

### 优点

兼容性很不错，尤其是在移动端，可作为兜底方案。

### 缺点

1. 延时比较大。由于 HLS 协议本身的切片原理，基本延迟都在 10s 以上。
2. 文件碎片，ts 切片较小，会造成海量文件，对存储和缓存都有一定的挑战。

## HTTP FLV 方案

HTTP-FLV 即将流媒体数据封装成 FLV 格式，然后通过 HTTP 协议传输给客户端。

播放的能力基于 Media Source Extensions(MSE)，是一个 W3C 草案，MSE 扩展了 HTML5 的 Video 和 Audio 标签能力，允许开发者通过 JS 来从服务端拉流提供到 HTML5 的 Video 和 Audio 标签进行播放。

[目前 MSE 的兼容情况](https://caniuse.com/?search=Media%20Source%20Extensions)

### 工作原理

MSE 目前支持的视频封装格式是 MP4，支持的视频编码是 H.264 和 MPEG4，支持的音频编码是 AAC 和 MP3。
封装格式的处理

1. 从服务端拉裸流(flv)过来，在前端将 flv 合成 MP4 片段进行播放。
2. 在服务端提前转封装好成 MP4 格式传输，在前端直接通过 MSE 接口来播放。

### 流说明

连续流。

一般方案都是服务端经摄像头推流转成 FLV，然后客户端拉流，拉过来的流解封装为 FLV，然后再转成 MP4 片段，再经由 MSE 播放即可。
这个解封装和转为 mp4 格式的过程，[flv.js](https://github.com/bilibili/flv.js) 帮忙实现了。

### 优点

延时低。

### 缺点

移动端的兼容性不好，iOS Safari 浏览器没有支持，部分低版本的 iOS 微信端也不支持。

## RTMP 方案

Real Time Messaging Protocol（简称 RTMP）是 Macromedia 开发的一套视频直播协议，现在属于 Adobe。

基于 TCP 协议传输，这套方案需要搭建专门的 RTMP 流媒体服务如 Adobe Media Server，并且在浏览器中只能使用 Flash 实现播放器。它的实时性非常好，延迟很小，**但无法支持移动端 WEB 播放是它的硬伤**。

浏览器端，HTML5 video 标签无法播放 RTMP 协议的视频，可以通过 video.js 来实现。https://blog.csdn.net/impingo/article/details/103077380?spm=1001.2014.3001.5502

## 以上三种方案比较

|              | Http flv                        | rtmp    | hls                             |
| ------------ | ------------------------------- | ------- | ------------------------------- |
| 传输协议     | http                            | tcp     | http                            |
| 视频封装格式 | flv                             | Flv tag | ts                              |
| 延时         | 低                              | 低      | 高                              |
| 数据分段     | 连续流                          | 连续流  | 切片文件                        |
| Html5 播放   | 可通过 h5 解封数据包播放 flv.js | 不支持  | 可通过 h5 解封数据包播放 hls.js |

## 其它方案

### WebRTC 方案

WebRTC 是一整套 API，其中一部分供 Web 开发者使用，另外一部分属于要求浏览器厂商实现的接口规范。WebRTC 解决诸如客户端流媒体发送、点对点通信、视频编码等问题。
桌面浏览器对 WebRTC 的支持较好，WebRTC 也很容易和 Native 应用集成。

### WebSocket-FLV

基于 WebSocket 传输 FLV，依赖浏览器支持播放 FLV。
WebSocket 建立在 HTTP 之上，建立 WebSocket 连接前还要先建立 HTTP 连接。

### RTP

基于 UDP，延迟 1 秒，浏览器不支持。

## 清晰度切换支持

[清晰度切换支持](https://cloud.tencent.com/document/product/454/7503#.E5.8A.9F.E8.83.BD.E4.BB.8B.E7.BB.8D)

**播放器本身是没有能力去改变视频清晰度的**，视频源只有一种清晰度，称之为原画，而原画视频的编码格式和封装格式多种，Web 端无法支持播放所有的视频格式，如点播支持以 H.264 为视频编码，MP4 和 FLV 为封装格式的视频。

多清晰度的实现依赖于服务端，进行实时转码，分出多路转码后的视频。
