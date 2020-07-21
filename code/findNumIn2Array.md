# 1. 二维数组中的查找

## 题目描述

在一个 `n * m` 的二维数组中，每一行都按照**从左到右递增**的顺序排序，每一列都按照**从上到下递增**的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例:

现有矩阵 `matrix` 如下：

```
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
```


给定 `target = 5`，返回 `true`。

给定 `target = 20`，返回 `false`。

 

限制：

```
0 <= n <= 1000
0 <= m <= 1000
```



> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof

## 题解

本题考虑对角线的方式走，可以获得最优复杂度`O(M+N)`；下面的代码采用从左下到右上的方式去判断，因为题目中给出的数组是有规律可循的，从左往右递增，从上到下递增。

那么当我们此时的值比目标值**大**时，我们需要往**更小**的地方（向左或者向上）走，而我们又是从左下角开始的，因此只能向上走；

当我们比目标值小时，我们需要往更大的地方走才能找到目标值，此时只能往右走。

最终到达右上角仍未找到目标值，则说明目标值不存在，如果中间超出上边界或者右边界，也说明目标值不存在。

### java 版本

```java
public boolean findNumIn2Array(int[][] matrix, int target) {
  	// 判断边界
    if (matrix == null || matrix.length < 1 || matrix[0] == null || matrix[0].length < 1)
        return false;

    int m = matrix.length;      // 行数
    int n = matrix[0].length;   // 列数

    // 从左下向右上遍历
    int i = m - 1;
    int j = 0;
    while (i >= 0 && j < n) {
        if (matrix[i][j] > target) { // 当前值比 target 大，说明 target 在上方
            i--;
        } else if (matrix[i][j] < target) { // 小则说明 target 在右边
            j++;
        } else { // 相等说明找到了
            return true;
        }
    }
    return false;
}
```



###  js 版本

```js
var findNumberIn2DArray = function(matrix, target) {
    if (!matrix || !matrix.length || !matrix[0] || !matrix[0].length) return false;
    let m = matrix.length;      // 行数
    let n = matrix[0].length;   // 列数
    let i = m - 1, j = 0;       // 从左下开始查询
    while (i >= 0 && j < n) {
         if (matrix[i][j] > target) { // 当前值大于目标值，说明目标值在上方
            i--;
        } else if(matrix[i][j] < target) { // 当前值小于目标值，说明目标值在右方
            j++;
        } else {
            return true;
        }
    }
    return false;
}
```





