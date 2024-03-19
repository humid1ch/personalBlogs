---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux] TCP协议介绍(4): 滑动窗口、流量控制、拥塞控制等概念 简单介绍分析...'
pubDate: 2024-01-20
description: 'TCP协议是面向连接的, 面向字节流的, 可靠的 传输层协议...'
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202403200107915.webp'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202403200107915.webp'
    alt: 'cover'
tags: ["Linux网络", "传输层", "协议", "TCP", "约字 -- 阅读时间≈分钟"]
theme: 'light'
featured: false
---

--

`TCP`协议需要保证数据传输的可靠性, 所以`TCP`协议的实现要比`UDP`协议复杂的多

