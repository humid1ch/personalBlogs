---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux] TCP协议介绍(4): 滑动窗口、流量控制、拥塞控制等概念 简单介绍分析...'
pubDate: 2024-01-20
description: 'TCP协议是面向连接的, 面向字节流的, 可靠的 传输层协议...'
author: '哈米d1ch'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202403200107915.webp'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202403200107915.webp'
    alt: 'cover'
tags: ["Linux网络", "传输层", "协议", "TCP", "约字 -- 阅读时间≈分钟"]
theme: 'light'
featured: false
---

# 滑动窗口

为了保障数据传输的可靠性, `TCP`协议实现了确认应答机制以及超时重传机制

为实现确认应答机制, `TCP`协议包头中包含了序号和确认序号

为实现超时重传机制, `TCP`协议同时拥有发送缓冲区与接收缓冲区

那么, 什么是滑动窗口呢?

使用`TCP`协议传输数据时, 发送方是可以发送一批数据的, 只要接收方完整收到了这一批数据, 就需要应答对应的确认序号

当发送方长时间没有接收到应答, 就需要重新将数据发送出去, 以此保证数据传输可靠

要实现超时重传, 就意味着数据发送之后不能丢弃, 需要存储一定的时间, 当接收到确认序号之后才会将已经接收到的数据进行丢弃

那么, 发送出去的数据但是还没有确认被接受的数据存储在什么地方呢? 实际还是在`TCP`的发送缓冲区中存储

`TCP`发送缓冲区可以看作是一块连续的空间, 内容大致可以分为三部分:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202407011441110.webp)

中间 **不用等待确认应答, 可以直接发送的数据** 部分, 即为 **滑动窗口**
