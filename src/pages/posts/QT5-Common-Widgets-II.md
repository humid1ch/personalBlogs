---
layout: '../../layouts/MarkdownPost.astro'
title: '[QT5] 常用控件介绍II: 按钮控件: PushButton、RadoiButton、CheckButton...'
pubDate: 2024-12-17
description: 'QT是一种GUI开发框架, 它内置有许多各种各样的控件, 接下来就对常用控件做一些介绍'
author: '哈米d1ch'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412171051996.webp'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412171051996.webp'
    alt: 'cover'
tags: ["QT", "控件", "约字 -- 阅读时间≈分钟"]
theme: 'light'
featured: false
---

前面一篇文章简单介绍了`QWidget`类中, 一些控件的公共属性

本篇文章的针对**按钮**相关控件展开介绍

# 按钮类控件

`QT`中拥有四种按钮类控件: `QPushButton` `QCheckBox` `QRadioButton` `QToolButton`

这四个控件均继承于`QAbstractButton`类:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231556020.webp)

此类是一个抽象类, 所以无法实例化出对象

它继承于`QWidget`, 提供了一些按钮可能需要的公共的属性: 点击信号、按压信号、图标、文本...

所以, 继承于它的四种按钮控件, 都默认具有这些属性

## `QPushButton`

`QPushButton`已经使用过很多次, 所以不再做过多介绍

只以`QPushButton`为例子, 使用一下`QAbstractButton`的一些属性

### `icon`

通过`icon`属性, 可以**设置按钮的图标样式**

不是按钮样式, 而是按钮图标的样式

> 如果需要使用到图标, 可以在[iconfont-阿里巴巴矢量图标库](https://www.iconfont.cn/)查找并使用

先将需要使用的图标使用`qrc`机制管理起来, 然后在进行使用:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231621557.webp)

然后通过`QT Designer`创建5个`QPushButton`:

![|biger](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231633287.webp)

`QAbstractButton`图标相关的接口:

| 接口                               | 功能                   |
| ---------------------------------- | ---------------------- |
| `void setIcon(const QIcon &icon);` | **设置按钮图标**       |
| `QIcon icon() const;`              | **获取按钮当前的图标** |
| `QSize iconSize() const;`          | **设置按钮图标的大小** |

创建完成之后, 通过代码为下面的四个`PushButton`添加`icon`:

```cpp
#include "widget.h"
#include "ui_widget.h"

#include <QIcon>

Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    // 为按钮设置图标 
    ui->pushBtn_up->setIcon(QIcon(":/up.png"));
    ui->pushBtn_up->setIconSize(QSize(88, 88));

    ui->pushBtn_down->setIcon(QIcon(":/down.png"));
    ui->pushBtn_down->setIconSize(QSize(88, 88));
        
    ui->pushBtn_left->setIcon(QIcon(":/left.png"));
    ui->pushBtn_right->setIcon(QIcon(":/right.png"));
}

Widget::~Widget() {
    delete ui;
}

void Widget::on_pushBtn_up_clicked() {
    QRect nowGeo = ui->pushBtn_ICON->geometry();
    if (nowGeo.y() < 5)
        return;

    ui->pushBtn_ICON->setGeometry(nowGeo.x(), nowGeo.y() - 5, nowGeo.width(), nowGeo.height());
}

void Widget::on_pushBtn_down_clicked() {
    QRect nowGeo = ui->pushBtn_ICON->geometry();
    if (nowGeo.y() > ui->pushBtn_up->geometry().y() - nowGeo.height() - 1)
        return;

    ui->pushBtn_ICON->setGeometry(nowGeo.x(), nowGeo.y() + 5, nowGeo.width(), nowGeo.height());
}

void Widget::on_pushBtn_left_clicked() {
    QRect nowGeo = ui->pushBtn_ICON->geometry();
    if (nowGeo.x() < 5)
        return;

    ui->pushBtn_ICON->setGeometry(nowGeo.x() - 5, nowGeo.y(), nowGeo.width(), nowGeo.height());
}

void Widget::on_pushBtn_right_clicked() {
    QRect nowGeo = ui->pushBtn_ICON->geometry();
    if (nowGeo.x() > this->geometry().width() - nowGeo.width() - 1)
        return;
    ui->pushBtn_ICON->setGeometry(nowGeo.x() + 5, nowGeo.y(), nowGeo.width(), nowGeo.height());
}

```

执行结果为:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231931754.gif)

### `autoRepeat`

从字面上看, `autoRepeat`是自动重复的意思

在按钮上, `autoRepeat`属性表示**点击事件的自动重复**

什么意思呢?

默认情况下, 在按钮上长按鼠标左键, 鼠标会触发`pressed`信号, 而不是`clicked`

`clicked`是按下和抬起两个动作, 长按鼠标左键只是一个按下动作

长按鼠标左键的结果为:

![|biger](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231954153.gif)

`autoRepeat`属性, 是设置鼠标长按时, **是否重复产生点击信号**, 而不是仅有按下

相关的接口有:

| 接口                        | 功能                               |
| --------------------------- | ---------------------------------- |
| `void setAutoRepeat(bool);` | **设置按钮的`autoRepeat`属性**     |
| `bool autoRepeat() const;`  | **获取按钮当前的`autoRepeat`属性** |

将`up`和`down`设置为`autoRepeat(true)`:

```cpp
Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    ui->pushBtn_up->setIcon(QIcon(":/up.png"));
    ui->pushBtn_up->setIconSize(QSize(32, 32));
    ui->pushBtn_up->setAutoRepeat(true);

    ui->pushBtn_down->setIcon(QIcon(":/down.png"));
    ui->pushBtn_down->setIconSize(QSize(32, 32));
    ui->pushBtn_down->setAutoRepeat(true);

    ui->pushBtn_left->setIcon(QIcon(":/left.png"));
    ui->pushBtn_left->setIconSize(QSize(32, 32));

    ui->pushBtn_right->setIcon(QIcon(":/right.png"));
    ui->pushBtn_right->setIconSize(QSize(32, 32));
}
```

![|biger](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231958387.gif)

### `shortCut`

`shortCut`是快捷键属性, 这个属性一定不会陌生

快捷键有关的接口为:

| 接口                                         | 功能                         |
| -------------------------------------------- | ---------------------------- |
| `void setShortcut(const QKeySequence &key);` | **设置按钮按压的快捷键**     |
| `QKeySequence shortcut() const;`             | **获取按钮当前按压的快捷键** |

可以以上面为例子, 设置上下左右的快捷键为`w` `s` `a` `d`:

```cpp
Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    ui->pushBtn_up->setIcon(QIcon(":/up.png"));
    ui->pushBtn_up->setIconSize(QSize(32, 32));
    ui->pushBtn_up->setShortcut(QKeySequence("w"));

    ui->pushBtn_down->setIcon(QIcon(":/down.png"));
    ui->pushBtn_down->setIconSize(QSize(32, 32));
    ui->pushBtn_down->setShortcut(QKeySequence("s"));

    ui->pushBtn_left->setIcon(QIcon(":/left.png"));
    ui->pushBtn_left->setIconSize(QSize(32, 32));
    ui->pushBtn_left->setShortcut(QKeySequence("a"));

    ui->pushBtn_right->setIcon(QIcon(":/right.png"));
    ui->pushBtn_right->setIconSize(QSize(32, 32));
    ui->pushBtn_right->setShortcut(QKeySequence("d"));
}
```

运行结果为:

![|biger](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412231946693.gif)

通过设置的`w` `a` `s` `d`就能点击按钮

并且从运行结果来看, `shortCut`快捷键的`autoRepeat`是默认`true`的

按下快捷键时, 默认重复产生`clicked`信号

---

上面通过`QKeySequence("w")`等, 设置字母区按键

自己手动输入, 终究容易出错

`QT`内置有按键名的枚举, 可以直接在实例化`QKeySequence`对象时使用

![|big](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412232011689.webp)

所以, 快捷键的设置可以改为:

```cpp
Widget::Widget(QWidget* parent)
    : QWidget(parent)
    , ui(new Ui::Widget) {
    ui->setupUi(this);

    ui->pushBtn_up->setIcon(QIcon(":/up.png"));
    ui->pushBtn_up->setIconSize(QSize(32, 32));
    ui->pushBtn_up->setShortcut(QKeySequence(Qt::Key_W));

    ui->pushBtn_down->setIcon(QIcon(":/down.png"));
    ui->pushBtn_down->setIconSize(QSize(32, 32));
    ui->pushBtn_down->setShortcut(QKeySequence(Qt::Key_S));

    ui->pushBtn_left->setIcon(QIcon(":/left.png"));
    ui->pushBtn_left->setIconSize(QSize(32, 32));
    ui->pushBtn_left->setShortcut(QKeySequence(Qt::Key_A));

    ui->pushBtn_right->setIcon(QIcon(":/right.png"));
    ui->pushBtn_right->setIconSize(QSize(32, 32));
    ui->pushBtn_right->setShortcut(QKeySequence(Qt::Key_D));
}
```

运行结果:



> `QT`提供的按键枚举, 不仅有这些, 基本上包括了所有的按键
>
> 不过都在同一个文件中, 可以查看一下

`QT`提供的按键枚举常量, 可以在使用时相加, 实现组合键的效果, 比如:

```cpp
ui->pushBtn_left->setShortcut(QKeySequence(Qt::CTRL + Qt::Key_A));
ui->pushBtn_left->setShortcut(QKeySequence(Qt::ALT + Qt::Key_A));
ui->pushBtn_left->setShortcut(QKeySequence(Qt::SHFIT + Qt::Key_A));
```

