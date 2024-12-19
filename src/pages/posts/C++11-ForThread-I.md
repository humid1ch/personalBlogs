---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++] C++标准中的线程: C++标准线程、锁、条件变量、原子操作的使用, '
pubDate: 2024-12-09
description: ''
author: '哈米d1ch'
cover:
    url: ''
    square: ''
    alt: 'cover'
tags: ["C++", "C++11", "多线程", "约字 -- 阅读时间≈分钟"]
theme: 'light'
featured: false
---

`Linux`线程的相关内容, 概念、锁、条件变量、信号量等在前面的文章中已经做了介绍

C++在`C++11`标准中, 新增了线程相关的类等: `std::thread` `std::mutex` `std::condition_variable` `std::atomic`等等

`C++11`标准中的线程, 只是为跨平台实现的, 底层概念上与`Linux`或`Windows`上没有差别

本篇文章, 只是对相关类、接口的使用介绍

# `std::thread`

`std::thread`是`C++11`实现的线程类, 头文件为`<thread>`:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412191946968.webp)

`std::thread`类的成员很少, 涉及到线程的`id`、创建、等待、分离、交换、销毁等

## `thread::id`

`std::thread`创建的线程也有自己的唯一标识, 它不是`Linux`中的`pthread_t`, 而是C++自己实现的标识

C++标准的线程标识类型是`std::thread::id`, 不是基础类型, 而是C++封装的一个类:

