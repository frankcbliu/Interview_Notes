# 15. 数值的整数次方

## 题目描述

实现函数`double Power(double base, int exponent)`，求`base`的`exponent`次方。不得使用库函数，同时不需要考虑大数问题。

**示例 1:**

输入: `2.00000, 10`

输出: `1024.00000`

**示例 2:**

输入: `2.10000, 3`

输出: `9.26100`

**示例 3:**

输入: `2.00000, -2`

输出: `0.25000`

解释: `2-2 = 1/22 = 1/4 = 0.25`

**说明:**

```
-100.0 < x < 100.0
n 是 32 位有符号整数，其数值范围是 [−231, 231 − 1] 。
```

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof

## 题解

```js
/**
 * @param {number} x
 * @param {number} n
 * @return {number}
 */
var myPow = function(x, n) {
    if (x === 0) return 0;
    let res = 1;
    if (n < 0) {
        x = 1/x;
        n = -n;
    }
    while (n) {
        if (n & 1) res *= x;
        x *= x;
        n = n/2; // 相当于右移一位
    }
    return res
};
```

