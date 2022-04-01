---
title: "leetcode-python-只出现一次的数字"
date: 2019-05-10T16:01:37+08:00
categories: ["历史文章"]
tags: ["python", "算法", "leetcode", "历史文章"]
draft: false
---

> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
> 说明：
> 你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？
> 示例 1:
> 输入: [2,2,1]
> 输出: 1
> 示例 2:
> 输入: [4,1,2,1,2]
> 输出: 4



题目要求不使用额外的空间来实现，所以使用位运算符实现，关于位运算符：
![image.png](https://i.loli.net/2019/08/14/nHARBmt2qMhawXG.png)
详细的请看菜鸟教程中的解释http://www.runoob.com/python/python-operators.html

代码如下

```python
def single_num(nums: list) ->int:
    a = 0
    for i in nums:
        a = a ^ i
    return a

if __name__ == '__main__':
    n = [2, 2, 1]
    print(single_num(n))
```

