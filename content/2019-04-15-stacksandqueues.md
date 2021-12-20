+++
title = "Stacks  and Queues"
slug = "stacksandqueues"
date = 2019-04-15T11:01:11.000Z
excerpt = "普林斯顿的算法课程学习笔记，第二周关于基础数据结构的学习笔记。"

[taxonomies]
tags = [ "Coding" ]
+++

Stacks（栈）：后进先出LIFO

主要方法：

- push()
- pop()：返回弹出的值
- isempty()
- size\[optional\]()

linked-list实现(Java)：

    inner class
    private class NODE
    {
        String item;
        Node next;
    }
    

Resizing array实现(Java):

当push超过数组长度时，翻倍数组（新建数组，复制原数据）。

Efficient solution.

・push(): double size of array s[] when array is full.

・pop(): halve size of array s[] when array is one-quarter full.

Queues（队列）：先进先出FIFO，排队

linked-list：

Generics（泛型）：

Primitive（基本类型）：

Iterator（迭代器）：

Bags（包）：

    public class Bag<Item> implements Iterable<Item>
        Bag() create an empty bag
        void add(Item x) insert a new item onto bag
        int size() number of items in bag
        Iterable<Item> iterator() iterator for all items in bag
    
