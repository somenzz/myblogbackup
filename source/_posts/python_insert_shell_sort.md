---
layout: post
title: "Python-排序-插入排序-优化"
date: 2018-12-15 16:38:57
comments: true
reward: true
tags: 
	- Python
	- 插入排序
---

>以下是本人学习极客时间的专栏《数据结构与算法之美》后，自己动手敲代码实现，并写下当时的思考，希望对你也有帮助。
系列文章：
[工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/)
[Python-排序-冒泡排序-优化](https://somenzz.github.io/2018/12/10/python_bubble_sort/)
[Python-排序-选择排序-优化](https://somenzz.github.io/2018/12/12/python_select_sort/)

<!-- more -->

插入排序，我想你也并不陌生。可以简单地这样理解，插入排序就是就是往一个有序的数列中添中新的数据，插入之后保证数据列仍然有序，因此叫插入排序。

那么具体是如何实现的呢？要想保证插入后数据仍然有序，就需要先确定插入数据的位置。 首先我们将待排序的数据分为两个区间，有序区间和无序区间，初始的有序区间只包含一个元素，也就是数组的第一个元素，其他的就是无序区间；然后，依次从无序区间中选取一个元素，在有序区间找到合适的插入位置将其插入，并保证已排序区间的数据一直有序，最后重复这个过程，直到无序区间的元素为空，算法结束。

关键点：找到合适的位置插入前，需要先将插入位置后面的元素，按顺序往后移动，空出位置后再将新元素插入。

你可以先试着自己写写代码，练习 Python 编码的能力，不能眼高手低。下面是我写的未优化的插入排序算法

## 未优化版插入排序

```python
#encoding=utf-8
def insert_sort(data_list):
    '''
    无优化版
    '''
    count=0 #统计循环次数
    length = len(data_list)
    for i in range(1,length ): #默认第一个位置的元素是已排序区间，因此下标从 1 开始
        tmp = data_list[i] #待插入的数据
        j = i 
        while j > 0: #从已排序区间查找插入位置
            count +=1
            if tmp < data_list[j-1]:
                data_list[j] = data_list[j-1]  #元素向后移动，腾出插入位置
            else:
                break
            j -= 1
        data_list[j] = tmp #插入操作
        print(data_list)
    print(f"总循环次数为 {count}")
    return data_list
```

上述代码中的 count 只是为了统计循环次数，目的是和优化版的进行对比，当然您也可以对时间复杂度进行分析来对比性能的差异。 print(data_list) 是为了打印出每一次插入后数据列的结果，您可以对比结果来理解插入排序算法。

我们先找一组数据试跑下：

```python
if __name__ == "__main__":
    unsort = [1,3,4,2,1,5,6,7,8,4]
    print(*insert_sort(unsort)) 
```

执行结果如下所示：

```python
[1, 3, 4, 2, 1, 5, 6, 7, 8, 4]
[1, 3, 4, 2, 1, 5, 6, 7, 8, 4]
[1, 2, 3, 4, 1, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 4, 5, 6, 7, 8]
总循环次数为 18
1 1 2 3 4 4 5 6 7 8
```

### 性能分析
- 空间复杂度：除了运行时需要一个临时变量存储交换的数据和下标，不需要额外的存储空间，因此空间复杂度是O(1)，是原地排序算法。
- 稳定性：对于值相同的元素，我们可以选择将后面出现的元素，插入到前面出现元素的后面，这样就可以保证原有的前后顺序不变，因此是一种稳定的排序算法。
- 时间复杂度：如果数据是有序的，我们不需要搬移任何数据，在查找插入位置时，从尾到头在有序区间查找插入位置，每次只需要比较一次即可确定插入位置，因此最好的时间复杂度为O(n)。如果数据是倒序的，每次都相当于在数据的第一个位置插入新数据，所以需要移动大量的数据，最坏时间复杂度为O(n^2)。平时时间复杂度，由于数据中插入一个元素的平均时间复杂度为O(n)，因此对于插入排序来说，每次插入操作都相当于在数组中插入一个数据，循环执行 n 次插入操作，所以平均时间复杂度为O(n^2)。

### 优化入口
当有序区间数据量很大时，查找数据的插入位置就会显得非常耗时，插入排序算法每次都是从有序区间查找插入位置，以此为切入点，我们可以使用二分查找法来快速确认待插入的位置，于是就有了优化版的插入排序算法，也叫二分查找插入算法。

## 优化版插入排序


```python
def insert_sort2(data_list):
    '''
    使用二分查找函数确定待插入元素在有序区间的插入位置
    '''
    count=0 #统计循环次数
    length = len(data_list)
    for i in range(1,length ): #默认第一个位置的元素是已排序区间，因此下标从 1 开始
        print(data_list)
        wait_insert_data = data_list[i] ## 等待插入元素
        move_index = i 
        insert_index,count1 = binary_search(data_list[0:i],wait_insert_data) #寻找插入位置
        count+=count1 #统计循环次数需要加上二分查找的循环次数
        while move_index > insert_index: #移动元素，直到待插入位置处
            count+=1
            data_list[move_index] = data_list[move_index - 1]
            move_index -= 1
        data_list[insert_index] = wait_insert_data #插入操作
        print(data_list)
    print(f"总循环次数为 {count}")
    return data_list


def binary_search(data_list,data):    
    """
    输入:有序列表，和待查找的数据data
    输出：data 应该在该有序列表的插入位置
    count 变量纯粹是为了统计循环次数而使用的，实际应用时可去除。
    """
    count = 0
    length = len(data_list)
    low = 0
    high = length-1
    ## 如果给定元素大于等于最后一个元素，则插入最后元素位置的后面
    ## 如果小于第一个元素，则插入位置0 
    if data >= data_list [length -1]: return length,0
    elif data < data_list [0]: return 0,0
    insert_index = 0 
    while low < high-1:
        count +=1
        mid = (low + high)//2 #python中的除法结果默认为浮点数取整数部分时使用 //
        if data_list[mid] > data:
            high = mid
            insert_index = high
        else:
            low = mid
            insert_index = low+1  #如果值相同或者值大于mid的值，那么插入位置位于其后面
    return insert_index,count
```

代码供参考，您可以自己动手写一写，这样会更加深印象，写一遍比看十遍的效果都要好。这里主要增加了二分查找的函数，二分查找的过程非常简单，不再此赘述。对循环次数的统计也加了了二分查找的统计。现在我们使用同样的数据列进行排序，如下所示


```python
if __name__ == "__main__":
    unsort = [1,3,4,2,1,5,6,7,8,4]
    print(*insert_sort2(unsort)) 
```

执行结果如下所示：


```python
[1, 3, 4, 2, 1, 5, 6, 7, 8, 4]
[1, 3, 4, 2, 1, 5, 6, 7, 8, 4]
[1, 2, 3, 4, 1, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 5, 6, 7, 8, 4]
[1, 1, 2, 3, 4, 4, 5, 6, 7, 8]
总循环次数为 14
1 1 2 3 4 4 5 6 7 8
```

从结果可以看出，总循环次数比未优化版少了 4 次，其实别小看这 4 次，当排序的数据量越大时，效果越明显。

### 优化之后的时间复杂度分析：
使用二分查找方法来确定插入位置，由于不是查找值相等的数据，而是基于比较的方法确认插入的合适位置，最好的情况是插入位置是有序区间的首部或尾部，只要和有序区间的首部或尾部元素比较一次即可，此时时间复杂度为O(1)，其他情况下，二分查找的方法需要执行到 low +1 = high 时才能确定插入位置，此时相当于求时间复杂度相当对 2^X = n 时，求 X，因为查询的次数就为 X，而 X 等于log<sub>2</sub>n**（以2为底，n的对数）**。即O(log<sub>2</sub>n) ，所以，二分查找排序比较次数为：X =log<sub>2</sub>n 。

二分查找插入排序耗时的操作有：n\* (比较 + 后移赋值)。时间复杂度如下：

- 最好情况：如果每次查找的位置是有序区的最后一位的后面一位，则无须进行后移赋值操作，其比较次数为：1  ，即 O(n\*1) = O(n)

- 最坏情况：如果每次查找的位置是有序区的第二个位置（如果是第一个位置，比较次数为 0 不是最坏情况），则需要的比较次数为：log<sub>2</sub>n，每次循环需要的赋值操作次数的数量级为  (n-1) 次。即O(n \*（log<sub>2</sub>n + （n-1）)) = O(n^2)。

- 平均时间复杂度：O(n^2)

##  分治思想的插入排序--希尔排序
上面介绍的插入排序相信大家已经清楚了，可是你有没有想过插入排序有个缺点，就是当待插入的元素如果需要插入到有序区间的首部时，需要移动大量的数据来腾出空位置供新元素插入。为了解决这个问题，希尔排序算法就产生了。

希尔排序是一种分组直接插入排序方法，其原理是：先将整个序列分割成若干小的子序列，再分别对子序列进行直接插入排序，使得原来序列成为基本有序。这样通过对较小的序列进行插入排序，然后对基本有序的数列进行插入排序，能够提高插入排序算法的效率。

直接插入排序是基于相邻的元素进行排序，如果说直接插入排序为步长为1 ，那么希尔排序就是先按步长为 K 来插入排序，然后在步长 K 排序的基础上再对步长 m 进行排序，当然 K 是大于 m 的，最后对步长 1 排
序。

### 希尔排序步长的计算：
步长 h 的初始值为 1，通过公式：h=3\*h+1 来循环计算，直到该间隔大于数组的大小时停止。 
,h 取值为（1，4，13，40…）

本人没有找到如此计算步长的原因，可能是这样计算步长会更加有效地减少元素移动次数吧，如果您知道原因，不访告诉我，非常感谢。

### 代码实现
假如数据的长度为 n 我就简单地采取除以 2 来求步长，最后到 1 结束，最终也可以达到效果，分别使用 for 循环和 while 循环来实现，你可以选择方便自己理解的句式来阅读，供参考，如有疑问请留言。


```python
def shell_sort(data_list):
    '''
    思想：分治策略
   使用 for 循环
    '''
    length = len(data_list)
    space  = length//2
    while space > 0:
        for i in range(space,length ): #默认第一个位置的元素是已排序区间，因此下标从 1 开始
            tmp = data_list[i] #待插入的数据
            index = i 
            for j in range(i-space,-1,-space): #从已排序区间查找插入位置
                if tmp < data_list[j]:
                    data_list[j+space] = data_list[j]  #元素向后移动，腾出插入位置
                    index = j #最后的j即为插入的位置
                else:
                    break
            data_list[index] = tmp #插入操作
            print(data_list)
        space = space // 2
    return data_list

def shell_sort2(data_list):
    '''
    思想：分治策略
    使用 while 循环
    '''
    length = len(data_list)
    space  = length//2
    while space > 0:
        i = space
        while i < length: #默认第一个位置的元素是已排序区间，因此下标从 1 开始
            tmp = data_list[i] #待插入的数据
            j = i
            while j >= space and data_list[j - space] > tmp: #从已排序区间查找插入位置
                data_list[j] = data_list[j-space]  #元素向后移动，腾出插入位置                    
                j -= space
            data_list[j] = tmp #插入操作
            print(data_list)
            i +=1
        space = space // 2
    return data_list
```

为方便查看 shell 排序的过程，我选取了简单的数列进行排序，如下所示：

```python
    unsort = [9,8,7,6,5,4,3,2,1]
    print(*shell_sort(unsort)) 
    unsort = [9,8,7,6,5,4,3,2,1]
    print(*shell_sort2(unsort)) 
```

执行结果如下所示：

```python
[5, 8, 7, 6, 9, 4, 3, 2, 1]
[5, 4, 7, 6, 9, 8, 3, 2, 1]
[5, 4, 3, 6, 9, 8, 7, 2, 1]
[5, 4, 3, 2, 9, 8, 7, 6, 1]
[1, 4, 3, 2, 5, 8, 7, 6, 9]
[1, 4, 3, 2, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
1 2 3 4 5 6 7 8 9
[5, 8, 7, 6, 9, 4, 3, 2, 1]
[5, 4, 7, 6, 9, 8, 3, 2, 1]
[5, 4, 3, 6, 9, 8, 7, 2, 1]
[5, 4, 3, 2, 9, 8, 7, 6, 1]
[1, 4, 3, 2, 5, 8, 7, 6, 9]
[1, 4, 3, 2, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 8, 7, 6, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
1 2 3 4 5 6 7 8 9
```

可以看出两种方法的执行结果是一样的。

- 原地排序算法：希尔排序不借助额外的存储空间，因此是原地排序算法。
- 稳定性：由于希尔排序是分组进行插入排序，值相同的元素会分布在不同的组中，因此他们的相对先后顺序会被打乱，因此它是一种不稳定的排序算法。
- 时间复杂度: O(log<sub>2</sub>n)，最好时间复杂度为 O(n)，最坏情况：O(log<sub>2</sub>n)， 平均时间复杂度：O(log<sub>2</sub>n)。

希尔排序的时间复杂度与增量的选取有关，但是现今仍然没有人能找出希尔排序的精确下界。一般的选择原则是：通过公式：h=3*h+1 来循环计算。

希尔排序在最坏的情况下和平均情况下执行效率相差不是很多，与此同时快速排序（O(log<sub>2</sub>n)）在最坏的情况下执行的效率会非常差。专家们提倡，几乎任何排序工作在开始时都可以用希尔排序，实际使用中证明它不够快，再改成快速排序这样更高级的排序算法。

##  为什么插入排序比冒泡排序更受欢迎
冒泡排序和插入排序的时间复杂度都是O(n^2)，都是稳定的原地排序算法，为什么插入排序就这么受欢迎呢？

前两篇文章有提到有序度，逆序度。其实不论怎么优化，冒泡排序的元素交换次数是一次的，等于原始数据的逆序度，插入排序也是同样，无论怎么优化，元素的移动次数也等于原始数据的逆序度。
下面可以对比冒泡排序的元素交换代码和插入排序的元素移动代码
-  冒泡排序的元素交换

```python
            if collection[j] > collection[j+1]:
                tmp = collection[j]
                collection[j] = collection[j+1]
                collection[j+1] = tmp
```


-  插入排序的元素交换

```python
            if tmp < data_list[j-1]:
                data_list[j] = data_list[j-1]  #元素向后移动，腾出插入位置
```

如果执行一个赋值语句花费的 cpu 时间为t ，那么一次交换冒泡消耗的 cpu 时间就为 3t，而插入排序则的元素移动只需要 t，当数据量非常大的时候这种耗时的差异就会非常明显，你可以生成大量的随机数据来测试下。虽然冒泡排序和插入排序的时间复杂度都为O(n^2)，但是如果希望把性能做到极致，肯定首选插入排序。

（完）


![个人公众号](/assets/img/wechat.jpg)
