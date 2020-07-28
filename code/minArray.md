# 9.旋转数组的最小数字

## 题目描述

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个**递增排序**的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

**示例 1：**

输入：

```
[3,4,5,1,2]
```

输出：

```
1
```

**示例 2：**

输入：

```
[2,2,2,0,1]
```


输出：

```
0
```



> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof



## 题解

```js
/** 暴力法
 * @param {number[]} numbers
 * @return {number}
 */
var minArray = function(numbers) {
    if (numbers.length === 0) return null;
    if (numbers.length === 1) return numbers[0];
    for (let i = 1; i < numbers.length; i++) {
        if (numbers[i] < numbers[i-1]) return numbers[i];
    }
    return numbers[0];
};

/** 二分法
 * @param {number[]} numbers
 * @return {number}
 */
var minArray = function(numbers) {
    if (numbers.length === 0) return null;
    if (numbers.length === 1) return numbers[0];
    let low = 0, high = numbers.length - 1;
    while (low < high) {
        let mid = low + Math.floor((high - low)/2);
        if (numbers[mid] < numbers[high]) {
            high = mid;
        } else if (numbers[mid] > numbers[high]) {
            low = mid + 1;
        } else {
            high--;
        }
    }
    return numbers[low];
};
```

