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

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408200830383.webp)

> 博主在`QT Creator`中修改了C++源文件的后缀, 默认应该是`.cpp`

## `QWidget` 

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

`main()`函数内

1. 首先创建了一个`QApplication`对象, 构造函数的参数是`argc`和`argv`, 即程序运行时传入的选项数及选项
2. 定义了一个`Widget`对象`w`, 这个类是用户创建项目时自定义命名的类, 选择`QWidget`为基类之后 默认命名就是`Widget`
3. 通过`w`调用`show()`成员函数
4. `return a.exec()`

重点看`Widget`类, 打开`widget.h`:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408221555667.webp)

首先, 因为`Widget`是`QWidget`的派生类, 所以需要先`#include <QWidget>`

然后使用两个宏, 在合适的`namespace`内声明了`Widget`类

```cpp
QT_BEGIN_NAMESPACE
namespace Ui {
	class Widget;
}
QT_END_NAMESPACE
```

`QT_BEGIN_NAMESPACE` `QT_END_NAMESPACE` 是`QT`官方库定义的两个宏, 其实际的内容:

1. **`#define QT_BEGIN_NAMESPACE namespace QT_NAMESPACE {`**
2. **`#define QT_END_NAMESPACE }`**