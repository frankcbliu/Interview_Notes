# 3. 替换空格

## 题目描述


请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

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

0 <= s 的长度 <= 10000

> 来源：力扣（LeetCode）
>
> 链接：https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/

## 题解

```js
/** 方法一
 *	对空格进行分割，再用%20拼接
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

