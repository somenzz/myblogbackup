---
layout: post
title: "Python-排序-冒泡排序-优化"
date: 2018-12-10 23:38:57
comments: true
reward: true
tags: 
	- Python
	- 冒泡排序
---

>这是我通过极客专栏《数据结构与算法之美》学习后的思考，分享一下，希望对你有所帮助。上一篇文章 [工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/) 的思维导图展现了这个专栏的内容。
<!-- more -->

说到算法中的排序，冒泡排序是最简单的一种排序算法了，甚至不学数据结构与算法的同学都会使用它。但是你有没有想过可以怎么优化？

什么是冒泡排序：就像水慢慢烧开，气泡从下拄上越来越大那样，第一次循环都把n个元素中最大的元素移动至最后位置，第二次从前 n-1 个位置中找出最大元素放在最后，重复执行，直到最后结果全部有序。


## 最基本的算法实现，无优化版：

```python
  def bubble_sort(collection):
    """
    无任何优化版
    """
    compare_count=0
    length = len(collection)
    for i in range(length-1):
        print(collection) #方便查看数组的排序过程
        for j in range(length-1-i):
            compare_count+=1
            if collection[j] > collection[j+1]:
                tmp = collection[j]
                collection[j] = collection[j+1]
                collection[j+1] = tmp
    print(f"总循环次数{compare_count}")
    return collection
```

下面来执行一下，看看执行的过程，及总循环次数：

```python
    print("bubble_sort begin.")
    unsorted = [3,4,2,1,5,6,7,8]
    print("bubble_sort end: ",*bubble_sort(unsorted))
```

执行结果如下：

```python
bubble_sort begin.
[3, 4, 2, 1, 5, 6, 7, 8]
[3, 2, 1, 4, 5, 6, 7, 8]
[2, 1, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
总循环次数28
bubble_sort end:  1 2 3 4 5 6 7 8
```

通过排序的过程可以发现，在第 4 次冒泡时，数据已经有序，因此可以加入判断，如果本次循环没有冒泡（交换），说明数据已经有序，可以直接退出，优化后的代码如下：

## 优化一

```python
def bubble_sort2(collection):
    """
    如果没有元素交换，说明数据在排序过程中已经有序，直接退出循环
    """
    compare_count=0
    length = len(collection)
    for i in range(length-1):
        swapped = False
        print(collection)
        for j in range(length-1-i):
            compare_count+=1
            if collection[j] > collection[j+1]:
                swapped = True
                tmp = collection[j]
                collection[j] = collection[j+1]
                collection[j+1] = tmp
        if not swapped: break  # Stop iteration if the collection is sorted.
    print(f"总循环次数{compare_count}")
    return collection
```

下面来执行一下，看看执行的过程，及总循环次数：

```python
    print("bubble_sort2 begin.")
    unsorted = [3,4,2,1,5,6,7,8]
    print("bubble_sort2 end:",*bubble_sort2(unsorted))
```

执行结果如下：

```python
bubble_sort2 begin.
[3, 4, 2, 1, 5, 6, 7, 8]
[3, 2, 1, 4, 5, 6, 7, 8]
[2, 1, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
总循环次数22
bubble_sort2 end: 1 2 3 4 5 6 7 8
```

至此，还有没有其他优化方法呢？ 聪明的你可能看到了，总循环次数是比较多的，仅比未优化版少了 6 次循环次数。有没有办法减少总循环次数呢？

观察数据可以发现，数据已经初始有序，可以分为两部分，无序部分 3 4 2 1 和有序部分 5 6 7 8 ，每次循环如果能够发现无序和有序的边界，然后下次冒泡仅对无序部分进行比较和冒泡，可大大减少比较次数（循环次数），从而加快速度。

问题是，怎么发现这个边界呢？ 
第一次冒泡的过程中，第一个元素 4 被移动到下标为【3】的位置（python 列表索引从 0 开始），位置 【3】就是有序部分的开始位置。

第二次冒泡的过程中，第一个元素 3 被移动到下标为【2】的位置（python 列表索引从 0 开始），位置 【2】就是有序部分的开始位置。
...
可以推断出，一次冒泡的过程中，最后一个被交换的元素下标即为无序和有序的边界，因而下次冒泡，仅对 0 ~ 边界 的元素冒泡即可大大减少循环次数。

## 优化二：

```python
def bubble_sort3(collection):
    """
    bubble_sort2的基础上再优化。
    优化思路：在排序的过程中，数据可以从中间分为两段，一段是无序状态，另一段是有序状态。
    每一次循环的过程中，记录最后一个交换元素的公交车，它便是有序和无序状态的边界
    下一次仅循环到边界即可，从而减少循环次数，达到优化。
    """
    compare_count=0
    length = len(collection)
    last_change_index = 0 #最后一个交换的位置
    border = length-1 #有序和无序的分界线
    for i in range(length-1):
        swapped = False
        print(collection)
        
        for j in range(0,border):
            compare_count+=1
            if collection[j] > collection[j+1]:
                swapped = True
                collection[j], collection[j+1] = collection[j+1], collection[j]
                last_change_index = j
        if not swapped: break  # Stop iteration if the collection is sorted.

        border = last_change_index # 最后一个交换的位置就是边界

    print(f"总循环次数{compare_count}")
    return collection
```

下面来执行一下，看看执行的过程，及总循环次数：

```python
    print("bubble_sort3 begin.")
    unsorted = [3,4,2,1,5,6,7,8]
    print("bubble_sort3 end:",*bubble_sort3(unsorted))
```

执行结果如下：

```python
bubble_sort3 begin.
[3, 4, 2, 1, 5, 6, 7, 8]
[3, 2, 1, 4, 5, 6, 7, 8]
[2, 1, 3, 4, 5, 6, 7, 8]
[1, 2, 3, 4, 5, 6, 7, 8]
总循环次数10
bubble_sort3 end: 1 2 3 4 5 6 7 8
```

可以看到结果的总循环次数为 10 ，与第二版相比，循环次数减少了一倍。

## 冒泡排序算法的性能分析：

#### 1、执行效率
最小时间复杂度：很好计算，最好的情况就是数据一开始就是有序的，因此一次冒泡即可完成，时间复杂度为 O(n)

最大时间复杂度：也很好计算，最坏的情况就是数据一开始就是倒序的，因此进行 n-1 次冒泡即可完成，时间复杂度为 O(n^2)

平均时间复杂度，严格来说平均时间复杂度就是加权平均期望时间复杂度，分析的时候要结合概率认的知识，对于包含 n 个数据的数组，有 n! 种排序方式，不同的排列方式，冒泡排序的执行时间肯定是不同的，如果要用概率认的方法定量分析平均时间复杂度，涉及的数据推理会很复杂，这里有一种思路，通过**有序度**和**逆序度**这两个概念来分析。有序度就是有顺序的元素的个数，比如 3，1 ，2 这三个数据有有序度为1 即 （1，2） 一个，相反，逆序度为 2，即（3，2）（3，1）这两个， 1， 2， 3 这三个数据的有序度为 3：（1，2）（1，3）（2，3），逆序度为 0，完全有序的数据序列的有序度也叫满有序度。
有个公式

```sh
逆序度 = 满有序度 - 有序度
```

排序的过程就是增加有序度，减少逆序度，最后达到满有序度，说明排序完成。逆序度也主浊元素的交换次数，最坏情况，初始状态的有序度为 0 ，逆序度为 n*(n-1)/2 , 所以要进行 n*(n-1)/2 次交换操作，最好情况，补充状态完全有序，逆序度为 0 不需要进行交换，这里平均交换次数我们可以取个平均值即 n\*(n-1)/4。

而比较次数肯定比交换次数要多，因而平均情况下，无论算法怎么优化，时间复杂度不会低于 n\*(n-1)/4，也就是 O(n^2)。

## 内存消耗
算法的内存消耗可以通过空间复杂度来衡量，冒泡排序仅需要一个变量
 tmp 来储存交换的数据，因此空间复杂度为 O(1)，空间复杂度为 O(1) 的排序算法，也叫 **原地排序算法**

## 排序算法的稳定性
针对排序算法，有一个重要的衡量指标，就是稳定性，这个概念是说，如果待排序的序列中存在值相等的元素，经过排序之后，相等元素之间原有的先后顺序不变。假如有序列 4，1，2，2，我们把第一个 2 叫 2'，第二个2 叫 2''，如果排序之后，为1，2'，2''，4 那么这个排序算法就是稳定的，否则就是不稳定的。稳不稳定有什么用吗，值都是一样的？当然有用，因为在软件开发中，要排序的数据不单单是一个属性的数据，而是有多个属性的对象，假如对订单排序，要求金额排序，订单金额相同的情况下，按时间排序。最先想到的方法就是先对金额排序，在金额相同的订单区间内按时间排序，理解起来不难，有没有想过，实现起来很复杂。

但是借助稳定的排序算法，就很简单了，先按订单时间排一次序，再按金额排一次序就可以了。

## 小结
对排序算法的分析无外乎时间复杂度（最好，最坏，平均），空间复杂度，稳定性这些方面，只要理解其思路，弄明白其适用场景，不需要死记。优化思路可以通过观察分析得出，还有一点，冒泡排序虽然使用了数组存储数据但是并没有使用数组随机访问的特性，因此改用链表这种存储结构，使用冒泡排序仍然是可以实现的，你可以尝试下。

关注个人微信公众号 somenzz 与你一起学习。
![0](https://upload-images.jianshu.io/upload_images/12989993-71bd27e3ea0f1770.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

