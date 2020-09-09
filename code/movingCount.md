# 11.机器人的运动范围

## 题目描述

地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

**示例 1：**

输入：

```
m = 2, n = 3, k = 1
```


输出：

```
3
```

**示例 2：**

输入：

```
m = 3, n = 1, k = 0
```

输出：

```
1
```

**提示：**

```
1 <= n,m <= 100
0 <= k <= 20
```



> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof

## 题解

```js
/**
 * @param {number} m
 * @param {number} n
 * @param {number} k
 * @return {number}
 */
var movingCount = function(m, n, k) {
    if (!k) return 1;
    let getDigitsSum = function(x) { // 获取位数之和
        let sum = 0;
        while (x) {
            sum += x%10;
            x = Math.floor(x/10);
        }
        return sum;
    };
    let visited = new Map(); // 记录是否被访问过
    let _movingCount = function(i, j) {
        if (i >= m || j >= n || 
            visited.get(i + ',' + j) === true ||  // 已经访问过
            (getDigitsSum(i) + getDigitsSum(j)) > k) // 位数和大于k
            return 0;
        visited.set(i + ',' + j, true);
        return 1 + _movingCount(i+1, j) +  // 向右搜索
                _movingCount(i, j+1);   // 向下搜索
    };
    return _movingCount(0, 0);
};
```

