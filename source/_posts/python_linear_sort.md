---
layout: post
title: "Python-排序-有哪些时间复杂度为O(n)的排序算法？"
date: 2018-12-25 20:38:57
comments: true
reward: true
tags: 
	- Python
	- 线性排序
---
![来燃烧你的卡路里](https://upload-images.jianshu.io/upload_images/12989993-75aea9218c42de80.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

人到中年，容易变得油腻，思想懒惰，身体就容易发胖。为了摆脱中年油腻，不如和我一起学习算法来烧烧脑子，燃烧你的卡路里。


<!-- more -->


**烧脑题目：如何在 O(n) 的时间复杂度内按年龄给 100 万用户信息排序？**

带着这个问题来学习下**三个线性排序**算法。

前几篇文章介绍了几个常用的排序算法：冒泡、选择、插入、归并、快速，他们的时间复杂度从 O(n^2) 到 O(nlogn)，其实还有时间复杂度为 O(n) 的排序算法，他们分别是桶排序，计数排序，基数排序，因为这些排序算法的时间复杂度是线性的，所以这类算法也叫线性排序。

你可能会问了，为什么这些时间复杂度低至 O(n) 的排序算法会很少使用呢？ 那就是因为这些排序算法对待排序的数据要求比较苛刻，这些算法理解其来比较简单，学习这类算法重要的是掌握它们的适用场景。下面我给出每一种算法的实现思路，Python程序实现和应用场景。

## 1、桶排序

桶排序，可以这样去理解：想像你面前有 m 个桶，有一堆待排序的 n 个数据，可以将这 n 个数据先按次序划分成 m 个区间，对应依次放入这 m 个桶里，然后对每个桶内的数据进行排序，再依次从 1 到 m 个桶里依次取出就得到有序的结果。

换成这样描述会更容易理解。假如要对 100 个订单的金额进行排序，订单的金额都是整数，订单金额从 1 到 100 不等，那么可以将这 100 个订单分成 10 个区间，放到 10 个桶中，1 至 10 元的放在 1 桶，11 至 20元的到 2 桶... 91 至 100 的放到第 10 个桶中，然后对每个桶内的数据进行快速排序，再依次从1 桶、2 桶、3 桶 、10 桶取出元素，就得到有序的订单信息。

你可能会问了，假如桶的个数是 m，每个桶中的数据量平均 n/m, 这个时间复杂度明明是 m\*(n/m)\*(log(n/m)) =  n log(n/m)，怎么可能是 O(n) 呢 ？ 这个问题非常好，原因是这样的，当桶的个数 m 接近与 n 时，log(n/m) 就是一个非常小的常数，在时间复杂度时常数是可以忽略的。比如极端情况下桶的个数和元素个数相等，即 n = m， 此时时间复杂度就可以认为是 O(n)。

#### 编程思路
```python
1、初始化桶的大小为K
2、获取 n 个数据中的最大值 max，最小值  min
3、将数据放入到 n/K +1 个桶中，a[i] 放入哪个桶的规则为  (a[i]-min)/K 
4、对 n/K 个桶分别进行快速排序并输出。
```
#### 算法实现 Python 版
```python
#encoding = utf-8
import random
from quick_sort import quick_sort

DEFAULT_BUCKET_SIZE = 5 

def bucket_sort(data_list,bucket_size = DEFAULT_BUCKET_SIZE):
    length = len(data_list)
    min = max = data_list[0]

    #寻找最小值和最大值
    for i in range(0,length):
        if data_list[i] < min:
            min = data_list[i]
        if data_list[i] > max:
            max = data_list[i]

    #定义多个桶并初始化
    num_of_buckets = (max - min) // bucket_size +1 
    buckets = []
    for i in range(num_of_buckets):
        buckets.append([])
    
    #将数据放入桶中
    for i in range(0,length):
        buckets[(data_list[i] - min)//bucket_size].append(data_list[i])
    
    #依次对桶内数据进行快速排序
    data_list.clear()

    for i in range(num_of_buckets):
        print(f"第{i}个桶排序前的内容是{buckets[i]}")
        quick_sort(buckets[i])
        # print(f"第{i}个桶排序后的内容是{buckets[i]}")
        for data in buckets[i]:
            data_list.append(data) 
    
if __name__ == "__main__":
    data_list = [random.randint(0,15) for _ in range(0,15)]
    print(data_list)
    bucket_sort(data_list)
    print(data_list)
```
执行结果如下所示：
```python
[13, 7, 12, 7, 8, 14, 3, 10, 1, 13, 0, 0, 4, 3, 5]
第0个桶排序前的内容是[3, 1, 0, 0, 4, 3]
第1个桶排序前的内容是[7, 7, 8, 5]
第2个桶排序前的内容是[13, 12, 14, 10, 13]
[0, 0, 1, 3, 3, 4, 5, 7, 7, 8, 10, 12, 13, 13, 14]
```
这里请注意：如果桶内的数据极不均匀，极端情况下，只有一个桶有数据，那么桶排序的时间复杂度就退化为 O(nlogn)。因此桶排序适合对每个区间内分布比较均匀的数据进行排序。

#### 桶排序适用场景
桶排序适合外部排序，外部排序就是数据在内存之外，比如磁盘上，数据量比较大，无法一次性读入内存。举个例子，假如老板给你一份 10 GB 大小的文件，是订单的交易明细数据，要求你按订单金额从大到小排序，而你的内存内有 4GB，实际可用内存只有 2 GB，那么此时就是桶排序发挥作用的时候了。
1. 将文件逐行读入内存（几乎每个编程语言都可以），扫描并记录最小值，最大值，假如最小值为 1 元，最大值为 10 万元，且都为整数，不是整数也没关系，可以先乘以 100 换成整数，排序后再除以 100 还原。

2. 设置初始的桶大小为 1000，也就是说每 1000 元做一个分区，放在一个桶内，这样就有 10万/1000 = 100 个桶。再次逐行读入文件，订单金额在 1 到 1000 元的放在第 1 个桶里，在 1001 到 2000 元的放第 2 个桶里，依次类推，这里的桶是个形象的比喻，其实就是对应磁盘上的文件，这里的放，其实就是逐行写入磁盘上的文件中，相当于将 10 GB的文件分隔成 100 个小文件。

3. 如果订单金额是均匀分布的，那么这 100 个小文件平均每个占内存 100 MB。第一次分区后如果小文件均小于可用内存大小，那么可以依次对这些小文件数据全部读入内存进行快速排序，排序完再写回磁盘，最后依次读取这些小文件并输出到一个大文件中，达到排序的效果。如果仍有小文件超过可用内在，那么再对该文件再次分区，直到可用内存能容下为止。


## 2、计数排序
计数排序是桶排序的一种特殊情况，当待排序的元素的最大值为 K 时，就把数据划分为 K 个桶。每个桶内的数据都是相等的，这样就节省了桶内排序的时间。

典型的应用场景就是：高考分数排名。高考成绩满分 750 分，就设置 751 个桶，对应 0，1，...750 的分数，只需要将数百万的考生按成绩放在每个桶中，再依次从每个桶中输出学生信息，就完成了排序。

那么，为什么叫计数排序呢？为方便描述，假如有 8 个考生，满分为 5 分，他们的成绩分别为 2，5，3，0，2，3，0，3。那么就设计 6 个桶，简单点,定义数组 B[6] 表示 6 个桶，每个桶存放该分数的考生个数，只需要遍历一下考生成绩，即可得到桶 B[6] = [2,0,2,3,0,1] ,B[3]=3 表示成绩为 3 的考生有 3 个，而成绩小于 3 的考生前面有 4 个，也就是说成绩为 3 的考生，应该排列在有序数组的   5，6，7 三个位置，如何得到其他分数所在的位置呢？

首先对 B[6] 累计求和，得到新的 B[6] = [2,2,4,7,7,8]，B[3]=7 就表示小于等于 3 分的考生个数为 7 个。
根据  B[6] = [2,2,4,7,7,8] 和初始序列 2，5，3，0，2，3，0，3，我们可以得到有序序列 [ 0,0,2,2,3,3,3,5 ] 。这里**使用另外一个数组来计数的实现方式非常巧秒**，如下所示：
```python
#encoding=utf-8
#实现极客专栏 数据结构与算法之美 第13节 线性排序中的计数排序算法
def counting_sort(data_list):
    length = len(data_list)

    #定义桶
    bucket = [] 
    max = data_list[0]
    for d in data_list:
        if d > max:
            max = d
    #初始化
    for i in range(max+1):
        bucket.append(0)

    #计数
    for i in range(length):
        bucket[data_list[i]] = bucket[data_list[i]] + 1
    
    ##累加
    for i in range(1,max+1):
        bucket[i] = bucket[i]+bucket[i-1]

    #计数排序,定义结果数组并初始化
    result = []
    for i in range(length):
        result.append(0)

    #从尾至头查找分数在result的插入位置，如果从头到尾的话就不是稳定的排序算法。
    for i in range(length-1,-1,-1):
        result[bucket[data_list[i]]-1] = data_list[i]
        bucket[data_list[i]] = bucket[data_list[i]] -1

    #将结果复制到原来的数组中，达到修改传入数组的效果
    for i in range(length):
        data_list[i] = result[i]


if __name__ == "__main__":
    data_list = [2,5,3,0,2,3,0,3]
    print(data_list)
    counting_sort(data_list)
    print(data_list)
    
```

执行结果如下所示：

```python
[2, 5, 3, 0, 2, 3, 0, 3]
[0, 0, 2, 2, 3, 3, 3, 5]
```
#### 计数排序适用场景

计数排序只能用在数据范围不大的场景中，如果数据范围 k 比要排序的数据 n 大很多，就不适合用计数排序了。而且，计数排序只能给非负整数排序，如果要排序的数据是其他类型的，要将其在不改变相对大小的情况下，转化为非负整数。

## 3、基数排序

我们再来看这样一个排序问题。假设我们有 10 万个手机号码，希望将这 10 万个手机号码从小到大排序，你有什么比较快速的排序方法呢？

如果直接用快排，时间复杂度是O(nlogn)，如果使用基数排序，时间复杂度为O(n)。

手机号码这类数据有个特点：定长，只要前面某一位大小确定，后面的位就不需要在一一比较。因此我们可以借助稳定的排序算法，先按照最后一位来排序手机号码，然后，再按照倒数第二位重新排序，以此类推，最后按照第一位重新排序。经过 11 次排序之后，手机号码就都有序了。

根据每一位来排序，我们利用上述桶排序或者计数排序，它们的时间复杂度可以做到 O(n)。如果要排序的数据有 k 位，那我们就需要 k 次桶排序或者计数排序，总的时间复杂度是 O(k\*n)。当 k 不大的时候，比如手机号码排序的例子，k 最大就是 11，所以基数排序的时间复杂度就近似于 O(n)。

这里给出我自己实现的代码（python）

```python
#encoding=utf-8
import random

class phone_num(object):

    num = ""

    def __init__(self,num=""):
        self.num = num
    
    def get_bit(self,bit):
        return int(self.num[bit-1:bit])
    
    def __str__(self):
        return self.num
    
    def __repr__(self):
        return self.num
    
def radix_sort(data_list):
    radix = 11
    ##借助稳定排序算法从尾至头排序 radix 次
    for i in range(radix,0,-1):
        counting_sort(data_list,i)

#改写的计数排序，方便基数排序调用,radix 指示是待排序数据的哪一位
def counting_sort(data_list,radix):
    length = len(data_list)
    #定义桶
    bucket = [] 
    max = data_list[0].get_bit(radix)
    for i in range(length):
        if data_list[i].get_bit(radix) > max:
            max = data_list[i].get_bit(radix)
    
    #初始化
    for i in range(max+1):
        bucket.append(0)

    #计数
    for i in range(length):
        bucket[data_list[i].get_bit(radix)] = bucket[data_list[i].get_bit(radix)] + 1
    
    ##累加
    for i in range(1,max+1):
        bucket[i] = bucket[i]+bucket[i-1]

    #计数排序,定义结果数组并初始化
    result = []
    for i in range(length):
        result.append(0)

    #从尾至头查找分数在result的插入位置，如果从头到尾的话就不是稳定的排序算法。
    for i in range(length-1,-1,-1):
        result[bucket[data_list[i].get_bit(radix)]-1] = data_list[i]
        bucket[data_list[i].get_bit(radix)] = bucket[data_list[i].get_bit(radix)] -1

    #将结果复制到原来的数组中，达到修改传入数组的效果
    for i in range(length):
        data_list[i] = result[i]

def create_phone_num():
    prelist = ["130", "131", "132", "133", "134", "135", "136", "137", "138", "139", "147", "150", "151", "152",
               "153", "155", "156", "157", "158", "159", "186", "187", "188"]
    randomPre = random.choice(prelist)
    Number = "".join(random.choice("0123456789") for i in range(8))
    phoneNum = randomPre +Number
    return phoneNum

if __name__ == "__main__":
    data_list = [phone_num(create_phone_num()) for _ in range(10)]
    print("排序前")
    for i in data_list:
        print(i)

    radix_sort(data_list)

    print("排序后")
    for i in data_list:
        print(i)
```
代码的最开始处，先定义电话号码的类，并实现返回某一位数值的函数get_bit()，这样在计数排序函数中根据某一位来排序了，也可以直接使用字符串数组，只不过那样写的话代码的可读性不好。代码还修改了计数排序函数，使之适用于基数排序。在排序前随机的生成了 10 个手机号码，执行结果如下所示：

```python
排序前
13834120744
13512786443
13243514461
13602161246
18691910549
14759154249
13080747777
13890979666
13995122272
15248505554
排序后
13080747777
13243514461
13512786443
13602161246
13834120744
13890979666
13995122272
14759154249
15248505554
18691910549
```
可以看出，基数排序调用了 11 次计数排序对手机号码进行排序，每次计数排序的时间复杂度为 O(n)，因此使用基数排序对类似这样的数据排序的时间复杂度也为 O(n)。

#### 基数排序的适用场景

基数排序对要排序的数据是有要求的，需要可以分割出独立的“位”来比较，而且位之间有递进的关系，如果 a 数据的高位比 b 数据大，那剩下的低位就不用比较了。除此之外，每一位的数据范围不能太大，要可以用线性排序算法来排序，否则，基数排序的时间复杂度就无法做到 O(n) 了。

在实际应用中，字符串之间排序就可以使用基数排序，如果待排序的字符串位数不一致，可以通过在字符串尾部补 0 来使他们位数一致，这与小数转整数后再排序的道理是一致的。


## 解答开篇
到这里，我想你已经知道如何给根据年龄给 100 万用户排序了，就类似按照成绩给 100 万考生排序。我们假设年龄的范围最小 1 岁，最大不超过 120 岁。我们可以遍历这 100 万用户，根据年龄将其划分到这 120 个桶里，然后依次顺序遍历这 120 个桶中的元素。这样就得到了按照年龄排序的 100 万用户数据。

（完）

>以下是本人学习极客时间的专栏《数据结构与算法之美》后，自己动手敲代码实现，并写下当时的思考，希望对你也有帮助。
系列文章：
[工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/)
[Python-排序-冒泡排序-优化](https://somenzz.github.io/2018/12/10/python_bubble_sort/)
[Python-排序-选择排序-优化](https://somenzz.github.io/2018/12/12/python_select_sort/)
[Python-排序-插入排序-优化](https://somenzz.github.io/2018/12/15/python_insert_shell_sort/)
[Python-排序-归并排序-哨兵的妙用](https://somenzz.github.io/2018/12/17/python_merge_sort/)
[Python|算法|快速排序|如何在O(n)查找第K大元素](https://somenzz.github.io/2018/12/18/python_quick_sort/)


加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat.jpg)
