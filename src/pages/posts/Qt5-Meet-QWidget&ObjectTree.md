---
layout: '../../layouts/MarkdownPost.astro'
title: '[QT5] 遇见QT5, 遇见GUI, 初识对象树'
pubDate: 2024-8-19
description: '简单介绍一下QT5中的一些基本的特性, Form是什么? QWidget有什么用? 为什么有QString不用std::string? 什么是对象树?'
author: '哈米d1ch'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408192056270.webp'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408192056270.webp'
    alt: 'cover'
tags: ["QT", "Windows", "GUI", "约1770字 -- 阅读时间≈5分钟"]
theme: 'light'
featured: false
---

# `QWidget`默认项目结构

使用`QT Creator`创建一个`QWidget`的默认项目之后, 可以看到整个项目的结构

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408200830383.webp)

首先从`main.cc`开始

> 博主在`QT Creator`中修改了C++源文件的后缀, 默认应该是`.cpp`

## `main.cc`

自动生成的`main`函数所在源文件:

```cpp
#include "widget.h"

#include <QApplication>

int main(int argc, char* argv[]) {
    QApplication a(argc, argv);
    Widget w;
    w.show();
    
    return a.exec();
}
```

先看`main()`函数

`main()`函数内, 首先创建了一个`QApplication`对象, 构造的参数是`argc`和`argv`, 即程序运行时传入的选项数及选项