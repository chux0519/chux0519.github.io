---
title: "Leetcode Rust Bfs"
date: 2019-10-22T22:40:20+08:00
showDate: true
tags: ["rust","leetcode"]
---

## 算法系列 - BFS

BFS 指广度优先搜索，是一种遍历树、图的方法。

BFS 通常会和队列相关，但不完全等于队列。以下是我在 leetcode 刷题后的整理。


<!--more-->

## 层级相关

首先存在一类题目，通常会和树的层级遍历相关，这类型的题目是存在固定解题模式的。

首先定义节点

```rust
#[derive(Debug, PartialEq, Eq)]
pub struct TreeNode {
    pub val: i32,
    pub left: Option<Rc<RefCell<TreeNode>>>,
    pub right: Option<Rc<RefCell<TreeNode>>>,
}
```

然后模板通常为以下，或稍微有些变化，但主体框架都是一致的。

```rust
// root: Option<Rc<RefCell<TreeNode>>>
let mut q = VecDeque::new();
q.push_back(root);
while !q.is_empty() {
    // 每一层
    let mut level = Vec::new();
    for _ in 0..q.len() {
        // 在遍历过程中，q 会被更新，进入下一层
        let front = q.pop_front().unwrap();
        if let Some(f) = front {
            let f = f.borrow();
            level.push(f.val);
            if f.left.is_some() {
                q.push_back(f.left.clone());
            }
            if f.right.is_some() {
                q.push_back(f.right.clone());
            }
        }
    }
    // 可能会有其他逻辑
}
```

属于这类型的题目有： q102, q103, q107, q111

## 和其他思想结合

除了一眼能看拿出来的层级遍历结构，某些题会使用到其他的数据结构或是思想。比如利用 hashset 进行缓存，然后在层级中做某些事情。

这类型的题型有：126, 127

对于这类特征不太明显的题目，只需要记得，比如最短路径、最近距离之类的描述，可以想想是不是和一用 BFS 的框架来思考就行了。

另外值得一提的是 127 题和 126 题的思路稍微有点不同，对于 126 题，它的维度要更高一点，即，想的时候，把返回结果的每个元素当成 BFS 的对象，这样可以稍作改动，减轻思考深度负担。

126 卡了我很久，这个[链接](https://leetcode.com/problems/word-ladder-ii/discuss/40434/C%2B%2B-solution-using-standard-BFS-method-no-DFS-or-backtracking)可以看到比较容易理解、纯粹的 BFS。
