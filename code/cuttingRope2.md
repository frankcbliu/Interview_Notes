# 13.剪绳子II

## 题目描述

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m - 1] 。请问 k[0]*k[1]*...*k[m - 1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

输入:`2`

输出: `1`

解释: `2 = 1 + 1, 1 × 1 = 1`

**示例 2:**

输入: `10`

输出: `36`

解释: `10 = 3 + 3 + 4, 3 × 3 × 4 = 36`

**提示：**`2 <= n <= 1000`

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof

## 题解

```js
/**
 * @param {number} n
 * @return {number}
 */
var cuttingRope = function(n) {
    if (n <= 3) return n-1;
    let bigPow = function(x, y) {
        let res = 1;
        for (let i = 0; i < y; i++) {
            res = (res*x) % 1000000007;
        }
        return res;
    };
    let count = Math.floor(n/3); // 能切成3的个数
    let rem = n%3; // 余数
    let res = null;
    if (rem === 0) {
        res = bigPow(3, count);
    } else if (rem === 1) {
        res = bigPow(3, count-1) * 4;
    } else {
        res = bigPow(3, count) * 2;
    }
    return res % 1000000007;
};
```

