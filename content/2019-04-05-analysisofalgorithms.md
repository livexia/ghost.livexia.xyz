+++
title = "Analysis Of Algorithms"
slug = "analysisofalgorithms"
date = 2019-04-05T15:12:59.000Z
excerpt = "普林斯顿的算法课程学习笔记，第一周算法分析的学习笔记。"

[taxonomies]
tags = [ "Coding" ]
+++

### Analysis Of Algorithms：算法分析

科学的方法：Observe、Hypothesize 、Predict 、Verify、Validate

- 观察自然界的一些特征。
- 假设一个与观察结果一致的模型。
- 使用假设预测事件。
- 通过进一步观察来验证预测。
- 重复验证，直到假设和观察结果一致。

### Observation

3-Sum problem：给定一系列的输入整数，输出其中三个相加为0的整数个数。

Java存在一个stopwatch，可以给出程序运行时间，stopwatch.elapsedTime()，给出stopwatch声明到当前所花费的时间。

双对数坐标：lg(T(N))=b*lg(N)+c

回归：a*N^b, a=2^c, b=lg ratio

影响b大小的因素：算法、输入数据

影响a大小的因素：计算机之间的差距、运行环境

### Mathematical Models

Don Knuth提出。

读取数组类似于二项式系数。

部分离散数学基础。

相近的数学模型非常有用。

### Order Of Growth Classification

Functions：1，logN，N，Nlog N，N^2，N^3，2^N

二分搜索：复杂度的证明

### Theory of Algotithms

Best case:

Worst case:

Average case:Random input，预测结果。

根据最差输入，设计算法进行规避。

构造随机化

### Memory
