---
layout: '../../layouts/MarkdownPost.astro'
title: '[QT5] 常用控件介绍-:'
pubDate: 2024-12-13
description: 'QT是一种GUI开发框架, 它内置有许多各种各样的控件, 接下来就对常用控件介绍一下'
author: '哈米d1ch'
cover:
    url: ''
    square: ''
    alt: 'cover'
tags: ["QT", "", "约字 -- 阅读时间≈分钟"]
theme: 'light'
featured: false
---

`QT`中已经内置了许多的控件: 点击按钮、单选按钮、复选按钮、文本框、下拉框、状态栏...

一个完善的`QT`桌面程序, 是由许多的控件组成的

所以, 控件是非常重要的内容, 常用的控件需要逐一了解

# 了解`QT`内置控件

在前面的文章中, 简单的使用过两个控件`QPushButton`和`QLabel`, 对应点击按钮和文本标签

除此之外, `QT`内置了种类非常丰富的控件(但是颜值并不高), 打开`QT Creator`->`Designer`(双击`QT`项目的`.ui`文件)就能看到`QT`内置的控件:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412161611939.webp)

在了解各种控件类之前, 先了解一下`QWidget`类

`QWidget`是`QT`中控件的通用属性类, 在`QT Designer`中查看:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412161912112.webp)

## `QWidget` **

`QWidget`拥有很多属性:

![|huger](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412161913847.webp)

对于常用的属性, 可以一一进行介绍一下

### `QWidget::enabled`

此属性, 用于设置控件的**可用状态**

添加两个`QPushButton`, 并将`enabled`属性设置为`false`和`true`:

```cpp
#include "widget.h"
#include "ui_widget.h"

#include <QPushButton>

Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    QPushButton* btn1 = new QPushButton(this);
    btn1->setText("按钮1");
    btn1->move(200, 200);
    btn1->setEnabled(false);

    QPushButton* btn2 = new QPushButton(this);
    btn2->setText("按钮2");
    btn2->move(300, 300);
    btn2->setEnabled(true);
}

Widget::~Widget() {
    delete ui;
}
```

执行结果为:

![|huge](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412161920522.webp)

可以看到, 按钮1为灰色, 处于不可选中状态; 按钮2, 则处于正常状态

控件的`enabled`属性是可以随时改变的:

`widget.h`:

```cpp
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QPushButton>

QT_BEGIN_NAMESPACE
namespace Ui {
    class Widget;
}
QT_END_NAMESPACE

class Widget : public QWidget {
    Q_OBJECT

public:
    Widget(QWidget* parent = nullptr);
    ~Widget();

private slots:
    void btn1ClickedHandler();
    void btn2ClickedHandler();

private:
    Ui::Widget* ui;
    QPushButton* btn1;
    QPushButton* btn2;
};
#endif // WIDGET_H
```

`widget.cc`:

```cpp
#include "widget.h"
#include "ui_widget.h"

#include <QPushButton>
#include <QDebug>

Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    btn1 = new QPushButton(this);
    btn1->setText("按钮1");
    btn1->move(200, 200);
    btn1->setEnabled(false);

    btn2 = new QPushButton(this);
    btn2->setText("按钮2");
    btn2->move(300, 300);
    btn2->setEnabled(true);

    connect(btn1, &QPushButton::clicked, this, &Widget::btn1ClickedHandler);
    connect(btn2, &QPushButton::clicked, this, &Widget::btn2ClickedHandler);
}

Widget::~Widget() {
    delete ui;
}

void Widget::btn1ClickedHandler() {
    qDebug() << "按钮1被点击, 槽函数执行";
}

void Widget::btn2ClickedHandler() {
    bool btn1Enabled = btn1->isEnabled();
    if (btn1Enabled)
        btn1->setEnabled(false);
    else
        btn1->setEnabled(true);
}
```

这段代码的执行结果为:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412161934913.gif)

### `QWidget::geometry`

此属性, 用于设置当前控件的位置和尺寸, 即 `(x, y)`和`(width, height)`

`QT`为`geometry`提供了几个接口, 用于**获取当前控件的位置和尺寸**或**设置当前控件的位置和尺寸**

```cpp
#include "widget.h"
#include "ui_widget.h"

#include <QPushButton>
#include <QDebug>
#include <QRect>

Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    btn1 = new QPushButton(this);
    btn1->setText("按钮1");
    btn1->move(200, 200);
    btn1->setEnabled(false);

    btn2 = new QPushButton(this);
    btn2->setText("按钮2");
    btn2->move(300, 300);
    btn2->setEnabled(true);

    QRect btn1Geom = btn1->geometry();
    QRect btn2Geom = btn2->geometry();
    qDebug("按钮1 坐标: (%d, %d), 尺寸: (%d, %d)", btn1Geom.x(), btn1Geom.y(), btn1Geom.width(), btn1Geom.height());
    qDebug("按钮2 坐标: (%d, %d), 尺寸: (%d, %d)", btn2Geom.x(), btn2Geom.y(), btn2Geom.width(), btn2Geom.height());

    connect(btn1, &QPushButton::clicked, this, &Widget::btn1ClickedHandler);
    connect(btn2, &QPushButton::clicked, this, &Widget::btn2ClickedHandler);
}

Widget::~Widget() {
    delete ui;
}

void Widget::btn1ClickedHandler() {
    qDebug() << "按钮1被点击, 槽函数执行";
}

void Widget::btn2ClickedHandler() {
    QRect btn1Geom = btn1->geometry();
    qDebug("按钮1 坐标: (%d, %d), 尺寸: (%d, %d)", btn1Geom.x(), btn1Geom.y(), btn1Geom.width(), btn1Geom.height());

    bool btn1Enabled = btn1->isEnabled();
    if (btn1Enabled) {
        // 可用
        btn1->setEnabled(false);
        btn1Geom.setX(200);
        btn1Geom.setY(200);
        btn1Geom.setWidth(100);
        btn1Geom.setHeight(30);

        btn1->setGeometry(btn1Geom);
        qDebug("重新设置按钮1 坐标: (%d, %d), 尺寸: (%d, %d)", btn1Geom.x(), btn1Geom.y(), btn1Geom.width(), btn1Geom.height());
    }
    else {
        // 不可用
        btn1->setEnabled(true);
        btn1Geom.setX(500);
        btn1Geom.setY(500);
        btn1Geom.setWidth(200);
        btn1Geom.setHeight(30);

        btn1->setGeometry(btn1Geom);
        qDebug("重新设置按钮1 坐标: (%d, %d), 尺寸: (%d, %d)", btn1Geom.x(), btn1Geom.y(), btn1Geom.width(), btn1Geom.height());
    }
}
```

这段代码的执行结果为:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412161952948.gif)

`geometry()`接口, 可以获取控件当前的坐标和尺寸, 返回值是一个`QRect`对象

`QRect`对象包含`x` `y` `width` `height`, 四个属性

> `QRect`对象, 可以通过`qDebug() << QRect`直接打印坐标和尺寸信息

`setGeometry()`接口, 则可以设置控件的坐标和尺寸, 可以通过传入`QRect`对象, 也可以通过传入四个`int`

函数原型:

```cpp
inline const QRect &QWidget::geometry() const;

inline void setGeometry(int x, int y, int w, int h);
void setGeometry(const QRect &);
```

