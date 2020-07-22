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

### JAVA 版本

首先最容易想到的就是暴力解法，遍历一遍，用一个栈来存值，然后再从栈中把值弹出写入数组。

```java
public int[] reversePrint(ListNode head) {
    Stack<Integer> stack = new Stack<>();
    while (head != null) { // 遍历链表，压入栈中
        stack.push(head.val);
        head = head.next;
    }
    int[] res = new int[stack.size()];
    int cnt = 0;
    while (!stack.isEmpty()) {
        res[cnt++] = stack.pop();
    }
    return res;
}
```

如果要节省栈的空间，我们可以考虑先遍历一次获取链表长度，再遍历一次，从数组末尾开始写入值。

```java
public int[] reversePrint(ListNode head) {
    int cnt = 0;
    ListNode temp = head;
    while (temp != null) {
        temp = temp.next;
        cnt++;
    }
    int[] res = new int[cnt];
    while (head != null) {
        res[--cnt] = head.val;
        head = head.next;
    }
    return res;
}
```

![image-20200723005305727](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh08iwb28gj30y20akmyc.jpg)

### js 版本

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

