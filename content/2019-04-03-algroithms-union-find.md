+++
title = "Algorithms——Union-Find"
slug = "algroithms-union-find"
date = 2019-04-03T12:18:39.000Z
excerpt = "普林斯顿的算法课程学习笔记，并查集的学习笔记。"

[taxonomies]
tags = [ "Coding" ]
+++

并查集（Union-Find）问题：研究动态连接性

基础算法：快速查找（Quick Find）、快速连接（Quick Union）

提升算法：带权的并查集（Weighted Union Find）、路径压缩后带权的并查集（Weighted Union Find with Path Compression）

快速查找（Quick Find）：维护数组id[i]，i代表各个连接单位，不同索引id[i]相同时代表这一系列i属于同一连通风量。若id[1]=id[2]=id[4]，则1、2、4三个单位处于同一连通分量，即1、2、4互相连接。

快速连接（Quick Union）：利用数组id[i]，维护森林，所有相连的单位处于同一棵树内，连接（Union）使得两颗不同的树合并为一棵树。id[i]的值为i的父亲，如果id[i]=i则i为i所处的树的根（root）。每当一棵树并入另一棵树，将并入树的根改为待并入树的根。

带权的并查集（Weighted Union Find）：改善Quick-Union，额外维护数组sz[i]，存储根为i的树的所有结点量/权。当两棵树合并时，比较两棵树的权，权较小的树并入权较大的树。改善的目的是使得在树中较少出现“一枝独秀”。树的深度不超过lgN，每当叶子的深度增加时，该树最少并入相同大小的树，此时深度+1，若共有N个节点，则最多可以进行lgN次合并，使得所有节点处于同一颗树，该树深度最大，一次合并深度增加1。

路径压缩后带权的并查集（Weighted Union Find with Path Compression）：改善Weighted Union Find，相比带权并查集并不维护额外的数组。当每一次进行并、查操作时，尽量使得当前节点直接指向根（root），示例代码中仅仅使查询节点指向其爷爷。

算法复杂度：
![](__GHOST_URL__/content/images/2019/04/M-union-find-operations-on-a-set-of-N-objects.png)
Python示例代码：

Quick-Find：

    class QF:
        id = []
        count = 0
        def __init__(self, n):
            self.id = list(range(n))
            self.count = n
    
        def union(self, p, q):
            if self.connected(p,q):
                pass
            else:
                old_p = self.id[p]
                for i in range(self.count):
                    if self.id[i] == old_p:
                        self.id[i] = self.id[q]
    
        def connected(self, p, q):
            if self.id[p] == self.id[q]:
                return 1
            return 0
    

Quick-Union：

    class QU:
        id = []
        count = 0
        def __init__(self, n):
            self.id = list(range(n))
            self.count = n
    
        def root(self, p):
            while p != self.id[p]:
                p = self.id[p]
            return p
    
        def union(self, p, q):
            if self.connected(p,q):
                pass
            else:
                self.id[self.root(p)] = self.root(q)
    
        def connected(self, p, q):
            if self.root(p) == self.root(q):
                return 1
            return 0
    

Weighted Union Find：

    class WQU:
        id = []
        sz = []
        count = 0
        def __init__(self, n):
            self.id = list(range(n))
            self.sz = [1] * n
            self.count = n
    
        def root(self, p):
            while p != self.id[p]:
                p = self.id[p]
            return p
    
        def union(self, p, q):
            if self.connected(p,q):
                pass
            else:
                rootp = self.root(p)
                rootq = self.root(q)
                if self.sz[rootp] >= self.sz[rootq]:
                    self.id[rootq] = rootp
                    self.sz[rootp] += self.sz[rootq]
                else:
                    self.id[rootp] = rootq
                    self.sz[rootq] += self.sz[rootp]
    
        def connected(self, p, q):
            if self.root(p) == self.root(q):
                return 1
            return 0
    

Weighted Union Find With Compression：

    class WQUPC:
        id = []
        sz = []
        count = 0
        def __init__(self, n):
            self.id = list(range(n))
            self.sz = [1] * n
            self.count = n
    
        def root(self, p):
            while p != self.id[p]:
                self.id[p] = self.id[self.id[p]]
                p = self.id[p]
            return p
    
        def union(self, p, q):
            if self.connected(p,q):
                pass
            else:
                rootp = self.root(p)
                rootq = self.root(q)
                if self.sz[rootp] >= self.sz[rootq]:
                    self.id[rootq] = rootp
                    self.sz[rootp] += self.sz[rootq]
                else:
                    self.id[rootp] = rootq
                    self.sz[rootq] += self.sz[rootp]
    
        def connected(self, p, q):
            if self.root(p) == self.root(q):
                return 1
            return 0
    

C语言示例代码：

#### TODO
