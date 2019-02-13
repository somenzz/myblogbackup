---
layout: post
title: "Python-排序-快速排序，如何在O(n)内找到第K大元素？"
date: 2018-12-18 16:38:57
comments: true
reward: true
tags: 
	- Python
	- 快速排序
---

>以下是本人学习极客时间的专栏《数据结构与算法之美》后，自己动手敲代码实现，并写下当时的思考，希望对你也有帮助。
系列文章：
[工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/)
[Python-排序-冒泡排序-优化](https://somenzz.github.io/2018/12/10/python_bubble_sort/)
[Python-排序-选择排序-优化](https://somenzz.github.io/2018/12/12/python_select_sort/)
[Python-排序-插入排序-优化](https://somenzz.github.io/2018/12/15/python_insert_shell_sort/)
[Python-排序-归并排序-哨兵的妙用](https://somenzz.github.io/2018/12/17/python_merge_sort/)

<!-- more -->

王争老师讲过，学习算法不是死记硬背一些源代码或概念，而是学习算法的实现思路、思维、应用场景，从而达到灵活运用。

比如现在要时间复杂度为 O(n)，在一个长度为 n 的数组中查找到第 K 大的元素，你会怎么做呢？ 你可能会说这很简单啊，第一次遍历数组找到第 1 大元素，第二次遍历找到第 2 大，...，第 K 次就可以找到第 K 大 但是这样的时间复杂度就不是 O(n),而是 K\*O(n),当 K 接近 n 时，时间复杂度就是 O(n^2)。 如果你运用快速排序算法的思想，你就可以在 O(n) 的时间复杂度内找到第 K 大元素。

## 快速排序算法

快速排序算法和归并排序算法一样，都是利用分治算法。但是它们却有本质的不同，归并排序是自下而上，先求解下面的子问题求，然后再逐层归并，最后全部有序；而快速排序是自上而下，下面的子问题解决后，数据就全部有序。

快速排序的思路是这样的，在数组中随机选取一个数据，例如选取最后一个元素 m 做为分区元素，比 m 小的放 m 的左边，反之放右边，再分别对左右边的分区再分别进行分区，直到分区元素缩小到 1 个，此时数据已经全部有序。
下面是我根据理解编写的快速排序代码（python语言）

```python
#encoding=utf-8
import random

def quick_sort(data_list):
    length = len(data_list)
    quick_sort_c(data_list,0,length)

def quick_sort_c(data_list,begin,end):
    """
    可以递归的函数调用
    """
    if begin == end:
        return
    else:
        #获取分区数据partition_data最后的下标
        index = partition(data_list,begin,end)
        print(data_list)
        quick_sort_c(data_list,begin,index)
        quick_sort_c(data_list,index+1,end)

def partition(data_list,begin,end):
    #选择最后一个元素作为分区键
    partition_key = data_list[end-1]

    #index为分区键的最终位置
    index= begin  
    for i in range(begin,end-1):
        if data_list[i] < partition_key:
            data_list[i],data_list[index] = data_list[index],data_list[i]  #交换
            index+=1
    
    data_list[index],data_list[end-1] = data_list[end-1],data_list[index] #交换
    return index

if __name__ == "__main__":
    data_list = [random.randint(0,100) for i in range(10)]
    # data_list = [25, 77, 52, 49, 85, 28, 1, 28, 100, 36]
    print("原始数组:", data_list)
    print("排序过程如下")
    quick_sort(data_list)
    print("最终结果:",data_list)
```

执行结果如下所示：

```python
原始数组: [66, 1, 10, 95, 87, 16, 14, 88, 87, 82]
排序过程如下
[66, 1, 10, 16, 14, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 88, 87, 95]
[1, 10, 14, 16, 66, 82, 87, 87, 88, 95]
[1, 10, 14, 16, 66, 82, 87, 87, 88, 95]
最终结果: [1, 10, 14, 16, 66, 82, 87, 87, 88, 95]
```

## 性能分析

快速排序是一种原地排序算法，不需要借助额外的存储空间；由于分区的过程中由于其他元素的影响，在交换位置时会破坏原有的先后顺序，比如 3，5，6，3，2 在第一次分区后，两个 3 的相对次序已经改变，因此快速排序是一种不稳定的排序算法；时间复杂度为 O(nlogn)，但在极端情况下会降低到 O(n^2)，比如在数据已经是有序的情况时，需要进行 n 次分区，每次分区需要平均扫描 n/2 个元素，因此这种情况下时间复杂度为 O(n^2)。

## O(n)的时间内查找第 K 大元素的方法 

通过观察运行上面快速排序的过程可以发现，第一个分区键为 82，在第一次分区后，它是数组中的第 6 个元素，那么可以断定，82 就是第 6 小元素，或者 82 就是第 （10-6+1）=5 大元素，需要查找最 3 大元素，那么这个元素一定在第一次分区的右部分进行分区操作，求得分区键的下标 index = n - K = 10 -3 = 7 时返回分区键即是所求得的数据。下面我通过代码实现如下：

```python
def find_top_k(data_list,K):
    length = len(data_list)
    begin = 0
    end = length
    index = partition(data_list,begin,end) #这里的partition函数就是上面快排用到的函数
    while index != length - K:
        if index >length - K:
            index = partition(data_list,begin,index)
        else:
            index = partition(data_list,index+1,end)
    return data_list[index]
```

执行一下看看效果：

```python
data_list = [25, 77, 52, 49, 85, 28, 1, 28, 100, 36]
for i in [1,2,3,4,5]:
    print(f"第 {i} 大元素是 {find_top_k(data_list,i)}")
```
执行结果如下所示

```python
[25, 77, 52, 49, 85, 28, 1, 28, 100, 36]
第 1 大元素是 100
第 2 大元素是 85
第 3 大元素是 77
第 4 大元素是 52
第 5 大元素是 49
```
下面解释一下为什么时间复杂度是O(n):

第一次分区查找，我们需要对大小为 n 的数组执行分区操作，需要遍历 n 个元素。第二次分区查找，我们只需要对大小为 n/2 的数组执行分区操作，需要遍历 n/2 个元素。依次类推，分区遍历元素的个数分别为、n/2、n/4、n/8、n/16.……直到区间缩小为 1。 如果我们把每次分区遍历的元素个数加起来，就是：n+n/2+n/4+n/8+…+1。这是一个等比数列求和，最后的和等于 2n-1。所以，上述解决思路的时间复杂度就为 O(n)。

## 小结

快速排序和归并排序都是分治的思想，代码都通过递归来实现，归并排序的重点在于 merge 函数，而快排的重点在于 partition 函数。归并排序优点:任何情况下时间复杂度稳定在 O(nlogn),缺点：不是原地排序算法，需要额外的内存空间。快速排序是一种原地排序算法，平均时间复杂度为O(nlogn)，但极端情况时间复杂度会退化成O(n^2)，虽然这种情况的概率非常小，仍需要合理的选择分区键，避免左右分区极度不平衡。

（完）

加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat.jpg)
