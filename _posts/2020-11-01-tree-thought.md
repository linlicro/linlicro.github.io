---
layout:     post
title:      算法专题01 - 养成树的思维方式
subtitle:    "\"tree-way thinking\""
date:       2020-11-01
author:     Lin
header-img: img/post-bg-coding0813.jpeg
catalog: true
tags:
    - algorithm
---

> 论「树」思维的重要性: 任何知识体系都是一个树，想要掌握一个知识体系关键是要这些知识变成一个树状结构。

二叉树是最容易培养框架思维的，而且大部分算法技巧，本质上都是树的遍历问题: 明确一个节点要做的事情，然后剩下的事抛给框架。

二叉树的模板(框架)一般都是递归，以下是递归的心法:

* 心法1: 写递归算法的关键是要明确函数的「定义」是什么，然后相信这个定义，利用这个定义推导最终结果，绝不要跳入递归的细节。
* 心法2: 把题目的要求细化成每个节点需要做的事情。

#### 递归框架

想象下「盗梦空间」的场景，从一个梦境进入下一个梦境，不断的往下进入...直到到达最底层:

```java
public void recur(int level, int param) {

 // terminator
 if (level > MAX_LEVEL) {
  // process result
  return;
 }

 // process current level
 process(level, param);

 // drill down
 recur(level + 1, param);

 // restore current status
}
```

*其实也是一个`for`循环: level就是索引。*

#### 树的3种遍历框架

```java
void traverse(TreeNode root) {
    // 前序遍历
    traverse(root.left)
    // 中序遍历
    traverse(root.right)
    // 后序遍历
}
```

可视化: [树的常规操作](https://visualgo.net/zh/bst)

## 二叉搜索树常规操作框架

#### 1. 如何把二叉树的所有的节点的值加一？

```java
void plusOne(TreeNode root) {
    if (root == null) return;
    root.val += 1;

    plusOne(root.left);
    plusOne(root.right);
}
```

#### 2. 如何判断两颗二叉树是否完全相同？

```java
boolean isSame(TreeNode root1, TreeNode root2) {
    if (null == root1 && null == root2) {
        return true;
    }
    if (null == root1 || null == root2) {
        return false;
    }

    return isSame(root1.left, root2.left) && isSame(root1.right, root2.right);
}
```

#### 3. BST的基本操作: 判断 BST 的合法性、增、删、查

`二叉搜索树`（Binary Search Tree，简称 BST）是一种很常用的的二叉树。它的定义是：一个二叉树中，任意节点的值要大于左子树所有节点的值，且要小于右边子树的所有节点的值。

判断 BST 的合法性:

```java
boolean isValidBST(TreeNode root) {
    return isValidBST(root, null, null);
}

boolean isValidBST(TreeNode root, TreeNode min, TreeNode max) {
    if (null == root) {
        return true;
    }
    if (null != min && min.val >= root.val) {
        return false;
    }
    if (null != max && max.val <= root.val) {
        return false;
    }
    return isValidBST(root.left, min, root) && isValidBST(root.right, root, max);
}
```

在BST中查找一个数是否存在:

```java
void BST(TreeNode root, int target) {
    if (root.val == target)
        // 找到目标，做点什么
    if (root.val < target)
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```

在 BST 中插入一个数:

```java
TreeNode insertIntoBST(TreeNode root, int val) {
    if (null == root) {
        return new TreeNode(val);
    }
    if (val > root.val) {
        root.right = insertIntoBST(root.right, val);
    }
    if (val < root.val) {
        root.left = insertIntoBST(root.left, val);
    }
    return root;
}
```

在 BST 中删除一个数:

```java
TreeNode deleteNode(TreeNode root, int key) {
    if (key == root.val) {
        // 找到 并删除
        if (null == root.left) {
            return root.right;
        }
        if (null == root.left) {
            return root.left;
        }
        TreeNode minNode = getMin(root.right);
        root.value = minNode.val;
        root.right = deleteNode(root.right, minNode.val);
    } else if (key < root.val) {
        root.left = deleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = deleteNode(root.right, key)
    }
}

TreeNode getMin(TreeNode node) {
    while (null != node.left) {
        node = node.left;
    }
    return node;
}
```

## leetcode 树相关题目

[tree](https://leetcode.com/tag/tree/)

[Binary Search Tree](https://leetcode.com/tag/binary-search-tree/)

## 参考资料

* leetcode
* [二叉搜索树操作集锦](https://mp.weixin.qq.com/s?__biz=MzAxODQxMDM0Mw==&mid=2247484518&idx=1&sn=f8ef8d7ce7959b4fd779e38f47419ac6&chksm=9bd7fa6eaca073785cb6f808421241bcb641203c8ec7f30a9269a221b3d92c661334af1b75f5&mpshare=1&scene=1&srcid=1016VKGMSUBiv0ArmK3CjvYq&sharer_sharetime=1602810380177&sharer_shareid=22ef14d9fe403127491b73da1fc63897#rd)
* [手把手刷二叉树（训练递归思维）](https://labuladong.gitbook.io/algo/shu-ju-jie-gou-xi-lie/2.3-shou-ba-shou-shua-er-cha-shu-xun-lian-di-gui-si-wei)
