# 5.重建二叉树

## 题目描述

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如，给出

```
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
```


返回如下的二叉树：

      3
     / \
     9  20
    /    \
    15    7

限制：

```
0 <= 节点个数 <= 5000
```



> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof



## 题解

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {number[]} preorder
 * @param {number[]} inorder
 * @return {TreeNode}
 */
var buildTree = function(preorder, inorder) {
    if (!preorder.length) return null;
    let head = new TreeNode(preorder[0]);
    let headIndex = inorder.indexOf(head.val); // 头节点在中序数组中的下标
    let leftPreorder = preorder.slice(1, headIndex+1); // 左子树前序
    let rightPreorder = preorder.slice(headIndex+1); // 右子树前序
    let leftInorder = inorder.slice(0, headIndex); // 左子树中序
    let rightInorder = inorder.slice(headIndex+1); // 右子树中序
    head.left = buildTree(leftPreorder, leftInorder); // 递归建立左右子树
    head.right = buildTree(rightPreorder, rightInorder);
    return head;
};
```

