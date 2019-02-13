---
layout: post
title: "Python-排序-选择排序-优化"
date: 2018-12-12 21:38:57
comments: true
reward: true
tags: 
	- Python
	- 选择排序
---

>这是我通过极客专栏《数据结构与算法之美》学习后的思考，分享一下，希望对你有所帮助。上一篇文章 [工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/) 的思维导图展现了这个专栏的内容。

选择排序的思想：将一组数据分为两部分，前面是已排序部分，后面是未排序部分，初始状态可认为位置 0 为已排序部分 (数组下标从0开始)，其余为未排序部分，每一次都从未排序部分选择一个最小元素放在已排序部分的末尾，然后已排序部分增加一个元素，未排序部分减少一个元素，直到数据全部有序。

<!-- more -->

算法的过程：
1、第一次从 1 到 n-1 个元素中选择一个最小数据与位置 0 的数据交换
2、第二次从 2 到 n -1 个元素中选择一个最小数据与位置 1 的数据交换
3、第三次从 3 到 n -1 个元素中选择一个最小数据与位置 2 的数据交换
......

思路很简单，可是要真正动手写起来，会发现数组的下标如果处理不当，会有 BUG，而且调试起来也挺费时间，建议在写代码时加上注释，方便自己，更方便别人的理解。

## 无优化版：
```python
def selection_sort(data_list):
    count = 0 
    length = len(data_list)
    for i in range(length -1):
        print(data_list) #打印每一次选择后的结果
        min_index = i #存储最小值的下标，以便最后交换
        for j in range(i+1,length):
            count +=1
            if data_list[min_index] > data_list[j]:
                min_index = j
        if min_index != i: #说明需要交换，否则不需要自己自己交换 
            tmp = data_list[i]
            data_list[i] = data_list[min_index]
            data_list[min_index] = tmp
    print(f"总循环次数为 {count}")
    return data_list
```
现在执行一下，看下代码的运行效果

```python
unsort = [3,4,2,1,5,6,7,8]
print("selection_sort 的结果",*selection_sort(unsort))
```
运行结果如下：
```python
[3, 4, 2, 1, 5, 6, 7, 8]
[1, 4, 2, 3, 5, 6, 7, 8]
[1, 2, 4, 3, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
总循环次数为 28
selection_sort 的结果 1 2 3 4 5 6 7 8
```
虽然得到了正确的结果，但是对于 [3, 4, 2, 1, 5, 6, 7, 8] 初始数据，已经基本有序，但是竟然循环了 28 次，那么有没有可以优化的地方呢？

细想一下，每次扫描未排序区间只选取一个最小值，那么是否可以每次扫描时选择一个最小元素和一个最大元素，分别放置在有序区间的尾部和尾部有序区间的头部呢？ 

当然是可以的，在循环选取时，当最小元素的位置+1 = 最大元素的位置时，数据已经全部有序，可以退出。下面是优化版的代码

## 优化版
```python
def selection_sort2(data_list):
    count = 0 
    length = len(data_list)
    for i in range(length -1):
        print(data_list) #打印每一次选择后的结果
        min_index = i #存储最小值的下标 
        max_index = length - i -1 #最大值的下标，以便最后交换

        for j in range(i+1,length - i -1):
            count +=1
            if data_list[min_index] > data_list[j]:
                min_index = j
            if data_list[max_index] < data_list[j]:
                max_index = j
            
            #退出条件
        if min_index +1 == max_index:
            break

        #前面的数据与最小值交换
        if min_index != i: #说明需要交换，否则不需要自己自己交换 
            tmp = data_list[i]
            data_list[i] = data_list[min_index]
            data_list[min_index] = tmp

        #后面的数据与最大值交换
        if max_index != length - i -1: #说明需要交换，否则不需要自己与自己交换 
            tmp = data_list[length - i -1 ]
            data_list[length -i -1 ] = data_list[max_index]
            data_list[max_index] = tmp


    print(f"总循环次数为 {count}")
    return data_list
```
同样地执行一下，和未优化的对比总循环次数和选择次数
```python
unsort = [3,4,2,1,5,6,7,8]
print("selection_sort2 的结果",*selection_sort2(unsort)
```
执行结果如下：

```python
[3, 4, 2, 1, 5, 6, 7, 8]
[1, 4, 2, 3, 5, 6, 7, 8]
[1, 2, 4, 3, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
总循环次数为 12
selection_sort2 的结果 1 2 3 4 5 6 7 8
```
从结果可以看出比未优化版的总循环次数少了 一倍，而且选择次数为 4 ，也有明显减少。在实际应用中，当数据量很大时，优化的结果还是很可观的。

## 性能分析

首先，选择排序的只需要一个变量做为交换，因此空间复杂度是O(1)，是一种原地排序算法。
其次，选择排序在未排序区间选择一个最小值，与前面的元素交换，对于值相同的元素，因为交换会破坏他们的相对公交车，因此它是一种不稳定的排序算法。

比如对于 4，1，4，2，5，这样的序列，
第一次选择后是这样的 ：1，4，4， 2， 5 ，此时先后的顺序未变，第二次选择后是这样的：1，2，4，4，5 ，需要拿第一个 4 与 2 交换，因此两个 4 的相对顺序已经变化，因此选择排序是一种不稳定的排序算法。

选择排序无论数据初始是何种状态，均需要在未排序元素中选择最小或最大元素与未排序序列中的首尾元素交换，因此它的最好、最坏、平均时间复杂度均为 O(n^2)。

想看一下选择排序执行过程的动态图吗，请访问以下链接：[选择排序的动态图](https://visualgo.net/en/sorting?slide=7)
（完）

![个人公众号](/assets/img/wechat.jpg)


