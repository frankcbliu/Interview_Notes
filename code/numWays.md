# 8.青蛙跳台阶问题

## 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

输入：

```
n = 2
```

输出：

```
2
```

**示例 2：**

输入：

```
n = 7
```

输出：

```
21
```

**提示：**

```
0 <= n <= 100
```

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof



## 题解

```js
/**
 * @param {number} n
 * @return {number}
 */
var numWays = function(n) {
    let cache = new Map(); // 使用map做缓存，避免重复计算
    cache.set(0, 1);
    cache.set(1, 1);
    let _numWays = function(n) {
        if (cache.has(n)) return cache.get(n);
        let res = _numWays(n-1) + _numWays(n-2);
        res = res > 1000000007 ? res % 1000000007 : res;
        cache.set(n, res);
        return res;
    }
    return _numWays(n);
};
```

