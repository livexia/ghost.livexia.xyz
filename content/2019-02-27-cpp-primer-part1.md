+++
title = "C++ Primer笔记-第一部分"
slug = "cpp-primer-part1"
date = 2019-03-04T06:57:50.000Z
excerpt = "C++ Primer 5th 第一部分 C++ 基础，学习笔记"

[taxonomies]
tags = [ "Coding" ]
+++

### 1.1

main 函数的返回类型必须为int。

<< 运算符左侧ostream对象，右侧为打印值，返回左侧对象

\>> 运算符左侧istream对象，右侧为运算对象（输入对象），返回左侧对象。从istream中读入数据，存入右侧对象。

endl 结束行，同时刷新流（确保缓冲区输出到设备）

:: 作用域运算符，指定命名空间（std）中的名字（cout），std::cout

cout 标准输出

cin 标准输入

clog log输出（Linux中的2）

cerr 错误输出（Linux中的2）

注释

1. 单行注释：//
2. 多行注释：注释界定符（/*和*/），中间行往往在开头包含*表明属于多行注释的一部分。

C++ 生成a-b间的随机数：rand() % (b - a + 1) + a

> 添加srand(time(NULL)); 生成更加随机的序列[stackOverflow](https://stackoverflow.com/questions/9459035/why-does-rand-yield-the-same-sequence-of-numbers-on-every-run)
> 
> 1000-9999 number = rand() % 9000 + 1000;

Linux 管道重定向说明：

> $ program <infile >outfile
> 
> /dev/null 表示空设备文件
> 
> 0 表示stdin标准输入
> 
> 1 表示stdout标准输出
> 
> 2 表示stderr标准错误

> 页数 52/864
