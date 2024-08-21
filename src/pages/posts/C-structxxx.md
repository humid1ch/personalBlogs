---
layout: '../../layouts/MarkdownPost.astro'
title: 'C结构体...'
pubDate: 2023-08-07
description: ''
author: '哈米d1ch'
cover:
    url: ''
    square: ''
    alt: 'cover'
tags: ["C语言", "结构体", "约1890字 -- 阅读时间≈5分钟"]
theme: 'light'
featured: false
---



# 结构体

## 结构体类型的定义

结构体是一种自定义类型

可以将多个不同类型的数据集合在一起形成一个结构体类型

一个结构体类型的定义方式是:

```cpp
// struct(关键字) name(类型名)
struct name {
    // 成员变量
    // type数据类型 var变量名
    type1 variable1;
    type2 variable2;
};
```

比如:

```cpp
struct student {
    int _age;
    char _name[20];
};
```

`struct student`是一个结构体类型, `student`是 **类型名**, `_age`和`_name`是结构体的 **成员变量**

C语言中, 在定义结构体类型时, **成员变量是不能给初始值的**, 只能在结构体内部声明成员变量

---

在了解了`typedef`之后, 结构体类型定义时, 还可以 **使用`typedef`给结构体类型取别名**:

```cpp
typedef struct student {
    int _age;
    char _name[20];
} stu, *pstu;
```

这个例子中, 用`typedef`将`struct student`取名为`stu`, 并对其指针类型取名为`pstu`

之后, 就可以直接用`stu`定义结构体变量, 用`pstu`定义结构体指针变量

---

C语言实现 链表的节点的定义使用了结构体:

```cpp
struct listnode {
  	int _data;
    struct listnode* _next;
};
```

在链表中, 可以通过`_next`找到下一个链表节点

不过, 这里有一个容易忽略的问题:

**`struct listnode`这个结构体类型还没有完成定义, 为什么在结构体类型的内部就可以使用`struct listnode`了定义变量了?**

实际上, 这里并不是使用`struct listnode`声明结构体变量, 而是 **声明此结构体指针变量**, 在固定的平台上指针的大小是固定的, 所以并不影响编译器处理

如果是下面这样定义, 就会报错:

```cpp
struct listnode {
  	int _data;
    struct listnode _next;
};
```

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408210940141.webp)

## 结构体变量的定义与初始化

### 定义

最基本的, 以`typedef struct student{};`为例

```cpp
typedef struct student {
    int _num;
    int _age;
    char _name[20];
} stu, *pstu;
```

使用`typedef`完成结构体类型定义之后, 可以分别有两种方式定义结构体变量 以及 两种方式定义结构体指针变量:

```cpp
// 定义结构体变量
struct student stu1;
stu stu2;

// 结构体指针变量
struct student* pstu1 = NULL;
pstu pstu2 = NULL;
```

> 其他定义结构体变量的方式:
>
> 
>
> 如果不使用`typedef`, 还可以在定义结构体类型的同时, 定义结构体变量:
>
> ```cpp
> struct student {
>     int _num;
>     int _age;
>     char _name[20];
> } stu1, stu2;
> ```
>
> ---
>
> **匿名结构体变量**
>
> 在定义结构体类型时, 不指定结构体名, 同时在后面给定变量名, 就可以定义 **匿名结构体变量**:
>
> ```cpp
> struct {
>     int _num;
>     int _age;
>     char _name[20];
> } stu1, stu2, *pstu1;
> 
> struct {
>     int _num;
>     int _age;
>     char _name[20];
> } stu3, *pstu2;
> ```
>
> 关于匿名结构体, 如果不是同一个匿名结构体类型, 即使结构体的成员相同, 编译器也不会认为其是相同的结构体类型
>
> 什么意思? 
>
> 像上面这样定义的匿名结构体变量和指针, 编译器会将 `stu1`、`stu2`和`pstu1` 认为是同一个结构体类型, `stu3`和`psut2`是同一个结构体类型
>
> 即使两个匿名结构体类型的成员一模一样
>
> ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211411082.webp)
>
> 无法将`stu3`赋值给`sut1`, 编译器不认他们是相同或相似类型
>
> 同样的`&stu2`赋值给`pstu2`, 编译器也会警告是不同的指针类型

### 初始化

结构体变量的初始化, 可以用`{ 成员变量值 }`的方式初始化, 比如:

```cpp
struct student stu1 = { 1, 2, "hhh" };
// 分别指定了 stu1中 _num, _age, _name的初始值
```

思考一下, 下面这些初始化 会把结构体变量初始化成什么:

```cpp
// 1
struct student stu1 = { };
// 2 
struct student stu2 = { 0 };
// 3
struct student stu3 = { 1 };
// 4
struct student stu4 = { 1, 2 };
```

打印这些结构体变量的成员值:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211021265.webp)

用这样的方式, 可不可以忽略结构体中前面的成员变量, 只初始化结构体中后面的成员变量?

```cpp
// 比如
struct student stu5 = { "xxx" };
```

很明显不可以:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211030312.webp)

---

不过可以通过另一种方式, 对结构体内后面的成员变量单独初始化

在C99之后, C语言可以通过`{ .成员变量名 = }`

```cpp
// C99 引入
struct student stu6 = { ._name = "xxx" };
// 更可以
struct student stu7 = { ._age = 12, ._name = "xxx" };

// 其他成员处理 同上
```

定义并初始化的结果:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211026295.webp)

## 结构体变量 成员的访问

结构体变量访问结构体成员, 可以使用`.`符号:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211349964.webp)

如果是结构体指针, 则可以通过`->`访问其指向结构体的成员:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211352577.webp)

## 结构体类型的大小(对齐)

```cpp
struct student {
    char _group;
    int _age;
    char _name[8];
};
```

如果单从成员变量的类型来看, `_group`是字符型1字节 `_age`是整型4字节 `_name`是字符数组8字节, 一共应该是`13`字节

不过, 使用`sizeof`计算出此结构体类型的大小是`16`字节:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408212350691.webp)

并且, 如果将`_name[8]`改成`_name[9]`, 那么此结构体类型的大小会变成`20`字节:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408212354972.webp)

出现这种情况, 就是因为 **结构体对齐** 的存在, 结构体大小是在 **结构体对齐** 之后在进行计算的

**结构体对齐**: 编译器对结构体成员在内存中做出的一些处理, 让结构体成员在内存中的存储位置满足一定的对齐要求

编译器也是按照 **结构体对齐的规则** 来进行处理的:

1. **结构体成员的偏移量**, 必须是 **成员对齐数** 的倍数
2. 结构体首成员的偏移量是0
3. 结构体总大小, 必须为 **结构体成员最大对齐数的倍数**
4. 如果结构体内嵌有结构体变量, 那么内嵌的结构体成员变量的成员对齐数是 此内嵌结构体的最大成员对齐数

> **结构体成员偏移量**
>
> 结构体成员的存储地址与结构体变量首地址 之间偏移的字节数, 就是结构体成员偏移量
>
> `offsetof(struct, member)`可以计算成员在结构体内部的偏移量:
>
> ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211923850.webp)
>
> 
>
> **成员对齐数**
>
> 
>
> **C语言编译器中, 成员对齐数是 成员变量类型大小 和 编译器默认对齐数 中的较小值**
>
> gcc, 不设置默认对齐数, 成员对齐数就是成员变量类型的大小
>
> MSVC 默认对齐数是`8`, 成员对齐数是 成员变量类型大小 和 默认对齐数 中的较小值
>
> ---
>
> 可以通过 `#pragma pack(n)` 设置默认对齐数, 让编译器处理, 如果设置为`#pragma pack(1)`, 表示默认对齐数为`1`
>
> ![|wide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211500782.webp)

按照结构体对齐规则, `struct student{};` 在内存中的对齐情况是这样的:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211939606.webp)

Linux平台下, 数据类型的大小就是成员的对齐数, 所以:

1. `_group`的对齐数是1
2. `_age`的对齐数是4
3. `_name`是`char`数组, 其对齐数与`char`相同, 所以是1

按照结构体对齐规则:

1. 首成员`_group`的偏移量为0.

    所以, `_group`在偏移量为0处存储

2. `_age`的偏移量, 需要是其对齐数(4)的倍数, 所以至少是4

    所以, `_age`在偏移量为4处存储

    因为`_group`只占1个字节, 所以前面空出3个字节

3. `_name`的偏移量, 需要是其对齐数(1)的倍数, 可以直接存储在下一字节, 是8

    所以, `_name`在偏移量为8处存储

4. 结构体大小, 必须是其最大成员对齐数的整数倍

    在存储完成员变量之后, 已经占用了`17`个字节, 最大成员对齐数是4

    所以, 结构体大小是`20`字节

分别计算 结构体变量 和 各成员变量的地址:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408211954552.webp)

能够发现

1. `_group`的地址与结构体变量地址相同
2. `_age`的地址为 结构体变量地址+4
3. `_name`的地址为 结构体变量地址+8

与计算偏移量相同

### 结构体对齐的意义

**结构体内存对齐, 可以在一定程度上提高结构体读取数据的效率**

32位CPU, 由于总线宽度限制, 单次访问内存数据的大小通常是4字节

以一个简单的结构体为例:

```cpp
struct stu {
    char _group;
    int _age;
};
```

分析一个此结构体变量 内存不对齐和对齐的情况:

![|lwide](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202408220110132.webp)

如果`CPU`要访问`_age`

对于不对齐的存储方式, 从CPU访问到`_age`数据开始, CPU最少要访问两次, 才能完整的访问到`_age`的数据

访问到`[0, 3]`时 首次包含`_age`数据, 但是不完整, 还需要再访问`[4, 7]`, 其中`[1, 4]`是`_age`的数据



而对于对齐的存储方式, CPU最少只需要访问一次, 就能完整的访问到`_age`的数据

访问到`[4, 7]`时 首次包含`_age`数据, 数据完整



结构体对齐会浪费一定的内存空间, 所以, 结构体对齐是一种典型的用空间换时间的思想
