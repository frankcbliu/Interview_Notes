# 2. 数组中重复的数字

## 题目描述

找出数组中重复的数字。


在一个长度为 `n` 的数组 `nums` 里的所有数字都在 `0～n-1` 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

输入：

```
[2, 3, 1, 0, 2, 5, 3]
```


输出：`2 或 3` 

限制：

```
2 <= n <= 100000
```



> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof



## 题解

### JAVA 版本

因为题目说明了所有数字在`0 ~ n-1`的范围内，且某些数字重复，那么我们可以这样考虑，从左到右遍历数组，将数组中的每个元素放到以它的值为索引的位置上，比如：

```
[1, 2, 3, 0, 1]
```

首先指向`arr[0]`，扫描到`1`，把`1`放到`arr[1]`上

```java
[1, 2, 3, 0, 1] // 交换 1 和 2
[2, 1, 3, 0, 1] // 交换 2 和 3
[3, 1, 2, 0, 1] // 交换 3 和 0
[0, 1, 2, 3, 1]
```

此时发现`arr[0] == 0`，继续往下；

`arr[1] == 1`、`arr[2] == 2`、`arr[3] == 3`；

直到发现`arr[4] != 4`，交换`arr[4] 和 arr[1]`，显然两者是一样的，数组仍然保持：

```java
[0, 1, 2, 3, 1]
```

再次判断是否重复，显然`arr[1] == arr[4]`，因此返回`arr[1]`，也就是其中一个重复值为`1`。



```java
// 交换 arr[i] 和 arr[j]
private void swap(int[] arr, int i, int j) {
    if (i != j) {
        arr[i] = arr[i] ^ arr[j];
        arr[j] = arr[i] ^ arr[j];
        arr[i] = arr[i] ^ arr[j];
    }
}

public int findRepeatNumber(int[] nums) {
    // 题目说明了必然存在重复的，这里就不考虑边界了
    for (int i = 0; i < nums.length; i++) {
        while (i != nums[i]) { // 不断交换直到当前 i == nums[i]，也就是出现重复
            swap(nums, i, nums[i]);
            if (nums[i] == nums[nums[i]]) {  // 判断是否重复
                return nums[i];
            }
        }
    }
    // 题目没说明不存在返回啥，我们返回一个 -1 代表不存在
    return -1;
}
```

![image-20200723005513945](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh08l0ecrtj30uo0a03zl.jpg)



### js 版本

遍历数组，并使用一个map存储对应值，如果map已经存在该值，说明重复，返回当前值。

```js

var findRepeatNumber = function(nums) {
    let map = new Map();
    for (let num of nums) {
        if (map.has(num)) return num;
        else map.set(num, 1);
    }
    return null;
};
```

