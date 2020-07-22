# 4. 从尾到头打印链表

## 题目描述

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

示例 1：

输入：

```text
head = [1,3,2]
```

输出：

```text
[2,3,1]
```

限制：

```
0 <= 链表长度 <= 10000
```

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/

## 题解

```js
/** 方法一
 * 辅助栈法：使用一个辅助数组存储从头到位的链表值，再用pop()使值倒序输出
 */
var reversePrint = function(head) {
    let temp = [], res = [];
    while (head) {
        temp.push(head.val);
        head = head.next;
    }
    while (temp.length) {
        res.push(temp.pop());
    }
    return res;
};
/** 方法二
 * 递归法
 */
var reversePrint = function(head) {
    let res = [];
    let _reversePrint = function(head) {
        if (head == null) {
            return;
        }
        _reversePrint(head.next);
        res.push(head.val);
    }
    _reversePrint(head);
    return res;
};
```

