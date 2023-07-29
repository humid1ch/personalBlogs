---
layout: '../../layouts/MarkdownPost.astro'
title: '[Linux] HTTP/HTTPS协议'
pubDate: 2023-07-27
description: ''
author: '七月.cc'
cover:
    url: ''
    square: ''
    alt: 'cover'
tags: ["Linux", "网络", "协议", "应用层", "HTTP"]
theme: 'light'
featured: false
---



---

![|cover](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307270935807.png)

---

应用层协议实际是规定应用层在传输数据时需要遵循的一系列规则和标准.

并且, 协议都是程序员制定的的.

上一篇文章中, 为实现一个简单的网络计算器制定了一个简单的协议. 虽然需要实现的功能非常的简单, 但是还是做了非常多的工作.

而网络上需要传输的数据是非常多非常复杂的. 比如一个视频、图片、网页等资源.

如果都需要每个程序员都自己制定自己的协议. 那是非常麻烦的.

所以, 一些其他程序猿所写的非常好用的协议, 就会形成一个应用层特定的协议的标准. 

然后就可以直接供所有有需求的程序猿使用.

比如应用层用的非常多的一些协议: `HTTP` `HTTPS` `FTP` `SMTP` `DNS` `...`

接下来要做的就是, 学习理解优秀的协议的一些使用和实现细节.

# HTTP协议

平时使用浏览器时, 我们会访问一些网站:`CSDN`、`百度`、`Gitee`等

访问这些网站, 是通过网址访问的, 比如: `www.baidu.com`

但是我们访问网站之后, 再从网址栏复制网址 会发现多了一些东西:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307271036499.png)

除了, `www.baidu.com` 前面还多出了`https://`. 我们看到的多出的一部分, 就是协议的部分.

## `url`

平时我们提到的网址, 就是`url`.

一个完整的`url`结构是这样的:

**`http://user:pass@www.example.jp:80/dir/index.html?uid=1#ch1`**

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307271112101.png)

其中最开始的部分就是表示协议, 不过我们当前的大部分网站使用的都是`https`了, 而不是`http`

并且之后, 是需要填写登录信息的. 不过现在已经看不到了. 现在都是以一个单独的网页或窗口的形式 输入账号密码登录, 然后像服务端发送请求, 再转换到`url`中再隐藏起来:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307271125732.png)

之后就是`域名`和`端口号`. 其中 使用域名访问网站时 是会被转换成对应的IP的, 也必须如此. 后面就是端口.

`IP:Port`可以确定网络中的一个服务器.

但是, 在我们平常访问网站时 **浏览器中显示的时候端口号是被隐藏的**. 但是真正访问网站的时候, **端口号是必须要传入的**.

那么也就是说, 在使用确定的协议时 在显示上 端口号是缺省的. 

那么, 通过浏览器访问指定的网站的时候, **浏览器必须自动为其添加端口号**

这就要介绍另外一个问题了, 浏览器如何知道端口号呢?

实际上, 一些众所周知的协议的服务, 端口号都是强绑定的. 

比如: `HTTP`是`80`, `HTTPS`是`443`, `ssh`服务则是`22`...这些可以说是约定成俗的. 操作系统会将系统中的端口号预留出来不让其他服务使用.

就好比生活中的电话: `110`是警察, `119`就是火警, `120`就是急救...

指明了协议以及域名和端口号, 就可以访问网站了.

### `http`和`https`的作用

我们在网络上查看、阅读的各种内容, 都是以网页的形式展现出来的. 主要都是`html`文件

比如这样:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281056882.png)

执行`wget www.baidu.com`就可以直接获取到一个`index.html`文件. 文件的内容:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281056768.png)

这就是百度首页面的`html`文件, 就是这个`html`文件, 呈现出了百度的首页.

这样看来, `http`和`https`好像是用来获取网页资源的. 或者说, 是用来传输文件资源的.

大概的流程就是 本地使用`http`或`https`协议向服务器发送 获取资源请求, 服务器将资源传输回来, 本地再接收就可以了

除了网页资源, 我们在网上查看的视频、图片都是文件, 都可以通过`http`和`https`协议传输. 这也就是为什么, `http`和`https`被称为 **超文本传输协议**, 不过`https`是更加安全的

即, `http`协议是向特定的服务器申请某种文件资源, 并获取到本地 然后进行展示或使用的传输协议

一般来说通过协议所申请的 **文件都在网络服务器(软件) 所在的服务器中存储着**, 如果没有在服务器中存储, 那就无法获取资源.

不过, 一般来说服务器中的文件是非常的多的. 此时, `url`中表示路径的部分, 就派上用场了.

即, `url`中 紧挨着域名以及端口 表示文件路径的部分, 就表示这此次所申请的文件资源在服务器中的路径. 这里路径的 第一个目录不是服务器的根目录, 而是设置的`web根目录`.

而服务器大多都是`Linux`系统, 所以这也是为什么`url`中表示路径的部分使用的是`/`作为分隔符, 因为Linux中路径分隔符就是`/`:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281527070.png)

又比如在CSDN中的一篇文章:

![|inline](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281530736.png)

就是 在CSDN的服务器中的某个用户名目录下的层层目录的中的某个编好号文件.

通过浏览器, 使用`https`协议向服务器中获取某个文件, 获取到了就在页面中展示出来. 

如果没有获取到, 一般会收到另一个文件内容:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281534665.png)

> 既然获取的是文件资源, C/C++又提供了文件打开读取等功能.
>
> 那么获取文件资源的过程其实就是, 接收到请求之后 根据提供的文件路径文件名找到文件, 然后打开.
>
> 打开之后, 读取文件内容, 再将文件内容响应回客户端就可以了

理解了`http`和`https`协议的作用以及, `url`的结构 再结合`url`的全称, 一下子就可以理解, `url`是什么.

`url: Uniform Resource Locator, 统一资源定位符`

### `urlencode`和`urldecode`

在`url`的内容中 像`/` `?` `#`这样的字符, 已经有一种特殊的作用了. 所以这些字符不能随意的出现.

那么, 如果`url`某个参数中带有特殊的字符, 就需要对特殊的字符进行编码 和 解码. 除了特殊的字符, 还有文字等.

`url`中 针对需要进行编码的符号的 编码规则是: 

1. 针对ASCII码表中的符号, 可以直接转换成16进制, 然后从右到左，取4位(不足4位直接处理)，每2位当作1位，前面加上`%`，编码成`%XY`格式
2. 针对非ASCII码表中的符号、文字等, 先 其它规则进行编码 再对其他规则的编码16进制结果的每字节数据前加上`%`. 编码成多个`%XY`的格式

比如:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281700316.png)

百度搜索`C++`, 在`url`中就显示为`C%2B%2B` `2B`就是`+`的16进制形式

`https://www.baidu.com/s?wd=C%2B%2B&rsv_spt=1&rsv_iqid=0xfdc1da9500081925&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&rqlang=cn&tn=baiduhome_pg&rsv_enter=1&rsv_dl=tb&oq=%2526lt%253B%252B%252B&rsv_btype=t&inputT=1&rsv_t=b33aMahumMWox0zFsNrY2he0Sn8D%2BQ4jTCh9Kdwti9jiIQq4qTDXa%2F09UiGMOLpg%2Bgds&rsv_pq=8dec283100024fa3&rsv_sug3=30&rsv_sug1=23&rsv_sug7=100&rsv_sug2=0&rsv_sug4=340`

其中, `wd=C%2B%2B`就表示搜索的`C++`

或是:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307281700927.png)

百度搜索`博客`, 在`url`中好像还是显示`博客`. 不过当复制出来:

`https://www.baidu.com/s?wd=%E5%8D%9A%E5%AE%A2&rsv_spt=1&rsv_iqid=0xfdc1da9500081925&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&rqlang=cn&tn=baiduhome_pg&rsv_enter=1&rsv_dl=tb&oq=%25E5%258D%259A%25E5%25AE%25A2&rsv_btype=t&inputT=1&rsv_t=48ffg6adF8KNYYP%2FQwg32HXkloqBfmRVDmN1V%2FLz0OMTPukALMBn7Iysz215bHNcwDNC&rsv_pq=e981c6280006c3e4&rsv_sug3=50&rsv_sug1=36&rsv_sug7=100&rsv_sug2=0&rsv_sug4=407&rsv_sug=1`

就会发现, `wd=%E5%8D%9A%E5%AE%A2`. 博客就通过其他的方式编码成了`%XY`的格式

## `http`协议请求格式

`http`协议的请求是 **字符串**, 由 **4部分组成, 每一部分由单行或多行组成. 每行以`\r\n`区分**

`http request`:

1. 第一行自成一部分: 

    **请求行**, 内容是 **请求方法 `url` `http`协议版本**

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307290959492.png)

    这里的`url`可以只是完整`url`中资源的路径, 也可以是一个完整的`url`

2. 第二部分由多行组成:

    **请求报头**, 内容是请求的各种属性. 每行结构为:`key: value`. `:`后必须有一个空格

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291017682.png)

3. 第三部分是单独一行的`\r\n`

    用来表示报头部分读取完毕:

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291018613.png)

4. 第四部分则是需要请求的资源的有效载荷, 也是请求资源的正文部分

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291032951.png)

将这四部分组合起来, 就是一个完整的`http requst`:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291033747.png)

按照协议填充字符串之后, 就可以向服务器发送 进行资源请求了

## `http`协议响应格式

与请求格式相同, `http`协议的响应格式也是 **字符串**. 同样是由 **4部分组成, 每一部分由单行或多行组成. 每行以`\r\n`区分**

`http request`:

1. 第一行自成一部分: 

    **响应行**, 内容是 **`http`协议版本 状态码 状态码描述**

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291037391.png)

    状态码即为, 请求的状态. 状态码描述即为, 对状态码的解释

2. 第二部分同样由多行组成:

    **响应报头**, 内容是响应正文的各种属性. 每行结构为:`key: value`. `:`后必须有一个空格

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291038451.png)

3. 第三部分是单独一行的`\r\n`

    用来表示报头部分读取完毕:

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291018613.png)

4. 第四部分则是需要响应回客户端的资源的有效载荷, 也是资源的正文部分

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291039594.png)

将这四部分组合起来, 就是一个完整的`http requst`:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291040189.png)

按照协议填充字符串之后, 就可以响应回客户端了

## 查看`http`协议格式

`http`协议的格式如上述所介绍的内容.

不过也可以 更加直观 具体的看一下请求和响应的内容:

1. 使用这些代码, 可以看到请求的内容:

    **`logMessage`:**

    ```cpp
    #pragma once
    
    #include <cassert>
    #include <cerrno>
    #include <cstdarg>
    #include <cstdio>
    #include <cstdlib>
    #include <cstring>
    #include <ctime>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <sys/types.h>
    #include <unistd.h>
    
    // 宏定义 四个日志等级
    #define DEBUG 0
    #define NOTICE 1
    #define WARINING 2
    #define FATAL 3
    
    const char* log_level[] = {"DEBUG", "NOTICE", "WARINING", "FATAL"};
    
    // 日志消息打印接口
    void logMessage(int level, const char* format, ...) {
        // 通过可变参数实现, 传入日志等级, 日志内容格式, 日志内容相关参数
    
        // 确保日志等级正确
        assert(level >= DEBUG);
        assert(level <= FATAL);
    
        // 获取当前用户名
        char* name = getenv("USER");
    
        // 简单的定义log缓冲区
        char logInfo[1024];
    
        // 定义一个指向可变参数列表的指针
        va_list ap;
        // 将 ap 指向可变参数列表中的第一个参数, 即 format 之后的第一个参数
        va_start(ap, format);
    
        // 此函数 会通过 ap 遍历可变参数列表, 然后根据 format 字符串指定的格式,
        // 将ap当前指向的参数以字符串的形式 写入到logInfo缓冲区中
        vsnprintf(logInfo, sizeof(logInfo) - 1, format, ap);
    
        // ap 使用完之后, 再将 ap置空
        va_end(ap); // ap = NULL
    
        // 通过判断日志等级, 来选择是标准输出流还是标准错误流
        FILE* out = (level == FATAL) ? stderr : stdout;
    
        // 获取本地时间
        time_t tm = time(nullptr);
        struct tm* localTm = localtime(&tm);
        char* localTmStr = asctime(localTm);
        char* nC = strstr(localTmStr, "\n");
        if (nC) {
            *nC = '\0';
        }
        fprintf(out, "%s | %s | %s | %s\n", log_level[level], localTmStr,
                name == nullptr ? "unknow" : name, logInfo);
    
        // 将C缓冲区的内容 刷入系统
        fflush(out);
        // 将系统缓冲区的内容 刷入文件
        fsync(fileno(out));
    }
    ```

    **`tcpServer.hpp`:**

    ```cpp
    #pragma once
    
    #include <iostream>
    #include <string>
    #include <cstdlib>
    #include <cstring>
    #include <unistd.h>
    #include <signal.h>
    #include <pthread.h>
    #include <sys/wait.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    
    #include "logMessage.hpp"
    
    #define SOCKET_ERR 1
    #define BIND_ERR 2
    #define LISTEN_ERR 3
    #define USE_ERR 4
    #define CONNECT_ERR 5
    #define FORK_ERR 6
    #define WAIT_ERR 7
    
    void handlerHttpRequest(int sock) {
        char buffer[1024];
        ssize_t s = read(sock, buffer, sizeof buffer - 1);
        if (s > 0) {
            std::cout << buffer << std::endl;
        }
    }
    
    class tcpServer {
    public:
        tcpServer(uint16_t port, const std::string& ip = "")
            : _port(port)
            , _ip(ip)
            , _listenSock(-1) {}
    
        ~tcpServer() {
            if (_listenSock >= 0)
                close(_listenSock);
        }
    
        void init() {
            // 先创建套接字文件描述符
            _listenSock = socket(AF_INET, SOCK_STREAM, 0);
    
            if (_listenSock < 0) {
                logMessage(FATAL, "socket() faild:: %s : %d", strerror(errno), _listenSock);
                exit(SOCKET_ERR); // 创建套接字失败 以 SOCKET_ERR 退出
            }
            logMessage(DEBUG, "socket create success: %d", _listenSock);
    
            struct sockaddr_in local;
            std::memset(&local, 0, sizeof(local));
    
            // 填充网络信息
            local.sin_family = AF_INET;
            local.sin_port = htons(_port);
            _ip.empty() ? (local.sin_addr.s_addr = htonl(INADDR_ANY))
                        : (inet_aton(_ip.c_str(), &local.sin_addr));
    
            // 绑定网络信息到主机
            if (bind(_listenSock, (const struct sockaddr*)&local, sizeof(local)) == -1) {
                // 绑定失败
                logMessage(FATAL, "bind() faild:: %s : %d", strerror(errno), _listenSock);
                exit(BIND_ERR);
            }
            logMessage(DEBUG, "socket bind success : %d", _listenSock);
            // 监听是否有其他主机发来连接请求, 需要用到接口 listen()
            if (listen(_listenSock, 5) == -1) {
                logMessage(FATAL, "listen() faild:: %s : %d", strerror(errno), _listenSock);
                exit(LISTEN_ERR);
            }
            logMessage(DEBUG, "listen success : %d", _listenSock);
        }
    
        // 服务器初始化完成之后, 就可以启动了
        void loop() {
            while (true) {
                struct sockaddr_in peer;          // 输出型参数 接受所连接主机客户端网络信息
                socklen_t peerLen = sizeof(peer); // 输入输出型参数
    
                // 使用 accept() 接口, 接受来自其他网络客户端的连接
                int serviceSock = accept(_listenSock, (struct sockaddr*)&peer, &peerLen);
                if (serviceSock == -1) {
                    logMessage(WARINING, "accept() faild:: %s : %d", strerror(errno), serviceSock);
                    continue;
                }
                // 连接成功之后, 就可以获取到连接客户端的网络信息了:
                uint16_t peerPort = ntohs(peer.sin_port);
                std::string peerIP = inet_ntoa(peer.sin_addr);
                logMessage(DEBUG, "accept success: [%s: %d] | %d ", peerIP.c_str(), peerPort, serviceSock);
    
                pid_t id = fork();
                if (id == 0) {
                    close(_listenSock);
    
                    if (fork() > 0)
                        exit(0);
    
                    handlerHttpRequest(serviceSock);
                    exit(0);
                }
                waitpid(id, nullptr, 0);
    
                close(serviceSock);
            }
        }
    
    private:
        uint16_t _port; // 端口号
        std::string _ip;
        int _listenSock; // 服务器套接字文件描述符
    };
    ```

    **`tcpServer.cc`:**

    ```cpp
    #include "tcpServer.hpp"
    
    void Usage(std::string proc) {
        std::cerr << "Usage:: \n\t" << proc << " port ip" << std::endl;
        std::cerr << "example:: \n\t" << proc << " 8080 127.0.0.1" << std::endl;
    }
    
    int main(int argc, char* argv[]) {
        if (argc != 3 && argc != 2) {
            Usage(argv[0]);
            exit(USE_ERR);
        }
        uint16_t port = atoi(argv[1]);
        std::string ip;
        if (argc == 3) {
            ip = argv[2];
        }
    
        tcpServer svr(port, ip);
    
        svr.init();
        svr.loop();
    
        return 0;
    }
    ```

    编译`tcpServer.cc`, 并运行可执行程序之后.

    在浏览器输入IP地址 和 端口号, 就可以看到服务器进程接收到了请求, 并打印了出来:

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291737446.png)

    其格式为:

    1. 第一行: 

        **请求方法:`GET`、`url`:`/`、`http`协议版本:`HTTP/1.1`**

    2. 之后, 则为请求报头相关内容

    3. 最后添加了一个`\r\n`空行

2. 还可以使用`telnet`连接到服务器之后, 向服务器发送请求, 然后可以看到 响应的内容:

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291749682.gif)

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291749607.png)

    这里相应的有效载荷其实就是百度首页的`html`文件内容

    ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202307291754534.png)

## 给服务器添加`http`响应
