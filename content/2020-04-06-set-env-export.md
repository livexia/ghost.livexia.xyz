+++
title = "Linux下set、env、export的不同"
slug = "set-env-export"
date = 2020-11-26T14:50:17.000Z
excerpt = "此文转载而来，在工作时对Linux下的各种环境变量设置有所疑惑，以下是对原文的翻译"

[taxonomies]
tags = [ "Coding" ]
+++

**此文转载而来，在工作时对Linux下的各种环境变量设置有所疑惑，以下是对原文的翻译**

> 原文：[What's the difference between set, export and env and when should I use each?](https://askubuntu.com/a/205698)

让我们考虑以下这个特定的例子，`grep`命令利用一个`GREP_OPTIONS` 环境变量（environment variable）来设置默认的选项。

现在，给定一个文件 `test.txt` 含有以下行：

    line one
    line two
    

运行命令 `grep one test.xt` 将会返回以下内容：

    line one
    

如果你运行 `grep` 时附带 `-v` 参数，将会返回所有不匹配的行，所以输出如下：

    line two
    

## **我们将尝试利用环境变量设定这个选项**

1. 
**不利用 `export` 设定环境变量** 将不会被你调用命令的环境所继承。

    GREP_OPTIONS='-v'
    grep one test.txt
    

结果：

    line one
    

很显然，参数 `-v` 没有成功的传入`grep`程序中

*只有在你希望设定一个变量仅在当前运行shell内生效时才采用这种形式，例如在`for i in \* ; do`中你不会希望  ` export $i`.

2. 
然而，这个变量将会被传给当前命令的运行环境中，所以你也可以这样做使其生效：

    GREP_OPTIONS='-v' grep one test.txt
    

这样就会返回预期的结果了

    line two
    

*当你想临时改变当前运行实例程序的环境时，采用这种方法。*

3. 
`export` 一个变量会使得这个变量被继承:

    export GREP_OPTIONS='-v'
    grep one test.txt
    

现在返回

    line two
    

*这种形式最常见的使用场景是在，在shell中设置变量以供随后启动的进程使用*

4. 
这里所说的都是在bash中的实现。 `export` 是bash的内建方法; `VAR=whatever` 则是bash的语法。 `env`, 在另一方面来说, 自己就是一个程序。当 `env` 被调用时, 下列事情将会发生:

1. 命令 `env` 被作为新的进程进行执行
2. `env` 修改环境的同时
3. 调起作为参数传入的命令， `env` 进程被 `command` 进程替换

Example:

    env GREP_OPTIONS='-v' grep one test.txt
    

这个命令会启动两个进程: (i) env and (ii) grep (实际上，第二个进程会替代第一个进程). 从 `grep` 进程的角度来看, 运行的结果与运行下面的命令一模一样：

    GREP_OPTIONS='-v' grep one test.txt
    

然而，当你不在bash内或者不希望唤起另一个shell时，你可以使用这个方法 (例如，当你使用 `exec()` 类似的函数而不是 `system()` 调用).

**关于 `#!/usr/bin/env` 的附加说明**

这也是为什么， 使用`#!/usr/bin/env interpreter` 而不是 `#!/usr/bin/interpreter`. `env` 并不需要完整的程序路径, 因为它使用 `execvp()` 函数遍历的搜索 `PATH` 变量，就像shell会做的一样,  然后将运行命令替换自身。 因此，它可以用作寻找解释器 (例如 perl、 python) 的"位置".

这也表示，通过修改当前的PATH变量，你可以影响到哪个python的程序会被调用。这使得下列命令有可能执行成功：

    echo -e '#!/usr/bin/bash\n\necho I am an evil interpreter!' > python
    chmod a+x ./python
    export PATH=.
    calibre
    

会返回如下结果，而不是调起Calibre

    I am an evil interpreter!
    

说明：Calibre是一个自由开源的电子书软件套装，由python编写，需要依赖python解释器才能运行
