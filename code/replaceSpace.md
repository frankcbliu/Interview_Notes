# 3. 替换空格

## 题目描述


请实现一个函数，把字符串 `s` 中的每个`空格`替换成`"%20"`。

示例 1：

输入：

```text
s = "We are happy."
```

输出：

```text
"We%20are%20happy."
```

限制：

```
0 <= s 的长度 <= 10000
```



> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/

## 题解

### JAVA版本

这题解决办法很多，这里尽量不使用正则来做，给`C++`同学也可以参考。

```java
public String replaceSpace(String s) {
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < s.length(); i++) {
        if (s.charAt(i) == ' ') {
            sb.append("%20");
        } else {
            sb.append(s.charAt(i));
        }
    }
    return sb.toString();
}
```

如果题目提供的是字符数组，那么需要先遍历一遍确认空格数，以便确定新数组大小，然后从后往前遍历即可，遇到空格反向填充为`02%`即可。

![image-20200723005403311](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh08js8fpfj30wm09cq41.jpg)

### js 版本

```js
/** 方法一
 *	对空格进行分割，再用 %20 拼接
 */
var replaceSpace = function(s) {
    return s.split(' ').join('%20');
};

/** 方法二
 *	正则匹配替换
 */
var replaceSpace = function(s) {
    return s.replace(/\s/g, '%20')
};

/** 方法三
 *	遍历字符串判断
 */
var replaceSpace = function(s) {
    let res = '';
    for (let i of s) {
        if (i === ' ') res += '%20';
        else res += i;
    }
    return res;
};
```

