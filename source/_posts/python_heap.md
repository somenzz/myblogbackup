---
layout: post
title: "堆的实现及工程应用(Python)"
date: 2019-02-24 20:38:57
comments: true
reward: true
tags: 
	- Python
	- 堆 
---



堆和栈是计算机程序设计中非常重要的数据结构，操作系统和数据库均有非常广泛的应用，掌握好这两种数据结构也可以高效地解决工程问题。今天分享一下在极客专栏学到到的堆的实现和工程应用，希望对你有所启发。

<!-- more -->


堆有两点需要了解，一是堆是一颗完全二叉树，完全二叉树就是只有最后一层有页子节点，而且页子节点是靠左排列的；二是堆中的每一个节点都大于其左右子节点（大顶堆），或者堆中每一个节点都小于其左右子节点（小顶堆）。

![哪些是堆](https://upload-images.jianshu.io/upload_images/12989993-bc24085a2b8ff365.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上图中，1 和 2 是大顶堆，3 是小顶堆，4 不是堆。


## 堆的实现

堆是一颗完全二叉树，完全二叉树使用数据存储最节省内存，因为不需要保存左右子节点的指针。
![使用数组存储堆](https://upload-images.jianshu.io/upload_images/12989993-effab879ca241c40.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图中第一个位置不存储数据，是为了方便计算，其实第一个位置也可以存储数据，计算时下标加 1 即可。

```python
    def __init__(self):
        '''
        初始化一个空堆，使用数组来在存放堆元素，节省存储
        '''
        self.data_list = []
```

堆还要实现的功能有：
1. 插入一个元素。
2. 删除堆顶元素。

#### 插入一个元素
先来实现插入一个元素，插入元素的过程中确保堆的两点，一个是确保它是完全二叉树，二是确保它符合大顶堆（小顶堆），本例以大顶堆为例。

完全二叉树使用数组存储，只要在数组的最后插入新元素，然后让其与父节点比较，如果大于父节点，则与父节点交换，直到小于父节点或到达要节点为止。以下两图有助于理解插入过程。
![1.插入元素](https://upload-images.jianshu.io/upload_images/12989993-ca5abb41af891771.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2.插入元素](https://upload-images.jianshu.io/upload_images/12989993-e19bf8038e28ac32.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面是代码实现：

```python
    def insert(self,data):
        '''
        先把元素放在最后，然后从后往前依次堆化
        这里以大顶堆为例，如果插入元素比父节点大，则交换，直到最后
        '''
        self.data_list.append(data)
        index = len(self.data_list) -1 
        parent = self.get_parent_index(index)
        #循环，直到该元素成为堆顶，或小于父节点（对于大顶堆) 
        while parent is not None and self.data_list[parent] < self.data_list[index]:
            #交换操作
            self.swap(parent,index)
            index = parent
            parent = self.get_parent_index(parent)
```
#### 删除堆顶元素
删除堆顶元素后，将数组最后一个元素放在堆顶位置，然后从上到下进行堆化，这样就可以确保在删除的过程中仍然是一棵完全二叉树。堆化就是查看当前节点与左右子节点进行比较，如果小于子节点，则与其交换，直到满足大顶堆的条件为止。

下面的图片有助于理解删除过程。

![删除堆顶元素的过程](https://upload-images.jianshu.io/upload_images/12989993-2d7778c27e8fcfc1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码实现：
```python
    def removeMax(self):
        '''
        删除堆顶元素，然后将最后一个元素放在堆顶，再从上往下依次堆化
        '''
        remove_data = self.data_list[0]
        self.data_list[0] = self.data_list[-1]
        del self.data_list[-1]

        #堆化
        self.heapify(0)
        return remove_data


    def heapify(self,index):
        '''
        从上往下堆化，从index 开始堆化操作 (大顶堆)
        '''
        total_index = len(self.data_list) -1
        while True:
            maxvalue_index = index
            if 2*index +1 <=  total_index and self.data_list[2*index +1] > self.data_list[maxvalue_index]:
                maxvalue_index = 2*index +1
            if 2*index +2 <=  total_index and self.data_list[2*index +2] > self.data_list[maxvalue_index]:
                maxvalue_index = 2*index +2
            if maxvalue_index == index:
                break
            self.swap(index,maxvalue_index)
            index = maxvalue_index
```

#### 测试

完整代码

```python
#encoding = utf-8


class heap(object):

    def __init__(self):
        '''
        初始化一个空堆，使用数组来在存放堆元素，节省存储
        '''
        self.data_list = []

    def get_parent_index(self,index):
        '''
        返回父节点的下标
        '''
        if index == 0 or index > len(self.data_list) -1:
            return None
        else:
            return (index -1) >> 1 

    def swap(self,index_a,index_b):
        '''
        交换数组中的两个元素
        '''
        self.data_list[index_a],self.data_list[index_b] = self.data_list[index_b],self.data_list[index_a] 


    def insert(self,data):
        '''
        先把元素放在最后，然后从后往前依次堆化
        这里以大顶堆为例，如果插入元素比父节点大，则交换，直到最后
        '''
        self.data_list.append(data)
        index = len(self.data_list) -1 
        parent = self.get_parent_index(index)
        #循环，直到该元素成为堆顶，或小于父节点（对于大顶堆) 
        while parent is not None and self.data_list[parent] < self.data_list[index]:
            #交换操作
            self.swap(parent,index)
            index = parent
            parent = self.get_parent_index(parent)

    def removeMax(self):
        '''
        删除堆顶元素，然后将最后一个元素放在堆顶，再从上往下依次堆化
        '''
        remove_data = self.data_list[0]
        self.data_list[0] = self.data_list[-1]
        del self.data_list[-1]

        #堆化
        self.heapify(0)
        return remove_data


    def heapify(self,index):
        '''
        从上往下堆化，从index 开始堆化操作 (大顶堆)
        '''
        total_index = len(self.data_list) -1
        while True:
            maxvalue_index = index
            if 2*index +1 <=  total_index and self.data_list[2*index +1] > self.data_list[maxvalue_index]:
                maxvalue_index = 2*index +1
            if 2*index +2 <=  total_index and self.data_list[2*index +2] > self.data_list[maxvalue_index]:
                maxvalue_index = 2*index +2
            if maxvalue_index == index:
                break
            self.swap(index,maxvalue_index)
            index = maxvalue_index
        

        
if __name__ == "__main__":
    myheap = heap()
    for i in range(10):
        myheap.insert(i+1)
    print('建堆:',myheap.data_list)

    print("删除堆顶元素：", [myheap.removeMax() for _ in range(10)])
 
```
运行结果如下：

```python
建堆: [10, 9, 6, 7, 8, 2, 5, 1, 4, 3]
删除堆顶元素： [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

删除堆顶数据和往堆中插入数据的时间复杂度都是 O(logn)，网上有很多证明方法，不再详述。


## 堆的应用

**1、堆排序**。

分两个过程：建堆和排序，建堆的过程就是堆插入元素的过程，我们可以对初始数组原地建堆，然后再依次输出堆顶元素即可达到排序的目的。建堆的时间复杂度为 O(n)，排序过程的时间复杂度为 O(nlogn)，堆排序不是稳定的排序算法，因为在排序的过程中存在将堆的最后一个元素跟堆顶元素交换的操作，可能改变原始相对顺序。

**2、优先级队列**。

优先级队列，顾名思义，它首先应该是一个队列。队列最大的特性就是先进先出。不过，在优先级队列中，数据的出队顺序不是先进先出，而是按照优先级来，优先级最高的，最先出队。如何实现一个优先级队列呢？方法有很多，但是用堆来实现是最直接、最高效的。这是因为，堆和优先级队列非常相似。一个堆就可以看作一个优先级队列。很多时候，它们只是概念上的区分而已。往优先级队列中插入一个元素，就相当于往堆中插入一个元素；从优先级队列中取出优先级最高的元素，就相当于取出堆顶元素。

干说优先级队列可能有点抽象，下面举两个实用的例子来说明优先级队列。

1）**合并有序小文件**。

假如有 100 个小文件，每个小文件都为 100 MB，每个小文件中存储的都是有序的字符串，现在要求合并成一个有序的大文件，那么如何做呢？

直观的做法是分别取每个小文件的第一行放入数组，再比较大小，依次插入到大文件中，假如最小的行来自于文件 a，那么插入到大文件中后，从数组中删除该行，再取文件 a 的下一行插入到数组中，再次比较大小，取出最小的插入到大文件的第二行，依次类推，整个过程很像归并排序的合并函数。每次插入到大文件中都要循环遍历整个数组，这显然是低效的。

而借助于堆这种优先级队列就很高效。比如我们可以分别取 100 个文件的第一行建一个小顶堆，假如堆顶元素来自于文件 a，那么取出堆顶元素插入到大文件中，并从堆顶删除该元素（就是堆实现中 removeMax 函数）, 然后再从文件 a 中取下一行插入到堆顶中，重复以上过程就可以完成合并有序小文件的操作。

删除堆顶数据和往堆中插入数据的时间复杂度都是 O(logn)，n 表示堆中的数据个数，这里就是 100。是不是比原来数组存储的方式高效了很多呢？

2）**高性能定时器**。

假如有很多定时任务，如何设计一个高性能的定时器来执行这些定时任务呢？假如每过一个很小的单位时间（比如 1 秒），就扫描一遍任务，看是否有任务到达设定的执行时间。如果到达了，就拿出来执行。这显然是浪费资源的，因为这些任务的时间间隔可能长达数小时。

借助于堆这种优先级队列我们这可以这样设计：将定时任务按时间先后的顺序建一个小顶堆，先取出堆顶任务，查询其执行时间与当前时间之差，假如为 T 秒，那么在 T - 1 秒的时间内，定时器什么也不需要做，当 T 秒间隔达到时就取出任务执行，对应的从堆顶删除堆顶元素，然后再取下一个堆顶元素，查询其执行时间。

这样，定时器既不用间隔 1 秒就轮询一次，也不用遍历整个任务列表，性能也就提高了。

**3、取 top k 元素。**

取 top k 元素的情形可分为两类，一类是静态数据集合，也就是说数据确定后不再增加新的元素，另一类是动态数据集合，会随时增加元素，但依然求第 k 大元素。

对于静态数据，我们可以先从静态数据依次插入小顶堆中，维护一个大小为 k 的小顶堆，遍历其余数据，依次插入到大小为 k 的小顶堆中，如果元素比 k 小，则不做处理，继续遍历下一个数据，如果比 k 大，则删除堆顶堆，并将该值插入到堆顶中，这样遍历结束时，堆顶元素就是第 k 大元素。

遍历数组需要 O(n) 的时间复杂度，一次堆化操作需要 O(logK) 的时间复杂度，所以最坏情况下，n 个元素都入堆一次，所以时间复杂度就是 O(nlogK)。

对于动态数据，处理方法也是一样的，相当于实时求 top k，那么每次求 top k 时重新计算一下即可，时间复杂度仍是 O(nlogK)，n 表示当前的数据的大小。我们可以一直都维护一个 K 大小的小顶堆，当有数据被添加到集合中时，我们就拿它与堆顶的元素对比。如果比堆顶元素大，我们就把堆顶元素删除，并且将这个元素插入到堆中；如果比堆顶元素小，则不做处理。这样，无论任何时候需要查询当前的前 K 大数据，我们都可以里立刻返回给他。

**4、取中位数。**

什么是中位数，先看下图：

![中位数](https://upload-images.jianshu.io/upload_images/12989993-248a5fb2b1cb85f4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如何利用堆求中位数？

对于一组静态数据，中位数是固定的，我们可以先排序，第 n/2 个数据就是中位数。每次询问中位数的时候，我们直接返回这个固定的值就好了。所以，尽管排序的代价比较大，但是边际成本会很小。但是，如果我们面对的是动态数据集合，中位数在不停地变动，如果再用先排序的方法，每次询问中位数的时候，都要先进行排序，那效率就不高了。

对于动态数据，假如个数为 n，当然 n 是会变化的。初始化时先对 n 个数据从小到大排序。 

当 n 为偶数时，我们维护两个堆，将有序数据中的前面  n/2 个元素维护成大顶堆，后面 n/2 的维护成小顶堆，小顶堆中的元素都大于大顶堆中的元素，大顶堆的堆顶元素和小顶堆的堆顶元素就是中位数。

当 n 为奇数时，同样维护两个堆，将前面 n/2 + 1 个元素维护成大顶堆，将后面 n/2 个元素维护成小顶堆，这样大顶堆的堆顶元素就是中位数。

下面的图有助于你理解：

![中位数1.jpg](https://upload-images.jianshu.io/upload_images/12989993-ecadc7310ab5fc58.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面考虑新插入元素，当新插入元素大于等于小顶堆堆顶元素时就插入小顶堆，否则就插入大顶堆。

这样有可能导致大顶堆和小顶堆的元素个数不满足上述 n 为奇数和偶数的两种情况，当不满足时就转移元素，当小顶堆的元素多于大顶堆时，就将小顶堆堆顶元素插入到大顶堆中，同时从小顶堆中删除堆顶元素。

下图有助于理解上述过程：

![中位数2.jpg](https://upload-images.jianshu.io/upload_images/12989993-1ff867a274a0e4cd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1）**中位数引申出的 99% 响应时间**

如何快速求接口的 99% 响应时间？

中位数的概念就是将数据从小到大排列，处于中间位置，就叫中位数，这个数据会大于等于前面 50% 的数据。99 百分位数的概念可以类比中位数，如果将一组数据从小到大排列，这个 99 百分位数就是大于前面 99% 数据的那个数据。

弄懂了这个概念，我们再来看 99% 响应时间。如果有 100 个接口访问请求，每个接口请求的响应时间都不同，比如 55 毫秒、100 毫秒、23 毫秒等，我们把这 100 个接口的响应时间按照从小到大排列，排在第 99 的那个数据就是 99% 响应时间，也叫 99 百分位响应时间。

同样地，维护两个堆，一个大顶堆，一个小顶堆。假设当前总数据的个数是 n，大顶堆中保存 n*99% 个数据，小顶堆中保存 n*1% 个数据。大顶堆堆顶的数据就是我们要找的 99% 响应时间。

每次插入一个数据的时候，我们要判断这个数据跟大顶堆和小顶堆堆顶数据的大小关系，然后决定插入到哪个堆中。如果这个新插入的数据比大顶堆的堆顶数据小，那就插入大顶堆；如果这个新插入的数据比小顶堆的堆顶数据大，那就插入小顶堆。

来个有趣味的问题：

如何在单机 1 GB 的内存中从 10 亿条搜索关键词日志记录中找出 top 10 最热门的关键词，假如每条关键词平均占用 50 字节。

这里给出方法（来自极客专栏，详见前文 [工作后，为什么还要学习数据结构与算法（文末福利）](https://somenzz.github.io/2018/12/09/algorthms/)）
>10 亿的关键词还是很多的。我们假设 10 亿条搜索关键词中不重复的有 1 亿条，如果每个搜索关键词的平均长度是 50 个字节，那存储 1 亿个关键词起码需要 5GB 的内存空间，的机器只有 1GB 的可用内存空间，所以我们无法一次性将所有的搜索关键词加入到内存中。这个时候该怎么办呢？

>相同数据经过哈希算法得到的哈希值是一样的。我们可以利用哈希算法的这个特点，将 10 亿条搜索关键词先通过哈希算法分片到 10 个文件中。

>具体可以这样做：我们创建 10 个空文件 00，01，02，……，09。我们遍历这 10 亿个关键词，并且通过某个哈希算法对其求哈希值，然后哈希值同 10 取模，得到的结果就是这个搜索关键词应该被分到的文件编号。

>我们针对每个包含 1 亿条搜索关键词及其搜索记录数（通过hash算法得到相同搜索关键词的记录数）的文件，利用散列表和堆，分别求出 Top 10，然后把这个 10 个 Top 10 放在一块，然后取这 100 个关键词中，出现次数最多的 10 个关键词，这就是这 10 亿数据中的 Top 10 最频繁的搜索关键词了。


优先级队列是一种特殊的队列，优先级高的数据先出队，而不再像普通的队列那样，先进先出。实际上，堆就可以看作优先级队列，只是称谓不一样罢了。求 Top K 问题又可以分为针对静态数据和针对动态数据，只需要利用一个堆，就可以做到非常高效率的查询 Top K 的数据。求中位数实际上还有很多变形，比如求 99 百分位数据、90 百分位数据等，处理的思路都是一样的，即利用两个堆，一个大顶堆，一个小顶堆，随着数据的动态添加，动态调整两个堆中的数据，最后大顶堆的堆顶元素就是要求的数据。

（完）



系列文章：

[工作后，为什么还要学习数据结构与算法(文末福利)](https://somenzz.github.io/2018/12/09/algorthms/)
[Python-排序-冒泡排序-优化](https://somenzz.github.io/2018/12/10/python_bubble_sort/)
[Python-排序-选择排序-优化](https://somenzz.github.io/2018/12/12/python_select_sort/)
[Python-排序-插入排序-优化](https://somenzz.github.io/2018/12/15/python_insert_shell_sort/)
[Python-排序-归并排序-哨兵的妙用](https://somenzz.github.io/2018/12/17/python_merge_sort/)
[Python|算法|快速排序|如何在O(n)查找第K大元素](https://somenzz.github.io/2018/12/18/python_quick_sort/)
[Python-排序-有哪些时间复杂度为O(n)的排序算法？](https://somenzz.github.io/2018/12/25/python_linear_sort/)
[Python-算法-二分法解决妹子的遇到难题](https://somenzz.github.io/2019/01/10/python_binary_search/)
[跳表的设计思路，值得每一个程序员学习](https://somenzz.github.io/2019/01/12/python_skiplist/)
[二叉树的Python实现](https://somenzz.github.io/2019/01/23/python_binart_tree/)



（完）

加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat2.jpg)
