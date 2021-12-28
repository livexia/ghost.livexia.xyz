+++
title = "构建二叉树解决Advent of Code 2021 18天"
slug = "build-a-binary-tree-to-solve-aoc-ay18"
date = 2021-12-28

[taxonomies]
tags = [ "Coding" ]
+++

记录在2021.12.28日，利用 Rust 构建二叉树解决 Advent of Code 2021 18天。

代码仓库：[livexia/advent-of-code-2021 Day 18](https://github.com/livexia/advent-of-code-2021/blob/main/aoc18)

Advent of Code 网站：[Advent of Code 2021 Day 18](https://adventofcode.com/2021/day/18)

---

在题目最初放出的时候，我曾就考虑过利用构建树来解决。但是那个时候我没有足够的耐心来理清楚所有的所有权的情况，于是最后我选择使用字符串解析的方式来完成。

## 树结构

根据题目所要求的，对于每个二叉树的节点的成员如下：

1. 可能有或者没有value
2. 可能有或者没有左、右子节点
3. 需要记录树的高度（深度）

所以我们可以这样描述树节点：(`Int` 是我指定的 `u8` 的别名)

```rust
struct Number {
    value: Option<Int>,
    left: Option<Box<Number>>,
    right: Option<Box<Number>>,
    height: usize,
}
```

因为 Rust 无法确定一个结构体递归的包含自己时的大小，所以需要用 Box 将左右节点进行包装，也就是说编译器在编译的时候，知道这里是一个指针（Box）。在写这篇文章的时候， 我查到还有利用数组进行构造二叉树的情况。参见：[When to use arrays and when to box trees?](https://users.rust-lang.org/t/when-to-use-arrays-and-when-to-box-trees/24805)

## 解析字符串构造树节点

### 字符串预处理

读入的数据为字符串，每一行为一个数字。将字符串的每个字符转为对应的数字，并进行倒叙存储。

```rust
fn from_str(s: &str) -> Result<Option<Box<Self>>> {
		let mut stack: Vec<u8> = s
		    .bytes()
		    .filter(|c| !c.is_ascii_whitespace())
		    .rev()
		    .collect();
		stack.pop();
		Ok(Number::from_stack(&mut stack))
}
```

### 递归生成树

弹出栈内容（这个时候弹出顺序应该是初始字符串的倒叙），针对所有可能的字符有如下的处理方式：

1. ‘[’ as u8：存在子节点，将剩余栈递归传入。
2. ‘]’ as u8：当前节点所有成员构造完成，返回当前节点。
3. ‘,’ as u8：当前节点只完成了左节点构造，仍需等待右节点。
4. 数字：是当前节点的一个不存在左右节点，但是value为数字值的子节点（叶子节点）。

```rust
fn from_stack(stack: &mut Vec<u8>) -> Option<Box<Self>> {
		let mut number = Number::new(None, None, None);
    while let Some(b) = stack.pop() {
        if b == ']' as u8 {
            break;
        } else if b == ',' as u8 {
            continue;
        } else if b == '[' as u8 {
            number.add_child(Number::from_stack(stack));
        } else {
            number.add_child(Some(Box::new(Number::with_value(b - '0' as u8))));
        }
    }
    number.update_height();
    Some(Box::new(number))
}
```

### 增加子节点

因为是倒叙弹出，那么每次增加节点的时候，必然是先增加左节点，然后是右节点，然后返回当前节点。

```rust
fn add_child(&mut self, child: Option<Box<Number>>) {
    if self.left.is_none() {
        self.left = child
    } else {
        self.right = child
    }
}
```

### 记录深度

因为后续需要根据节点深度判断是否进行操作，所以我选择以根节点深度为0，而叶子节点的深度最大的形式来存储节点深度。递归的对子树的深度进行更新。

```rust
fn update_height(&mut self) {
    if let Some(left) = self.left.as_mut() {
        left.height = self.height + 1;
        left.update_height();
    }
    if let Some(right) = self.right.as_mut() {
        right.height = self.height + 1;
        right.update_height();
    }
}
```

## 对树进行爆炸

根据题目的说明，理论上只有当节点的两个子节点都是叶子节点时才会出现爆炸的情况。

爆炸处理，就是当一个节点的高度大于3（从0开始）的时候，需要将该节点左子树的值给该子树左边最近的叶子节点，该节点右子树的值给该子树右边最近的叶子节点。对于树的遍历，存在前序、中序、后序和层次遍历四种，其中前序、中序、后序中大致都是先遍历左子树，然后遍历右子树的顺序，理论上在这里选择任意一种都无所谓，因为无论是哪一种情况都需要更新最新的最左边的节点。而层次遍历并无法针对题目的要求有何优点，不考虑。

针对当前节点深度大于3的情况，那么需要将当前节点的左子树的值传给最近的左子树，如果是从左到右的顺序，那么理论上我们可以更新一个变量，这个变量记录最左边的叶子节点的地址，因为需要对这个叶子节点进行修改，所以需要是 `mut` 。理论上，当前节点爆炸之后，当前节点无论再何种情况下都是会变成值为0的叶子节点。

假如当前的节点深度小于4，那么针对左右子树进行递归的搜索即可。针对题目的要求，每一次最多只有一个节点会爆炸，所以传递一个变量 exploded 记录是否已经出现过爆炸，如果已经出现过，那么不再进行爆炸。实际上假如不加入这个变量，那么在处于同一棵树下的两个子树的深度都大于3的情况时，会导致两次爆炸进行干扰。

```rust
fn explode<'a>(
    &'a mut self,
    leftmost: Option<&'a mut Option<Box<Number>>>,
    right_value: Option<u8>,
    exploded: bool,
) -> (Option<&'a mut Option<Box<Number>>>, Option<u8>, bool) {
    if self.height > 3 && !exploded {
        let right_value = self.right.take().unwrap().value;
        self.value = Some(0);
        if let Some(left_value) = self.left.take().unwrap().value {
            if let Some(Some(leftmost)) = leftmost {
                if let Some(value) = leftmost.value {
                    leftmost.value = Some(value + left_value);
                }
            }
        }
        (None, right_value, true)
    } else {
        let (leftmost, right_value, exploded) =
            Number::find_right(&mut self.left, leftmost, right_value, exploded);
        let (leftmost, right_value, exploded) =
            Number::find_right(&mut self.right, leftmost, right_value, exploded);
        (leftmost, right_value, exploded)
    }
}
```

那么如何找到右边的第一个叶子节点呢？继续进行遍历，找到的第一个叶子节点就是当前节点右边的第一个子树。当找到时，修改该叶子节点的值。假如当前节点不是叶子节点时，对当前节点进行爆炸操作。

```rust
fn find_right<'a>(
    child: &'a mut Option<Box<Number>>,
    leftmost: Option<&'a mut Option<Box<Number>>>,
    right_value: Option<u8>,
    exploded: bool,
) -> (Option<&'a mut Option<Box<Number>>>, Option<u8>, bool) {
    let value = right_value.clone();
    if child.as_ref().unwrap().value.is_some() {
        if let Some(value) = value {
            child.as_mut().unwrap().value =
                Some(child.as_ref().unwrap().value.unwrap() + value);
            (leftmost, None, exploded)
        } else {
            (Some(child), right_value, exploded)
        }
    } else {
        child
            .as_mut()
            .unwrap()
            .explode(leftmost, right_value, exploded)
    }
}
```

## 对树进行分裂

假如当前树存在叶子节点的值大于9的时候，需要对该叶子节点进行分裂。注意每次分裂，根据题目要求，只能对一个叶子节点进行分裂。

```rust
fn split(&mut self) -> bool {
    if let Some(value) = self.value {
        if value > 9 {
            let left_value = value / 2;
            self.add_child(Some(Box::new(Number::with_value(left_value))));
            self.add_child(Some(Box::new(Number::with_value(value - left_value))));
            self.value = None;
            self.update_height();
            return true;
        }
        false
    } else {
        if self.left.as_mut().unwrap().split() {
            return true;
        }
        if self.right.as_mut().unwrap().split() {
            return true;
        }
        false
    }
}
```

## Reduce操作

Reduce的流程是，不停的进行爆炸，直到树的深度都不大于3的时候，对树进行分裂，当树无法爆炸和分裂的时候，reduce完成。

```rust
fn reduce(&mut self) {
    let mut reduced = false;
    while !reduced {
        let result = self.explode(None, None, false);
        if result.2 {
            reduced = false;
            continue;
        }
        reduced = !self.split();
    }
}
```

## 计算magnitude

假如当前节点是叶子节点，直接返回节点的值。节点的magnitude，是左节点magnitude的3倍和右节点magnitude的2倍的和。递归进行计算即可。

```rust
fn calc_magnitude(&self) -> i32 {
    if let Some(value) = self.value {
        return value as i32;
    }
    let mut result = 0;
    result += 3 * self.left.as_ref().unwrap().calc_magnitude();
    result += 2 * self.right.as_ref().unwrap().calc_magnitude();

    result
}
```

## 总结

这次花了大概8个小时完成了这个遗留的任务，现在完成了反过来看，倒是没有那么复杂了。最复杂的地方就是爆炸遍历树的时候，需要记录并修改已经遍历过的节点，这就难倒了我。通过这次的重复练习，我对 Rust 中关于所有权和生命周期的理解更深了一点，对于利用迭代解决问题的思维更加熟悉了。

**针对涉及到的所有权和生命周期的几个小点：**

1. 对于 `Option<Box<TreeNode>>` 这样的节点 `node` ，假如需要修改这个节点，那么需要使用 `node.as_mut().unwrap()` 取得 `&mut Box<TreeNode>`
2. 对于需要取得子节点的值，并设子节点为 `None` 的情况，可以使用 `take` 。这个时候的子节点的所有权就从节点中取得了，子节点生命周期不再同节点有关。
    
    ```rust
    let node = Some(Box::new(TreeNode::new()));
    let left = node.left.take();    // node.left.is_none();
    ```
    
3. 对于需要将引用作为函数参数的情况中，不用害怕加上生命周期标志，特别是需要修改子树的情况中。

**对于涉及到递归的情况：**

1. 注意递归在什么情况下会停止
    1. 在计算深度的时候，当不存在子节点的时候，递归就不再深入了。
    2. 在计算magnitude的情况中，当遇到叶子节点的时候，递归就不再深入了。
    3. 在进行分裂的情况中，当分裂已经发生时，递归就不再深入了。
    4. 在爆炸过程中，当左右叶子节点的值都耗尽时，递归就不再深入了。但是为了方便，我利用变量 exploded 来表示，实际上当爆炸发生过后，实际上递归就不再深入了。
2. 当前状态和下一个状态的关系
    1. 在计算深度的时候，子节点的深度是父节点的深度加一。
    2. 在计算magnitude的情况中，父节点的magnitude是子节点的magnitude计算而来。
    3. 在进行分裂的情况中，叶子节点的构造取决于父节点的值。
    4. 在爆炸过程中，下一个叶子节点的值需要加上当前节点的右叶子节点的值。对于左叶子，实际上是当前的左叶子的引用是由上一个状态更新的引用值。
    

## 参考

1. [https://gist.github.com/aidanhs/5ac9088ca0f6bdd4a370](https://gist.github.com/aidanhs/5ac9088ca0f6bdd4a370)
2. [https://zh.wikipedia.org/wiki/树的遍历](https://zh.wikipedia.org/wiki/%E6%A0%91%E7%9A%84%E9%81%8D%E5%8E%86)
3. [https://users.rust-lang.org/t/when-to-use-arrays-and-when-to-box-trees/24805/1](https://users.rust-lang.org/t/when-to-use-arrays-and-when-to-box-trees/24805/1)