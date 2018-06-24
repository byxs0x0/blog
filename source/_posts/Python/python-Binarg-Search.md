---
title: SQL盲注所探究的Python二分法
date: 2018-06-14 20:13:04
categories: Python
tags:
description:
---

马丹，感觉自己以前学的二分法是假的！！！
重新来好好玩一下！！！
<!-- more -->
>二分搜索（英语：binary search），也称折半搜索、对数搜索，是一种在**有序数组**中查找某一特定元素的搜索算法。

<br>
下面是**查询某字母在字符表中python代码**
Python也可以用字符切片的方式来实现

<br>
### while形式
```python
# coding:utf-8
import sys

ARR = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v',
       'w', 'x', 'y', 'z']
ARR_LEN = len(ARR) - 1 # 由于数组从0开始，所以需要-1

def BinSearch(str):
    left = 0
    right = ARR_LEN
    '''
    left与right比较必须用<=号，而不是<号，原因是mid的计算在循环内。
    例如在[4,5]中搜索5。进入循环，计算mid值后(left,mid,right)分别为(0,0,1)
    传入4>ARR[0]，所以left=1，然后重新循环，这是(left,mid,right)分别为(1,0,1)
    如果while条件是left<right，那么1<1，不成立，不进入循环，最后的结果为4。所以必须为<=
    '''
    while left <= right:
        mid = (left + right) >> 1 # 平均数向下取整
        if str > ARR[mid]:
            # 传入值大于中间值，中间值向左区间舍去，同时中间值向后一位
            left = mid + 1
        elif str < ARR[mid]:
            # 传入值小于中间值，中间值向右区间舍去，同时中间值向前一位
            right = mid - 1
        else:
            # 传入值等于中间值，直接停止循环
            break
    print('%s在字母表中的位置是第 %d 位'%(str, mid+1))

if __name__ == "__main__":
    BinSearch(sys.argv[1])

```

<br>
### 递归形式
如果上面的代码能理解，那么递归形式非常的好理解
```python
# coding:utf-8
import sys

ARR = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v',
       'w', 'x', 'y', 'z']

def BinSearch(left, right, str):
    mid = (left + right) >> 1
    if str > ARR[mid]:
        return BinSearch(mid+1, right, str)
    elif str < ARR[mid]:
        return BinSearch(left, right-1, str)
    return mid + 1

if __name__ == "__main__":
    str = sys.argv[1]
    left = 0
    right = len(ARR) - 1
    result = BinSearch(left, right, str)
    print('%s在字母表中的位置是第 %d 位' % (str, result))
```

<br>
### 查询字符不在数组中的情况
**上面的前提都是字符在数组中的前提，那么字符不再数组会怎么样呢？**

首先，如果字符不再数组中，在倒数第二次循环的结果必然会变为`left=mid=right`(不清楚可以调试一下)
在最后一次循环时由于传入值与`ARR[mid]`值不可能相等，那么无论大于或者小于，都会造成`left>right`
最后便会造成答案混乱
所以我们可以在判断的时候加入一种情况,**当left>right时，提示字符不再其中**


<br>
### SQL盲注所使用的二分法
在前两种二分法写法中都判断了，当传入值与中间值相等的情况。
但是我们在盲注的时候使用二分法，我们的SQL语句都是以**数据值>有序数组中间值**的情况下来判断。
(也可以改变判断条件，但是在循环中会发送多次请求包)

例如`SELECT * FROM users WHERER id = 1 AND ((MID(DATABASE(), 1, 1)) > 's')`,在上述语句中
 - 大于的情况下，页面会返回`id=1`的数据内容
 - 小于和等于的情况下，页面都没有数据内容

所以我们需要改变原有的二分法结构，在只根据两种情况下也能得到最后的答案
```python
# coding:utf-8

ARR = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v',
       'w', 'x', 'y', 'z']
ARR_LEN = len(ARR) - 1

def BinSearch(str):
    left = 0
    right = ARR_LEN
    mid = (left + right) >> 1
    while left < right:
        if str > ARR[mid]:
            # 大于，取后半段
            left = mid + 1
        else:
            right = mid
        mid = (left + right) >> 1
    print('%s在字母表中的位置是第 %d 位'%(str, mid+1))

if __name__ == "__main__":
    for i in ARR:
        BinSearch(i)
```
根据while的二分法改写
 - 先看判断条件部分，如果大于成立，那么必定就比原来的数大，所以还是`left=mid+1`，小于和相等的情况下是相同的，所以我们只能把值取到`right=mid`
 - 在看while的判断条件，原本是`<=`，这边必须改为`<`，因为改写了条件语句，最后的值必然是3个数都相同，如果是`<=`，那么会死循环
 - 如果循环条件变为`<`，当`left=right`的情况时，由于mid值在循环内，跳出了循环，并且mid值没有被重新计算(可参照while循环写法的注释)，那么最后答案就会错误
 - 所以我们在循环之前先计算一次mid值，并且在每个循环末尾在计算mid值，便补缺了`left=right`情况下少的那一次


只要值是在区间内，无论怎么循环，值都会在区间中，并且最后一次循环必然`left=mid=right`，也就是只剩一个值
所以最后那个值必然是答案。


### SQL盲注之Order By 注入
这是我在[Injection](http://byxs0x0.cn/2018/06/12/Write%20Up/ah-Web-Write-Up/)这道题目中碰见的一种情况
其实就是大于和等于的情况是情况1，而小于的情况是情况2

```python
# coding:utf-8
import math

ARR = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v',
       'w', 'x', 'y', 'z']
ARR_LEN = len(ARR) - 1

def BinSearch(str):
    left = 0
    right = ARR_LEN
    mid = int(math.ceil((left + right) / 2.0))
    while left < right:
        if str < ARR[mid]:
            # 大于，取后半段
            right = mid - 1
        else:
            left = mid
        mid = int(math.ceil((left + right) / 2.0))
    print('%s在字母表中的位置是第 %d 位'%(str, mid+1))

if __name__ == "__main__":
    for i in ARR:
        BinSearch(i)
```
在改写的时候，需要注意的就是**向上取整**

<br>
### 总结
自己以前到底是怎么理解的，居然还活着用过来了
