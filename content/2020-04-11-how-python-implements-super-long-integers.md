+++
title = "Python如何实现大数"
slug = "how-python-implements-super-long-integers"
date = 2020-04-11T09:58:23.000Z
excerpt = "翻译，python是如何实现超级大数的呢"

[taxonomies]
tags = [ "Coding" ]
+++

## 文章说明

- 来自电台 [Python Bytes Podcast](https://pythonbytes.fm/)[Episode #176: How python implements super long integers](https://pythonbytes.fm/episodes/show/176/how-python-implements-super-long-integers) 的推荐
- 文章链接：[How python implements super long integers?](https://arpitbhayani.me/blogs/super-long-integers)
- 以下是对文章的翻译和部分个人的理解

## 翻译说明

直到存储节的内容都是由我自己进行翻译，但是存储之后的部分都是在翻译工具 [Deppl](https://www.deepl.com/translator) 的基础上润色而来，如想要支持原作者请前往原文链接，或者文章底部的相关位置对作者进行支持。

在此推荐这个翻译工具，翻译质量的确是比我之前使用的工具都要好很多，翻译时间也没有明显的落后。

## 引言

当你在例如C语言的底层语言上进行编码，你需要担心为你的整数选择正确的数据类型与限定词。在每一步中，您都需要考虑 `int` 是否足够，还是应该选择 `long `，甚至更长的 `long double` ？但是当你利用  python 进行编码时，你不需要担心这些 “不重要的” 细节，因为python支持任意大小的整数。

在C语言中，当你尝试利用内置的 `powl` 函数计算 2^20000 时，将会给出 `inf` 作为输出

    #include <stdio.h>
    #include <math.h>
    
    int main(void) {
      printf("%Lf\n", powl(2, 20000));
      return 0;
    }
    
    $ ./a.out
    inf
    

但是对于python来说，真的是小菜一碟

    >>> 2 ** 20000
    39802768403379665923543072061912024537047727804924259387134 ...
    ...
    ... 6021 digits long ...
    ...
    6309376
    

Python一定在内部做了一些美妙的事情来支持容易大小的整数，今天就让我们一探究竟吧！

## 形式与定义

Python中的一个整数实际上是一个C的结构体，定义如下：

    struct _longobject {
        PyObject_VAR_HEAD
        digit ob_digit[1];
    };
    

`PyObject_VAR_HEAD` 是一个宏，这个宏将会展开成为一个 `PyVarObject` ，结构如下：

    typedef struct {
        PyObject ob_base;
        Py_ssize_t ob_size; /* Number of items in variable part */
    } PyVarObject;
    

其他拥有 `PyObject_VAR_HEAD` 的类型是：

- `PyBytesObject`
- `PyTupleObject`
- `PyListObject`

这表明一个整数，就像一个 `tuple` 或一个 `list` 一样，其长度是可变的，是我们对它如何支持巨型长整数的第一认识。 宏扩展后的 `_longobject` 可以粗略地看成视作以下定义：

    struct _longobject {
        PyObject ob_base;
        Py_ssize_t ob_size; /* Number of items in variable part */
        digit ob_digit[1];
    };
    

有一些在 `PyObject` 结构体中字段,  将会在引用计数（垃圾回收）中被使用，详细讲解这些字段就需要一篇独立的文章。我们目前主要关注的字段是  `ob_digit` 和一定程度的 `ob_size`。

## 解码 `ob_digit`

`ob_size`表示 `ob_digit` 中实际元素的个数。为了在给数组 `ob_digit` 分配内存时更加高效，python会预留数组长度，同时依赖 `ob_size` 的值来决定实际有多少元素在数组中。

## 存储

存储一个整形最简单的方式是在，数组中每一个元素存储一个十进制中数的每位，然后就可像小学数学一样进行加减法等运算操作。

用这种方法，数字 `5238` 将会被存储为： `8325`，如下图：
![image](https://user-images.githubusercontent.com/15051530/79039986-ae626f00-7c17-11ea-9d8c-9a6618f85e9c.png)
这种方法效率很低，因为我们将耗费32位（uint32_t）来存储一个小数位，而实际上这个小数位的范围只有0到9，本来可以很容易地用4位来表示。写出像python这样多功能的语言，作为一个核心开发人员必须要能够更加有效的利用资源。

那么，我们能不能做得更好呢？ 当然可以，否则，这篇文章在互联网上应该是没有立足之地的。下面我们就来深入了解一下python是如何存储超长整数的。

## Pythonic的方式

python并不是在数组 `ob_digit` 的每一个项目中只存储一个十进制数字，而是将数字从基数10转换为基数2^30 ，数组中每个元素的范围是从为从0到2^30 - 1的数字。

在十六进制的数字系统中，基数是16~2^4，这意味着十六进制数字的每个  `bit` 的范围是0~15。同样，对于python来说，"digit "的基数是2^30 ，这意味着它的范围也就是十进制从 `0 ~ 2^30 - 1= 1073741823`。

这样一来，python几乎有效地使用了每个位数32位的分配空间，并且保持了自身的资源优势，仍然可以像小学数学一样执行加减法等操作。

根据平台的不同，Python使用的是32位无符号整数组的30位数位，或者16位无符号整数组的15位数位。它需要几个位来执行操作，这些操作将在以后的一些文章中讨论。

例子：1152921504606846976

如前所述，对于Python来说，一个 "数字 "是基数2^30 ，因此如果将1152921504606846976转换为基数2^30，则得到100

1152921504606846976 = 1 * (2^30)2 + 0 * (2^30)1 + 0 * (2^30)0

由于 `ob_digit` 先将其最低位保留下来，所以它将被存储为 `001`，存储为3个不同的位数。

这个值的 `_longobject` 结构将持有

- 
`ob_size `为 `3`

- 
`ob_digit `为 `[0, 0, 1]`

![image](https://user-images.githubusercontent.com/15051530/79039977-a5719d80-7c17-11ea-8163-35c93dbd28e7.png)
原作者创建了一个解释器展示 [demo REPL](https://repl.it/@arpitbbhayani/super-long-int?language=python3) ，它将输出python内部存储整数的方式，同时也有对结构成员的引用，如 `ob_size `、`ob_refcount `等。

# 大数的运算

现在我们对python如何支持和实现任意精度整数有了一定的了解，是时候了解各种数学运算是如何在它们实现的。

## 加法

I整数是以 "数位为单位 "进行持久化，这意味着加法就像我们在小学时学过的那样简单，而python的源码告诉我们，它也正是这样实现的.。文件[longobject.c](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c)中名为[x_add](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c#L3116)的函数执行两个数的加法。

    ...
        for (i = 0; i < size_b; ++i) {
            carry += a->ob_digit[i] + b->ob_digit[i];
            z->ob_digit[i] = carry & PyLong_MASK;
            carry >>= PyLong_SHIFT;
        }
        for (; i < size_a; ++i) {
            carry += a->ob_digit[i];
            z->ob_digit[i] = carry & PyLong_MASK;
            carry >>= PyLong_SHIFT;
        }
        z->ob_digit[i] = carry;
    ...
    

上面的代码片段取自 `x_add` 函数，你可以看到它在每个数位上进行迭代，执行数字加法，并计算和进位。

> 当加法的结果是一个负数时，事情就变得有趣了。`ob_size` 的符号是整数的符号，这意味着，如果你有一个负数，那么 `ob_size` 将是负数。`ob_size `的绝对值将决定 `ob_digit `中的位数。

## 减法

与加法的实现方式类似，减法也是以数字为单位进行的。文件[longobject.c](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c#L3150)中名为[x_sub](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c)的函数执行两个数字的减法。

    ...
        for (i = 0; i < size_b; ++i) {
            borrow = a->ob_digit[i] - b->ob_digit[i] - borrow;
            z->ob_digit[i] = borrow & PyLong_MASK;
            borrow >>= PyLong_SHIFT;
            borrow &= 1; /* Keep only one sign bit */
        }
        for (; i < size_a; ++i) {
            borrow = a->ob_digit[i] - borrow;
            z->ob_digit[i] = borrow & PyLong_MASK;
            borrow >>= PyLong_SHIFT;
            borrow &= 1; /* Keep only one sign bit */
        }
    ...
    

上面的代码片段来自于`x_sub`函数，你可以看到它是如何迭代数字，并执行减法，计算和借位。确实和加法很相似。

## 乘法

同样，一个最简单的方法来实现乘法，就是我们在小学数学中所学的乘法，但它的效率并不高。为了保证效率，Python实现了[Karatsuba算法](https://en.wikipedia.org/wiki/Karatsuba_algorithm)，它可以用O( nlog23)的基本步数来实现两个n位数的乘法。

这个算法略显复杂，不在本文的范围之内，但你可以在文件 [longobject.c](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c) 中的 [k_mul ](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c#L3397)和 [k_lopsided_mul](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c#L3618) 函数中找到它的实现。

## 除法和其他运算操作

所有关于整数的操作都在文件 [longobject.c](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c) 中定义了，找到并跟踪每一个操作都非常简单。警告：详细了解每一个操作都需要一些时间，所以在开始略读之前先拿些爆米花。

# 对较常使用的整数的优化

Python [预先分配](https://docs.python.org/3/c-api/long.html#c.PyLong_FromLong) 在-5到256的范围内的小整数。这种分配发生在初始化过程中，由于我们不能更新整数 (不变性)，所以这些预分配的整数是单整数，直接引用而不是重新分配。这意味着，每次我们使用/创建一个小整数时，python都会返回预分配的整数的引用，而不是重新分配。

这种优化可以在宏 `IS_SMALL_INT` 和函数[longobject.c](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c#L35)中的函数[get_small_int](https://github.com/arpitbbhayani/cpython/blob/0-base/Objects/longobject.c#L43)中找到。这样python可以为常用的整数节省大量的空间和计算量。

---

这是《Python内部》系列的第二篇文章。第一篇文章是[我是如何改变我的Python并让它变得可疑的](https://arpitbhayani.me/blogs/i-changed-my-python)，它帮助你迈出了Python源代码的第一步，为你成为Python核心开发者铺平了道路。

如果你想看更多这样的文章，请订阅我的时事通讯，并将文章直接发送到你的收件箱。我每周五都会写一些关于工程、系统设计和编程的文章。给原作者点赞[@arpit_bhayani](https://twitter.com/arpit_bhayani)。你可以找到原作者之前的文章[@arpitbhayani.me/blogs](https://arpitbhayani.me/blogs)。
