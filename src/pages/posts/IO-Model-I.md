---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux-IO] 五种IO模型介绍(1): 理解IO、五种IO模型的概念、'
pubDate: 2024-12-03
description: ''
author: '哈米d1ch'
cover:
    url: ''
    square: ''
    alt: 'cover'
tags: ["Linux系统", "Linux网络", "IO", "约字 -- 阅读时间≈分钟"]
theme: 'light'
featured: false
---

# 重新理解`IO`

在`Linux`系统中, 使用文件`IO`相关的系统调用对文件描述符操作时, 比如`read()`、`recv()`或`recvfrom()`, 默认是阻塞模式的

即, **默认打开的文件描述符没有可读取数据时, `read()`、`recv()`或`recvfrom()`会阻塞等待, 直到可以读取到数据时, `read()`和`recv()`才能将数据从内核拷贝到用户空间中**

以最简单的**命名管道通信**为例:

`fifoServer.cc`

```cpp
#include <iostream>
#include <cstring>
#include <cerrno>
#include <sys/wait.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define IPC_PATH "./.fifo" // 命名文件路径

using std::cerr;
using std::cout;
using std::endl;

int main() {
    umask(0);
    if (mkfifo(IPC_PATH, 0666) != 0) {
        cerr << "mkfifo error" << endl;
        return 1;
    }

    int pipeFd = open(IPC_PATH, O_RDONLY);
    if (pipeFd < 0) {
        cerr << "open error" << endl;
        return 2;
    }

    cout << "命名管道文件, 已创建, 已打开" << endl;

    char buffer[1024];
    while (true) {
        cout << "阻塞" << endl;
        ssize_t ret = read(pipeFd, buffer, sizeof(buffer) - 1);
        cout << "阻塞结束" << endl;
        buffer[ret] = 0;

        if (ret == 0) {
            cout << "\n客户端(写入端)退出了, 我也退出吧";
            break;
        }
        else if (ret > 0) {
            cout << "客户端 -> 服务器 # " << buffer << endl;
        }
        else {
            cout << "read error: " << strerror(errno) << endl;
            break;
        }
    }

    close(pipeFd);
    cout << "\n服务端退出……" << endl;
    unlink(IPC_PATH);

    return 0;
}
```

`fifoClient.cc`

```cpp
#include <cstdio>
#include <cstring>
#include <fcntl.h>
#include <iostream>
#include <sys/stat.h>
#include <sys/wait.h>
#include <unistd.h>

#define IPC_PATH "./.fifo" // 命名文件路径

using std::cerr;
using std::cout;
using std::endl;

int main() {
    int pipeFd = open(IPC_PATH, O_WRONLY); // 只写打开命名管道, 不参与创建
    if (pipeFd < 0) {
        cerr << "open fifo error" << endl;
        return 1;
    }

    char line[1024]; // 用于接收命令行的信息
    while (true) {
        printf("请输入消息 $ ");
        fflush(stdout); // printf没有刷新stdout, 所以手动刷新
        memset(line, 0, sizeof(line));
        if (fgets(line, sizeof(line), stdin) != nullptr) {
            // 由于fgets 会接收 回车, 所以将 line的最后一位有效字符设置为 '\0'
            line[strlen(line) - 1] = '\0';
            // 向命名管道写入信息
            write(pipeFd, line, strlen(line));
            
            if (strcmp(line, "quit") == 0)
                break;
        }
        else {
            break;
        }
    }
    close(pipeFd);
    cout << "客户端(写入端)退出啦" << endl;

    return 0;
}
```

这段代码演示了简单的命名管道通信通信, 并在阻塞前后输出了标识:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412031439077.gif)

可以看到, 当客户端未发送数据时, 服务端尝试读取`pipeFd`数据 处于阻塞状态

当客户端发送了数据时, `read()`能够从`pipeFd`中读取数据, 就结束阻塞

`write()`等一系列写操作的系统调用也是一样的, 当管道满了无法继续写入, 就会陷入阻塞, 直到管道不满才会继续写入

从例子中可以重新理解一下`IO`操作:

**一般情况下的`IO`操作, 实际由*阻塞 和 拷贝数据*两个状态组成** (`read()`、`write()`等的本质, 是将数据在内核和用户之间进行拷贝)

阻塞, 就是等待`IO`事件就绪, 拷贝数据, 就是将数据从内核或用户空间拷贝出来

这样最简单的`IO`操作, 大多数情况下**处于阻塞状态的时间占比更长**

此时, 要提高`IO`效率 从思路上看不难, 减少阻塞时间就好了

如何减少阻塞时间, 提高`IO`效率, 就是`IO`模型的作用

# 五种`IO`模型

五种`IO`模型: **阻塞`IO`** **非阻塞`IO`** **信号驱动`IO`** **多路转接`IO`(多路复用)** **异步`IO`**

## `IO`模型概念

### **阻塞`IO`**

阻塞`IO`是最常见的`IO`模型

在上面的命名管道通信例子中, 使用的就是阻塞`IO`. **`Linux`进程所有的文件描述符默认都是阻塞的**

**在内核将数据准备好之前, 系统调用会一直阻塞等待数据, 使整个执行流陷入等待, 直到数据准备好, 拷贝完毕 系统调用再返回, 即为 阻塞`IO`**

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412031637776.webp)

### 非阻塞`IO`

非阻塞`IO`稍微复杂一些

阻塞`IO`, 会等待数据准备就绪 并拷贝完成之后, 再进行返回. 非阻塞`IO`不会如此

**对于非阻塞`IO`, 如果内核还未准备好数据, 系统调用不会阻塞等待, 而是会直接返回`EWOULDBLOCK`或`EAGAIN`错误码, 此时系统调用结束, 执行流可继续执行其他代码**

非阻塞`IO`, 多了一个询问动作, 而不是呆呆地在内核中等待数据

一次系统调用, 就**只询问一次** 内核数据是否就绪. 如果数据没有就绪, 就返回一个错误码, 不再做其他动作; 如果就绪, 就拷贝数据再返回

因为, 非阻塞`IO`操作, 如果数据未准备好, 系统调用就直接返回了

所以, **非阻塞`IO`操作, 往往需要反复执行尝试从文件描述符中拷贝数据**, 即 **轮询操作**, 这其实是一种对`CPU`资源的浪费

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412031710760.webp)

> 阻塞`IO`不会浪费`CPU`资源, 因为阻塞时执行流不占用`CPU`资源

### 信号驱动`IO`

信号驱动`IO`, 从字面上就能猜出个大概

`Linux`进程信号的产生与进程的运行是异步的, 即 进程接收到信号之前, 信号的产生和发送不由进程控制

`Linux`存在一个`SIGIO`信号, 当文件描述符为信号驱动`IO`, 当数据准备就绪时, 操作系统就会给进程发送`SIGIO`信号

因此, 捕捉`SIGIO`信号, 并在此信号处理函数中进行数据拷贝, 就能实现信号驱动`IO`

**对于信号驱动`IO`, 不主动调用 系统调用, 而是捕捉`SIGIO`信号, 并在`SIGIO`信号处理函数中调用 系统调用, 实现`IO`由信号驱动**

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412031902830.webp)

### 多路转接`IO`**



### 异步`IO`

异步`IO`也是结合进程信号实现的

信号驱动`IO`, 是在文件描述符数据就绪之后, 给进程发送`SIGIO`信号, 让进程执行系统调用拷贝数据

而异步`IO`, 是在设置好异步`IO`请求之后, 进程就可以放手了, 直到内核完成数据拷贝, 数据已经拷贝到用户空间之后, 内核会给进程发送信号, 再通过自定义信号处理函数, 进行数据处理

这里, 在数据拷贝完成之后, 内核发送的 进程信号和信号处理函数, 都是在设置异步`IO`请求时设置好的

即, **对于异步`IO`, 需要对指定文件描述符设置异步`IO`请求(读/写、信号、信号处理、数据存储位置等), 然后等待数据和拷贝数据的工作不由进程执行, 全权交给内核, 内核完成数据拷贝之后会向进程发送信号, 进程处理信号 完成异步`IO`**

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202412031949925.webp)

## 相关名词概念

上面概念中, 有几个名词需要理解一下: **阻塞**和**非阻塞** **同步**和**异步**

**阻塞**和**非阻塞**, 一般根据系统调用正常执行的状态判断

**阻塞:** 系统调用在正常执行并获取结果之前, 会将当前执行流挂起, 暂停整个执行流的运行, 直到获取到结果, 才会恢复并返回

**非阻塞:** 系统调用正常执行, 即使不能马上获取结果, 也不会将执行流挂起, 而会立刻返回, 即使没有获取预想结果

`IO`的**同步**和**异步**, 是根据系统调用对执行流的影响判断的

**同步:** `IO`操作的在执行时, 会阻塞执行流的正常运行, 直到`IO`操作完成, 执行流才会继续运行

**异步:** `IO`操作的执行, 不会影响原执行流的正常运行, `IO`操作完成之后, 会通过回调函数或信号的方式通知执行流处理数据

> 这里的同步与异步, 所用场景是`IO`操作
>
> 线程中有**同步**和**互斥**, 与这里的并没有关系
>
> **线程的同步**, 是指一种控制线程以一定顺序执行的策略
>
> **线程的互斥**, 是指通过锁等手段, 控制线程不能同时运行的策略

## 多路转接**
