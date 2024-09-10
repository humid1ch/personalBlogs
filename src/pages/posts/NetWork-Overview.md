---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux] 网络及其原理简单概述: 协议、协议分层、网络协议栈、局域网内部通信原理、不同局域网通信原理 简单介绍...'
pubDate: 2023-04-23
description: '本篇文章首次接触网络, 将简单介绍一下网络的概念以及网络通信原理的简单理解'
author: '哈米d1ch'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251818539.webp'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251818539.webp'
    alt: 'cover'
tags: ["Linux网络", "TCP-IP", "OSI协议模型", "约10327字 -- 阅读时间≈27分钟"]
theme: 'light'
featured: false
---

Linux的相关介绍, 从本篇文章开始 就要进入网络部分了. 

之前关于Linux介绍的内容都是系统编程方面的. 不过介绍系统, 其实基本都是为介绍网络做铺垫.

本篇文章先介绍一下有关网络要涉及的部分内容, 先了解一下网络. 

---

# Linux 网络概述

从网络的初始形态出现开始, 到现在已经发展了60年左右. 我们都知道, 网络最常用的功能就是实现多计算机之间的通信和数据传输.

那么, 在网络出现之前 计算机之间怎么实现数据传输呢？

## 网络发展

其实在网络没有出现之前, 多个计算机是无法直接数据传输的. 不像现在, 我们可以通过网页、软件……实现数据的传输和通信.

### 独立模式

网络出现之前, 计算机之间都处于一种独立的状态, 没有丝毫的联系.

如果存在三台计算机A、B、C, 各自有不同的功能和数据, 一个业务需要通过这三台计算机处理.

处理业务需要从A到C的顺序进行处理, 那么当A处理完之后, 需要将业务数据传输到B继续处理, B处理完之后, 又需要将业务数据传输到C做处理.

那么, 业务在计算机中处理完成之后, 就需要将业务数据从计算机中拿出来, 再传输到另一台计算机中. 也就是说 需要一个可以存储数据的中间媒介,  就像我们现在使用的U盘、移动硬盘等可以存储数据的硬件. 这样才能完成数据再不同计算机之间的传输.

![|big](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/image-20230424114812249.webp)

如果多台计算机之间总是需要数据的传输, 这样的方式毫无疑问是非常的麻烦的. 

那么就有人提出, 可不可以将计算机连接起来, 方便传输数据呢？

这就出现了网络的最初形态. 

### 网络互联

在介绍最初形态的网络之前, 我们重新了解一下计算机的结构, 进而理解一下`网络`

我们知道, 目前绝大多数计算机的结构都是按照 `冯诺依曼体系` 设计的.

也就是说, 我们计算机一般都可以看作 存储器-输入输出设备-中央处理器 组成的.

而我们明白, 计算机内部的各种硬件之间也是需要数据交互的, 而它们之间的交互其实就是通过 主板中的连接各硬件的线路来交互的.

![|huger](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/image-20230424155848181.webp)

如果将上图看作是一个计算机的结构简略图, 那么, 它们之间的连接线, 就可以看作是主板上连接各设备的线路.

这是一台计算机内部的体系结构. 一台计算机内部就存在许多的设备和连接线路, 并且设备之间可以通过这些线路来进行数据交互. 

那么, 可不可以将两台计算机也通过线路连接起来呢？只要将不同 `计算机之间也通过线路连接起来`, 不就有了不同计算机之间 `数据交互的可能性` 了吗？ 

假如, 计算机中的存在一个叫网卡的输入输出设备, 我们通过 `"一根线"` 将两台计算机中的`网卡设备连接起来`, 让两台计算机可以通过网卡`进行数据交互`. 那么, 这跟线其实就可以看作 `"网线"`. 那么 其实就可以将 这两台通过网线连接起来的计算机`看作`一个 只有两台计算机的 `"网络"`.

既然, 两台计算机通过线路连接起来, 进而可以传输数据, 可以看作 `"网络"`. 

那么, 一台 `计算机内部的设备` 其实都是 `通过不同的线路相连接` 起来的, 并且可以相互传输数据, 那么 也同样可以将 **`计算机内部的结构看作为一个小型的网络结构`** 

那么, 再回过头来看两台计算机相连接, 其实就是将两台计算机其中的`网卡设备连接起来`了, 只不过 这里的线要`比`一个`计算机内部的设备之间相连接的线要长`的多. 

---

根据计算机内部的各设备相连接的情况. 我们可以以类似的方式将多个不同的计算机的网卡设备相连接起来, 以实现多计算机之间数据交互的功能.

![|big](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/image-20230424172530267.webp)

计算机`内部设备之间连接的线`比较`短`, 且比较`密集`.  在实际中, 这些线传输数据 可能相互之将会造成一定的干扰.

而 连接不同计算机的线就相对较`长`, 没有那么密集, 一般情况下不会出现不同的线互相干扰的情况, 或干扰不强. 但是, 长线有长线的问题：

1.  **`可靠性`**, 即 如何能保证数据在长线中传输时, 不会发生数据丢失或损坏等问题呢？如果发生了怎么样弥补呢？
2.  **`效率`**, 即 如何保证数据在长的线中传输的速度要快一些呢？
3.  **`如何寻找到对方`**, 即 这么长的线, 如何找到传输数据的目的方呢？

其实网络知识基本上都是围绕 `线太长会出现的问题` 展开的~

小型的网络就是这样形成的. 

### 局域网LAN 和 广域网WAN

当需要连接的计算机越来越多、整个网络越来越大时, 就诞生出了一些设备：`交换机`、`路由器`等

`交换机`, 通常用于在 `同一个计算机网络中连接多个设备`, 并通过网络协议来实现数据的传输和交换. 因为计算机太多连接会出现一些问题, 交换机可以减少的出现的问题, 比如 安全性、传输效率等问题.

`路由器`, 当不同的计算机网络之间也需要交换数据时, 就需要使用路由器, 将不同的计算机网络之间连接起来. 交换机会根据 `不同网络的网络协议来实现数据包的转发`. 实现网络之间的通信.

当网络之间通过交换机、路由器连接起来形成更大的网络, 局域网就诞生了.

一般来说, 局域网不会太大. 我们上面举的几个计算机直接连接在一起的例子, 也可以称为一个局域网.

更大的网络就是, 广域网WAN等.

## 认识 "协议"

协议是什么？在生活中, 协议其实就是一种约定, 而在网络的世界. 其实协议也是一种约定.

我们可以举一个`生活中的单的例子` 来解释一下什么是协议. 

比如在学生时期 课堂上的小测试, 你和你的朋友可能会设置一些手势来传递问题的答案. 比如, 问方: 先用手势表示题号; 回答方: 用手势表示会不会做, 然后再用手势表示答案选什么……

这些手势肯定不是上课测验时才想出来的, 肯定是你跟朋友`早就约定好的`. 这些约定好的 什么手势表示什么内容, `其实就是一种协议`.

在计算机中同样如此. 计算机内部设备传输数据传输的都是二进制数据, 如果各设备之间不对二进制数据约定好读取、发送的协议, 那么就没有办法正常的发送、获取数据. 

网络同样如此, `在网络上, 不同计算机传输的数据也是以二进制的形式传输的`, 但是 二进制的形式可能有所不同：有些可能用 频率的强弱来表示10, 有些可能使用信号的波峰波谷来表示10. 即 不同的主机之间虽然都是二进制的形式传输信息. 但是由于 **`表示的形式不同, 这些二进制数据就不一定可以被正确、正常的读取到`**.

所以, 在传输数据时, 就需要提前约定好双方的数据格式. 

但是, **`只约定好数据格式, 是不能做好不同主机之间的通信的`**. 

举一个生活大家都听过、玩过的一个游戏的例子: 萝卜蹲

萝卜蹲的游戏规则非常的简单：约定好每个人是什么萝卜, 然后被点到名字的萝卜的要做萝卜蹲, 并点名其他萝卜蹲.

这个 `游戏规则` 其实就制定了一个传递 `数据的格式` , 每个人都要做动作且点其他人的名. 一般来说, 这么简单的游戏规则 大家都可以听得懂.

但是, 如果 `6个人一起玩`, `5个人都是说中文`, 但是有 `一个人说法语、日语或者其他可能听不懂的语言`. `别人怎么玩`？

所以, 要 `实现主机间正常的通信`, 其实 **`除了约定好数据格式之外, 还需要遵守制定好的标准`**.

为什么呢？

1. 计算机生产厂商有很多
2. 计算机操作系统, 也有很多
3. 计算机网络硬件设备, 还是有很多

计算机生产厂商多, 每家厂商的计算机内部实现数据通信的方式就可能不同.

操作系统多, 不同的操作系统内部数据的表现形式就也可能不同.

网络硬件设备多, 不同的设备也可能会存在一些大大小小的差异.

为了确保不同厂商、操作系统、网络硬件的主机之间可以正常的通信, 就需要 **`制定出一个可以让所有的厂商、操作系统、网络硬件同一约定的标准`**.

这个标准, 就可以被称为 **`网络协议`**. 网络协议, 一般包括了数据传输的格式、传输速率、错误检测与纠正等方面的规定. 以方便数据的传输、主机的通信.

## 协议的分层

关于分层, 其实学习C/C++ 和 Linux 的过程中, 我们已经解除了许多分层的概念：

比如, C++ 默认实现了类, 可以实现类内函数, 通过实例化对象来调用类内函数. 但是C语言默认 `不支持在结构体内定义成员函数`

虽然C语言不支持在结构体内部定义成员函数, 不过我们可以通过 函数指针 在函数内部定义指针变量, 来实现通过结构体变量 执行结构体所包含的函数 的功能. 这就是一种软件分层

还有我们在介绍Linux 进程时, 我们介绍了 `进程地址空间` 的概念. 在介绍Linux文件系统时, 我们介绍了 `file_struct` 知道了Linux中一切皆文件. 这两种 实现形式其实都是软件分层的一种实现. 系统是一层, 中间是一层, 还有上层的软件或底层硬件又是一层.

通过这样的对软件的分层有许多的优点:

1. 对软进行分层的同时, 其实也对软件的问题进行了归类.

	 进行分层之后, 其实已经将不同层的问题分开了, 因为 不同层之间基本上就是调用与被调用的关系. 当某个功能出现问题之后, 可以通过定位功能的实现层级来确定问题的出现层级, 可以进一步定位问题.

2. 软件分层其实本质上就是: 软件上的解耦

	 软件分层之后, 层与层之间是调用与被调用之间的关系, 就将不同的功能之间进行了解耦

3. 软件分层 也可以便于我们 进行代码分析

软件是可以分层的, 也存在一些优点. 

而 事实上, 网络的本身的代码就是层状结构的. 那么对应的 `网络协议其实也是分层的`

那么如何理解简单的理解网络协议分层呢?

在生活中, 我们每个人一定都打过电话, 我们就可以通过 `打电话` 这个例子来简单的理解一下 `协议的分层`

我们每个人打电话交流时, 一定都是 `用相同的语言来交流` 的. 如果使用不同的语言 很可能就无法交流.

并且, 我们在打电话时, `打电话的人会认为 自己是与对方直接通信的`.

而 实际上, 所谓的直接通信是并不存在的.

我们打电话交流使用同一种语言 就可以看作 在人的层面使用一种语言协议来进行通信

而 我们知道, 打电话交流时 我们说话声音传输到通讯设备中时其实是一种 `模拟信号`, 但是数据传输需要 `数字信号`.

我们打电话实现交流 其实是 先将模拟信号转化为数字信号, 然后 `数字信号再在通讯设备之间传输` 的.

那么 我们就可以将打电话的过程最简单的看作是两个层级: 

1. 人, 用户层级. 人与人之间通过一定的语言(协议)进行通信.
2. 设备层级. 设备之间通过一定的协议 传输数字信号.

那么, 为什么我们打电话的时候 会认为 自己会与对面的人直接通信的呢?

其实是因为, 我们打电话时 **不会去关注中间的数据** 是以怎么样的形式存在 是以什么样的方式传输的.

同理, 设备之间传输数据时, 也不会在意打电话的人说的是什么语言, 说的语速多多快. 

所以 打电话时 才会看作 `人是与人通信的`. 那么 同理, 其实也可以看作 `设备是与设备通信的.`

以这样的视角来看, 其实可以进一步看作: **`同层协议设备之间直接通信, 可以忽略底层的网络.`**

**`这也就是 网络协议是分层的. 那么, 不同层就需要有自己的协议.`**

分层之后, 不同层级之间进行数据传输只需要 `调用不同层级提供的接口` 就可以了, 不需要关注底层是如何实现的.

那么既然网络协议是分层的, 那么 协议一共多少层呢? 每层的内容又是什么呢?

下面, 就来介绍一下 **`网络协议栈`** 的内容.

## 网络协议栈

### OSI七层模型

OSI（Open System Interconnection，开放系统互连）七层网络模型称为 **开放式系统互联参考模型**, 是一个 **逻辑** 上的定义和规范

OSI 七层模型把网络从逻辑上分为了7层. 每一层都有相关、相对应的物理设备，比如路由器，交换机;

OSI 七层模型是一种 **框架性的设计方法**，其最主要的功能使就是帮助不同类型的主机实现数据传输;

它的最大优点是将服务、接口和协议这三个概念明确地区分开来，概念清楚，理论也比较完整. 通过七个层次化的结构模型使不同的系统不同的网络之间实现可靠的通讯;

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/2023-06-08_22-48.webp)

OSI 七层模型在框架上 逻辑设计上 是十分的优秀的. 

七个层次, 每个层次都有特定的功能和责任, 这种层次结构 `使得网络设计和实现更加模块化和可控`. 不同层级之间更加独立, `方便扩展与更新`. 可以根据不同层级的功能和责任, `快速定位错误 问题`.

并且, OSI 七层模型更是 ISO组织制定的一种描述网络协议和服务的标准. 

但是, 存在一个问题就是, 在实际的软件 网络服务开发中. 如果完全按照OSI七层模型进行开发, 实际上是不方便的.

由于制定标准和实际开发的人员并不是同一批人. 而 实际开发的具体情况 是不可能在定制标准的时候就制定好的.

比如会话层, 会话层主要负责 通信在何时建立, 何时断开的问题. 但是 通信的建立和断开 实际上绝大多数情况都是根据具体的业务来设定的. **底层指定的协议是做不到对上层业务的完全支持的** 

还有表示层和应用层. 由于业务的不同, 表示层的数据转换也不是固定的, 应用层所使用的特定的协议也是多种多样的.

这上三层的协议, 在操作系统的底层中是不能非常统一的实现的. 所以 **操作系统底层中协议一般包含网络层和传输层**.

数据链路层则是由驱动进行支持. 上三层的协议是业务开发时, 具体设计的协议.

所以 在实际的开发中, 并不会使用 OSI七层模型. 而是使用另一种模型: **`TCP/IP 五层(四层)模型`**

### TCP/IP五层(四层)模型

TCP/IP实际上是一组协议的代名词, 它包括许多的协议. 组成了 TCP/IP 协议簇.

TCP/IP 通讯协议采用了 5层的层级结构，每一层都呼叫它的下一层所提供的网络来完成自己的需求.

> - **`物理层:`** `负责光/电信号的传递方式`. 比如现在以太网通用的网线(双绞 线)、早期以太网采用的的同轴电缆(现在主要用于有线电视)、光纤, 现在的wifi无线网使用电磁波等都属于物理层的概念。物理层的能力决定了最大传输速率、传输距离、抗干扰性等. 集线器(Hub)工作在物理层.
> - **`数据链路层:`** `负责设备之间的数据帧的传送和识别`. 例如网卡设备的驱动、帧同步(就是说从网线上检测到什么信号算作新帧的开始)、冲突检测(如果检测到冲突就自动重发)、数据差错校验等工作. 有以太网、令牌环网, 无线LAN等标准. 交换机(Switch)工作在数据链路层.
> - **`网络层:`** `负责地址管理和路由选择`. 例如在IP协议中, 通过IP地址来标识一台主机, 并通过路由表的方式规划出两台主机之间的数据传输的线路(路由). 路由器(Router)工作在网路层.
> - **`传输层: `** `负责两台主机之间的数据传输`. 如传输控制协议 (TCP), 能够确保数据可靠的从源主机发送到目标主机.
> - **`应用层: `** `负责应用程序间沟通`，如简单电子邮件传输（SMTP）、文件传输协议（FTP）、网络远程访问协议（Telnet）等. 我们的网络编程主要就是针对应用

即使TCP/IP网络协议栈是 5层的, 但是也可以对应 OSI七层模型:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/image-20230515224404521.webp)

而一般网络开发的情况下, 物理层考虑的比较少. 因此很多时候也可以称为 **`TCP/IP四层模型`**

一般而言

> - 对于一台 **`主机`**, 它的操作系统内核实现了 **传输层到物理层** 的内容;
> - 对于一台 **`路由器`**, 它实现了从 **网络层到物理层**;
> - 对于一台 **`交换机`**, 它实现了从 **数据链路层到物理层**;
> - 对于 **`集线器`**, 它只实现了 **物理层**;

但是并不绝对. 很多交换机也实现了网络层的转发; 很多路由器也实现了部分传输层的内容(比如端口转发).

### TCP/IP网络协议栈 与 操作系统 的关系 **

上面介绍了网络的简单的一些概念. 但是并没有涉及关于操作系统的内容. 而本篇文章的标题是 Linux 网络概述.

那么, 操作系统与网络协议有什么关系呢? 

首先可以从一张图来表述: 

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/image-20230516223515037.webp)

这张图 表述的内容 其实是 **`网络中不同层级的协议 其实是操作系统整个体系中各个部分中的一个模块`**.

即, 操作系统的内核中 是包含着传输层和网络层的协议的, 驱动程序中 是包含着数据链路层的协议的, 硬件中 是包含着物理层的协议的.

应用层的协议就是开发时 和 用户使用时需要使用到的协议了. 并且, **`传输层到用户层`** , 操作系统是提供有 **`系统调用`** 的进行操作的.

总结一句话就是, **`TCP/IP协议是操作系统中的一个模块, 网络协议栈 实际上是隶属于操作系统的`**

而且, 在从硬件到操作系统对应的层级中, 由于硬件的不同, 驱动的不同, 操作系统内核的不同, 其包含的协议模块可能也是不同的. 但是尽管不同平台协议实现细节是不同的, 却依旧要使用这样的标准将 TCP/IP协议实现起来. 这是为了保证不同平台之间可以因为使用相同的标准进行通信.

那么, 不同的平台之间通信的大概流程是什么样的呢?

首先, 一定是用户先产生网络信息, 需要通过网络发送到另一个平台的用户. 

用户产生网络信息, 通过应用层的协议, 然后穿过整个网络协议栈, 到达当前平台的物理层.

然后再经过网络转发, 被对方的物理层接收到, 然后再提交到对方的数据链路层, 一直向上提交, 最终到达对方的应用层, 然后在被用户接收到.

即, 整个过程大概是这样的, 我们假设 存在两个TCP/IP网络协议标准的平台, Windows发送, Linux接收, 那么对应的流程用图片表述:

![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/image-20230517000923841.webp)

两个主机之间通过网络通信的整个大概流程, 大概就是这样的.

**`用户产生的数据包(网络信息)的流向, 在主机内是 自顶向下 或 自底向上 流动的`**. 为什么呢?

因为, 网络通信数据包的传递是通过硬件实现的. 如果是用户产生的数据, 不能一下子就传输到指定的硬件, 所以数据需要从用户层层传输到硬件上. 所以 发送时数据包的流向就是 **自顶向下**, 接收时数据包的流向就是 **自底向上**

并且, 目前来说, 所有的IO操作都是这样的.

---

数据包传输的整个过程会经过许多不同的层级, 但是在用户看来, 其实就只是 我向你发送信息而已, 即 此用户在与对方用户直接通信

因为用户看不到数据包在下层的传输过程. 不单单用户看不到数据包在其下层的传输, 其实 `每个层级都看不到其下层的传输过程`.

这也就是 **`同层协议都认为自己在和对方直接通信`**

### 重新以计算机的视角看待 网络协议栈

我们已经粗略的介绍了 两个主机之间 从用户到用户, 数据包的传输过程.

那么, 从计算机的视角如何看到网络协议栈呢？

在上面的内容中, 我们提到 协议其实就是一种约定. 人与人之间的约定, 就是一种协议.

约定都是有内容的, 有成本的. 相互约定好的人, 是需要记忆约定的. 

而 计算机之间的约定又如何体现呢？

1.  **`提现在代码逻辑方面`**
2.  **`提现在数据上`**

如何解释呢？

其实可以从生活中的例子来帮助理解.

>  我们在网上买东西的时候, 比如一个键盘、鼠标、显示器等. 并不是像 我们直接在超市买东西 那样, 直接得到我们买的东西.
>
> 而是需要从 商家 经过 快递 再到 买家手里. 这是 在网上买东西时, 一个商品的运输过程.
>
> 而且, 商家也并不会直接将单独的一个商品 给快递公司, 快递公司也不是直接将商品给到买家的.
>
> 商家首先会对商品进行包装. 然后快递公司会对商品进行二次包装, 并且快递公司二次包装的时候 还会添加一些额外的信息: 快递单号、发件信息、收件信息等. 这些信息并不是给买家看的, 而是快递在运输时使用的. 
>
> 商品 从 商家到买家手中, 是经历了一些运输过程的, 并且 为了方便、安全的运输, 商家、快递公司会对商品添加一定的额外信息. 最终到买家手中的才会是一个完整的商品. 
>
> 而, 买家收到的商品并不仅仅是一个目的商品, 还有许多的其他物品或信息. 只不过这些物品和信息一般不需要买家关注.

这个过程, 实际上就以某种角度 反映了不同主机之间通过网络协议栈进行数据通信的过程.

`商家 和 买家, 看做两台主机的用户. 快递公司就可以看做是下面的 网络协议栈.`

- 快递在运输过程中, 会多出快递单等信息. 快递单实际上并不是给用户看的. 而是给快递公司内部 和 快递小哥看的. `快递单上的数据` 就可以看做 是 `快递公司内部和快递小哥之间的协议`.

	这一点, 也反映了 **`实际的 网络通信中, 为了维护被传输的数据 实际是需要在原数据的基础上添加其他数据(协议数据)的`**.

	这是 **`协议在数据上的提现`**. 

	而, 同层协议都认为自己再与对方直接通信. 就好比商家与买家直接交流, 只要快递正常运输, 这个过程双方是不在乎的, 也就不在乎快递单上的内容. 

	那么也就是说, 协议数据实际上是为了同层级通信 而添加的.

	协议数据是为了同层级通信, 那么同层级就需要都能读取协议数据. 那么 **`每一个层级就需要制定自己的协议`**. 进而根据协议来添加同层级可以读取的协议数据. 

- 快递公司和快递小哥 会根据快递单上的数据 来运输商品. 快递是从哪里发货的, 通过什么途径, 送到什么地方等.

	这则反映了 **`实际的 网络通信中, 被传输的数据 会根据协议数据 通过设定、编写的代码逻辑 在不同的主机之间进行传输`**. ~可能感受不到~ 

	这是 **`协议在代码逻辑上的提现`**

网络协议栈, 介绍到这里 其实可以得到三个结论：

1. **`数据包(网络信息) 在主机内是 自顶向下 或 自底向上 流动的`**
1. **`同层协议都认为自己在和对方直接通信, 每一个层级就需要制定自己的协议`**
1. **`实际的 网络通信中, 为了维护被传输的数据 实际是需要在原数据的基础上添加其他数据(协议数据)的`**

根据这三个结论, 我们可以在细化一下 两个主机的通信过程：

1. 当主机一要向主机二传输 "你好" 信息时, 生成的信息数据包 在主机一内部的网络协议栈中 的变化 是这样的:

    ![PC1 |wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308211616981.gif)

    首先, 主机一 产生的数据 在主机一内部 的 `流向是从上到下` 的.

    并且, 传输过程中 不同层会在 接收到的上层数据包中 `添加本层的协议数据`, 这部分数据是拼接在 `原始数据的开头` 部分的

    - 实际上, **`每层添加的这部分协议数据, 被称为 报头`**

        即, 不同层会为接收到的数据包 添加报头

    整个过程 被称为 `对数据进行` **`封装`** `的过程`

2. 主机一 将数据 传输到主机二时, 是先传输到主机二的物理层的

3. 主机二 接收到数据之后, 数据在主机二内部的网络协议栈的变化是这样的:

    ![PC2 |wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308211616263.gif)

    首先, 接收到的数据包 在主机二内部的网络协议栈中的 流向是从下到上的.

    然后 当前层会 `将发送方对应层添加的协议数据(报头)进行分析、解包`, 然后将解包后的数据包 向上层传输

    这个动作, 其实就是 **`同层协议可以看作是与对方直接通信`**. 因为 **`对应层只会针对相应的报头进行解包`**, 那么就可以看作是 `同层协议直接与对方通信`

    最终, 经过层层的解包, 还 原出主机一发送的, 给用户看的数据

## 局域网内部通信原理简单介绍

> **`本系列文章在介绍网络时, 非必要不会涉及物理层内容`**

提问, 如果两台主机在同一个局域网中, 那么这两台主机可以直接通信吗?

答案肯定是可以的. 

> 局域网是 对较小的地理范围内所建立的一组相互连接的计算机和网络设备的集合
>
> 局域网又被称为以太网. 关于以太, 还有一个故事, 感兴趣的话可以去了解一下.

但是局域网内的通信原理是什么呢?

来句一个简单的例子来建立一个最基本的概念:

假设一个班级正在上课, 老师点名提问某个同学回答问题. 点名的这个过程, 就可以看作是局域网内通信的过程.

我们假设, 班级内每个同学和老师都有唯一的名字. 那么老师点到一个同学的名字, 那个同学就一定知道老师实在喊他, 因为在这个班级里 他的名字是唯一的.

老师点到一位同学的名字, 这位同学可以直接听到 做出回答问题. 这个过程中, 不仅被点名的同学可以听到老师的声音, 其他同学也可以听到老师的声音, 但是其他同学不会回应老师, 因为老师没有叫到自己. 

但是还有其他情况: **班级里每个人是可以随意说话的**, 当班里的秩序非常的混乱, 每位同学都在自说自话. 这时候, 被点名的同学很有可能就听不到老师的点名, 进而也就无法回应.

这个例子中的某些现象, 可以映射局域网通信时的一些现象. 也可以映射出局域网通信的原理.

首先, 这个班级内部就可以看作是一个局域网, `每位学生和老师都是局域网内的主机`.

学生和老师的名字, 可以看作局域网内部每个主机的`MAC地址`.

> MAC地址 是 识别网络中物理网络设备的唯一标识符. 所有物理网络设备的MAC地址都是唯一的.
>
> MAC地址由一个48位的二进制数字组成, 一般表现为 12位的十六进制数
>
> 在Win和Linux 都可以通过指令的方式查看 当前主句物理网络设备的MAC地址:
>
> - Win: 
>
>     打开 Windows Powershell 或者 CMD, 执行 `ipconfig /all` 即可查看 当前主机网络的信息, 可以在其中找物理设备的MAC地址:
>
>     ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306181112482.webp)
>
> - Linux:
>
>     打开终端, 执行 `ifconfig`, 即可查看 当前主机的部分网络信息:
>
>     ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306181134260.webp)

每台可以连接到网络的主机都要有一个 `唯一的标识符`, 这个标识符就是主机中 `物理网络设备的MAC物理地址`

>  **`物理网络设备的MAC地址是全球唯一的`**

>  班级里, 老师点名学生回答问题, 其实就是在找学生的 "MAC地址". 找到了, 学生就可以回答问题.

但是, 一个局域网中的任意一台主机, 都是可以在任意时刻发送消息的. `如果, 许多的主机同时发送消息, 那么就可能非常的混乱, 就导致主机之间互相干扰, 进而扰乱通信`. 我们将这种现象称为 **`碰撞`**, 并将可能发生碰撞的一部分主机、设备称为 **`碰撞域`**

>  碰撞, 其实就像班级里许多学生都在说话, 乱糟糟的. 这时候, 班级内的人传达信息, 由于被其他人干扰了其实很有可能不能正确、完整的传到目标人那. 

既然局域网内部有可能发生碰撞, 进而可能导致 通信异常. 那么 也就需要一定的解决方法.

实际上的解决方法 思路很简单, 即: 通信前, 先进行 `碰撞检测`, 检测是否有其他主机已经在通信. 如果有就进行 `碰撞避免`, 即 等其他主机不通信了, 然后再通信

---

![局域网通信的简单原理 |big](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306181217971.webp)

这是局域网通信时, 不同主机之间可能发生的数据流动图, 即 `局域网通信的简单原理图`. 

负责发送和接受数据的主机, 分别需要向以太网中发送数据、从以太网中获取数据. 并且, 发送数据之前, 还要在以太网中检测是否可能发生碰撞. 

也就是说, `以太网中的不同主机是都可以看到以太网的`. 那么, 从系统的角度来看待以太网的话, 其实以太网就是一个 **`临界资源`**

## 不同局域网通信原理简单介绍

通过上面一部分文章, 我们了解到 同一局域网内的不同主机是可以直接通信的. 

那么不同局域网之间该如何通信呢?

![不同局域网通信简单原理 |big](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251011777.webp)

此图, 即为不同局域网通信的简单的原理图. 

当局域网一的某台主机 要向局域网二的某台主机发送信息时. *局域网一主机 对数据的封装过程 和 局域网二主机 对数据的解包过程* 与 *局域网内部通信, 两台主机的封装与解包的过程* 是完全相同的.

 **`不同的是 封装完毕的数据包的转发的过程`**

局域网内部通信时, 数据包可以直接在局域网上转发到其他主机.

而 不同局域网之间通信 不能如此. 不同局域网之间想要进行通信 **`需要通过路由器进行转发`**.

路由器每连接一个局域网, 就可以看作是此局域网内的一台主机.

上图中显示, 路由器连接了两个局域网, 那么也就是说 路由器分别可以和 `局域网一内的主机与局域网二内的主机直接通信`

那么, **`想要局域网一与局域网二通信, 就可以通过将数据包传输给路由器, 再由路由器传输给另一个局域网中的目标主机`**. 

以上图为例, 这一部分的实现细节 可以分为这几步:

1. 局域网一的主机, 将封装完毕的数据包 通过 以太网 直接发送给路由器
2. 路由器 将以太网协议数据 进行解包, 然后 向上传输到路由器自身的IP层
3. 再将数据包向下传输到路由器自身的数据链路层, 添加局域网二数据链路层协议的协议数据 封装数据包
4. 然后 再将封装好的数据包 直接传输到 局域网二的目标主机

进而完成局域网一与局域网二的通信.

局域网内部通信 可以通过MAC地址直接通信, 但是 `不同局域网之间` 的通信不能通过MAC地址进行通信, 而是 **`必须要通过IP地址进行通信`**

为什么? 因为不同主机的网络设备的MAC地址通常是杂乱无章的, 而 同一局域网下的主机的ip地址 通常是在一个子网范围内的.

那么, `路由器就可以通过 ip 判断需要将接收到的数据包 向哪个局域网转发`.

上面例子中, 如果 局域网一主机的ip为 `IPA`, 局域网二主机的ip为 `IPB`, 即 **目标ip为IPB**. 那么, 路由器在接收到数据包之后, 就可以通过目标ip, 将数据包转发到子网范围包括目标ip 的局域网二中. 

---

了解了 不同局域网通过网关(用于连接不同网络的设备)进行通信之后, 其实我们可以发现一个特点, 即 **`从IP层往上(包括IP层), 发送主机和接收主机 看到的数据是一模一样的.`** 

网关设备会对数据链路层的协议报头进行解包、封装, 但不会对更上层的数据进行改动. 

所以 网络, 也被称为 **IP网络**

## 数据包 封装的部分细节

上面的网络通信原理简单介绍时, 举出的例子的主机内部协议都是 指定的, 即:

![ |big](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202306251130568.webp)

但是, 实际上不同主机的不同层级所使用的协议很大可能是不同的. 不同层级的协议也有很多:

1. 应用层: `TFP` \ `HTTP` \ `HTTPS `......
2. 传输层: `TCP` \ `UDP` \ `ICMP` ......
3. 网络层: `IP` \ `ARP` \ `RARP` ......

而且, 接收主机在解包时是需要将数据包向上层传递的. 

那么 **`接收主机如何确定要将有效载荷交付给上层哪一个协议呢?`**

> 有效载荷: 解包后, 需要向上层交付的数据包内容

实际上, 需要的交付信息在解包时就已经获取到了. 既然如此, 其实 **`发送主机在封装报头时, 就需要考虑未来解包时, 将自己的有效载荷交付给上层的哪一个协议`**

并且, **在解包时 确定将自己的有效载荷交付给上层的哪一个协议的过程**, 被称为 **`有效载荷的分用`**

那么, 结论就是:

1. 一般而言, 任何报头属性里面, 一定要存在一些字段, 支持我们进行封装和解包

    即, 报头中 一定要`存在描述 数据包哪些部分是报头, 哪些部分是有效载荷 的字段`

2. 一般而言, 任何报头属性里而, 一定要存在一些字段, 支持我们进行分用

    即, 报头中 一定要`存在描述 有效载荷需要交付给上层哪一个协议 的字段`

这就是, 封装的部分细节

---

本篇文章到此结束, 感谢阅读~