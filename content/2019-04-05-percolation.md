+++
title = "Percolation"
slug = "percolation"
date = 2019-04-05T15:12:17.000Z
excerpt = "普林斯顿的算法课程学习笔记，并查集的练习渗透（Percolaction）的学习笔记。"

[taxonomies]
tags = [ "Coding" ]
+++

Java基础：

关键点：

时间复杂度：允许Backwash时间复杂度下降，所有Memory/Timing Test passed，但是会有部分测试安利错误。出现原因，当系统已经渗透成功，如果最底层存在open sites，所有open sites都会被标记为full，无论其是否与top连接。

#### 要求分析/限制：

主要包含两个实验（Java Class），第一个是渗透模型具体API限制如下。

    public class Percolation {
       public Percolation(int n)                // create n-by-n grid, with all sites blocked
       public    void open(int row, int col)    // open site (row, col) if it is not open already
       public boolean isOpen(int row, int col)  // is site (row, col) open?
       public boolean isFull(int row, int col)  // is site (row, col) full?
       public     int numberOfOpenSites()       // number of open sites
       public boolean percolates()              // does the system percolate?
    
       public static void main(String[] args)   // test client (optional)
    }
    

第二个是蒙特卡洛模拟（Monte Carlo simulation），模拟是为了探究当一个，具体API限制如下。

    public class PercolationStats {
       public PercolationStats(int n, int trials)    // perform trials independent experiments on an n-by-n grid
       public double mean()                          // sample mean of percolation threshold
       public double stddev()                        // sample standard deviation of percolation threshold
       public double confidenceLo()                  // low  endpoint of 95% confidence interval
       public double confidenceHi()                  // high endpoint of 95% confidence interval
    
       public static void main(String[] args)        // test client (described below)
    }
    

- 方法II：95/100，错误3

#### 遇到问题：

Backwash解决方法：

1. 利用两个WQUF（路径压缩的带权并查集），第一个含有一个虚拟Top、Bottom，该wquf可以快速判断是否渗透。第二个wquf只含虚拟Top，不会出现Backwash，在判断isFull的时候，利用第二个wquf来判断。缺点：内存/时间复杂度翻倍。
2. 直接取消虚拟Bottom，判断是否渗透时，循环扫描Bottom line的每一个site是否连接到Top。缺点时间复杂度大大增加，请求wquf中find方法的数量大大增加。
3. 修改WQUF，在每一个site中存储一个“状态码”，状态码代表其是否连接到虚拟Top或者虚拟Bottom，每次union都更新该状态，当且仅当某一同时含有Top/Bottom时渗透发生。参见@sigmainfy的方法。参见示例代码II。

Java示例代码I：

    import edu.princeton.cs.algs4.In;
    import edu.princeton.cs.algs4.StdOut;
    import edu.princeton.cs.algs4.WeightedQuickUnionUF;
    
    // way2.Use both virtual bottom and top, backwash exists
    // score 95/100
    //Correctness:  30/33 tests passed
    //Memory:       8/8 tests passed
    //Timing:       20/20 tests passed
    public class Percolation {
        private boolean[][] g;
        private int count;
        private int opensites;
        private WeightedQuickUnionUF weightedQuickUnionUF;
    
        public Percolation(int n) {
            if (n <= 0) {
                throw new IllegalArgumentException("outside its prescribed range");
            }
            count = n;
            g = new boolean[n + 1][n + 1];
            weightedQuickUnionUF = new WeightedQuickUnionUF(n * n + 2);
            for (int i = 1; i < n + 1; i++) {
                for (int j = 1; j < n + 1; j++) {
                    g[i][j] = false;
                }
            }
        }
    
        public void open(int row, int col) {
            if (!isOpen(row, col)) {
                g[row][col] = true;
                if (row == 1)
                    weightedQuickUnionUF.union(0, (row - 1) * count + col);
                if (row == count)
                    weightedQuickUnionUF.union(count*count+1, (row - 1) * count + col);
                if (row>1 && isOpen(row - 1, col))
                    weightedQuickUnionUF.union((row - 1) * count + col, (row - 2) * count + col);
                if (row<count && isOpen(row + 1, col))
                    weightedQuickUnionUF.union((row - 1) * count + col, row * count + col);
                if (col>1 && isOpen(row, col - 1))
                    weightedQuickUnionUF.union((row - 1) * count + col, (row - 1) * count + col - 1);
                if (col < count && isOpen(row, col + 1))
                    weightedQuickUnionUF.union((row - 1) * count + col, (row - 1) * count + col + 1);
                opensites += 1;
            }
        }
    
        public boolean isOpen(int row, int col) {
            if (row >= 1 && row <= count && col >= 1 && col <= count)
                return g[row][col];
            else {
                throw new IllegalArgumentException(row + "," + col + " outside its prescribed range");
            }
        }
    
        public boolean isFull(int row, int col) {
            if (row >= 1 && row <= count && col >= 1 && col <= count)
                return weightedQuickUnionUF.connected(0, (row - 1) * count + col);
            else {
                throw new IllegalArgumentException(row + "," + col + " outside its prescribed range");
            }
        }
    
        public int numberOfOpenSites() {
            return opensites;
        }
    
        public boolean percolates() {
            return weightedQuickUnionUF.connected(0, count*count+1);
        }
    
        public static void main(String[] args) {
            In in = new In(args[0]);      // input file
            int n = in.readInt();
            Percolation percolation = new Percolation(n);
            while (!in.isEmpty()) {
                int row = in.readInt();
                int col = in.readInt();
                percolation.open(row, col);
                StdOut.println("isPercolates: " + percolation.percolates());
            }
        }
    }
    
    
