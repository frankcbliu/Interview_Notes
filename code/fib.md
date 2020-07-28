# 7.斐波那契数列

## 题目描述

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

```
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```


斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例 1：**

输入：

```
n = 2
```

输出：

```
1
```

**示例 2：**

输入：

```
n = 5
```

输出：

```
5
```

**提示：**

```
0 <= n <= 100
```

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof



## 题解

```js
/**
 * @param {number} n
 * @return {number}
 */
var fib = function(n) {
    let cache = new Map(); // 使用map做缓存，避免重复计算
    cache.set(0, 0);
    cache.set(1, 1);
    let _fib = function(n) {
        if (cache.has(n)) return cache.get(n);
        let res = _fib(n-1) + _fib(n-2);
        res = res > 1000000007 ? res % 1000000007 : res;
        cache.set(n, res);
        return res;
    }
    let res = _fib(n);
    return res;
};
```

