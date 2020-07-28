# 6.用两个栈实现队列

## 题目描述

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

**示例 1：**

输入：

```
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
```

输出：

```
[null,null,3,-1]
```

**示例 2：**

输入：

```
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
```

输出：

```
[null,-1,null,null,5,2]
```

**提示：**

```
1 <= values <= 10000
最多会对 appendTail、deleteHead 进行 10000 次调用
```

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof



## 题解

```js
var CQueue = function() {
    this.stack1 = [];
    this.stack2 = [];
};

/** 
 * @param {number} value
 * @return {void}
 */
CQueue.prototype.appendTail = function(value) {
    this.stack1.push(value);
};

/**
 * @return {number}
 */
CQueue.prototype.deleteHead = function() {
    if (this.stack2.length !== 0) return this.stack2.pop(); // 如果栈2不为空，则弹出
    else if (this.stack1.length === 0) { // 如果栈1和栈2不为空，返回-1
        return -1;
    } else { 
        while (this.stack1.length) { // 如果栈1不为空，将栈1所有元素弹出，push到栈1
            this.stack2.push(this.stack1.pop());
        }
        return this.stack2.pop(); // 弹出栈2
    }
};

/**
 * Your CQueue object will be instantiated and called as such:
 * var obj = new CQueue()
 * obj.appendTail(value)
 * var param_2 = obj.deleteHead()
 */
```

