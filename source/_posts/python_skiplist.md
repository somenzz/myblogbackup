---
layout: post
title: "跳表的设计思路，值得每一个程序员学习"
date: 2019-01-12 20:38:57
comments: true
reward: true
tags: 
	- Python
	- 跳表
---


学习《数据结构与算法之美》中的第 17 节 「为什么redis一定要用跳表来实现有序集合」后，觉得很有价值，以自己的理解整理出下文，分享给爱学习的你，希望你可以看懂。


<!-- more -->

上篇文章[二分法解决妹子遇到的难题](https://somenzz.github.io/2019/01/10/python_binary_search/) 介绍了二分查找算法法的强大之处。二分查找算法之所以能达到 O(logn) 这样高效的一个重要原因在于它所依赖的数据结构是数组，数组支持随机访问一个元素，通过下标很容易定位到中间元素。而链表是不支持随机访问的，只能从头到尾依次访问。但是数组有数组的局限性，比如需要连续的内存空间，插入删除操作会引起数组的扩容和元素移动，链表有链表的优势，链表不需要先申请连续的空间，插入删除操作的效率非常高。在很多情况下，数据是通过链表这种数据结构存储的，如果是有序链表，真的就没有办法使用二分查找算法了吗？

实际上对有序链表稍加改造，我们就可以对链表进行二分查找。这就是我们要说的跳表。下面我们来看一下，跳表是怎么跳的。

![image.png](https://upload-images.jianshu.io/upload_images/12989993-d048ddc4095f5557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图是一个简单的有序的单链表，如果要查找某个数据，只能从头至尾遍历链表，查找到值与给定元素时返回该结点，这样的查询效率很低，时间复杂度是为O(n)。

假如对链表进行改造，先对链表中每两个节点建立第一级索引，再对第一级索引每两个节点建立第二级索引。如下图所示：
![](https://upload-images.jianshu.io/upload_images/12989993-041013e24f5cfcdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于上图中的带二级索引的链表中，我们查询元素 16，先从第二级索引查询 1 -> 7->13，发现16大于13 ，然后通过 13 的 down 指针找到第一级索引的 17，发现 16 小于17 ，再通过13 的 down 指针找到链表中的 16，只需要遍历 6 个节点就完成 16 的查找。如果在单链表中直接查找 16 的话，只能顺序遍历，需要遍历 10 个节点，是不是效率上有所提升呢，由于数据量较小，遍历 10 个节点到遍历 6 个节点你可能觉得没有提升多少性能，那么请看下图：

![](https://upload-images.jianshu.io/upload_images/12989993-8934124a8d62cd66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从图中我们可以看出，原来没有索引的时候，查找 62 需要遍历 62 个结点，现在只需要遍历 11 个结点，速度是不是提高了很多？所以，当链表的长度 n 比较大时，比如 1000、10000 的时候，在构建索引之后，查找效率的提升就会非常明显。

**这种带多级索引的链表，就是跳表。**，是不是很像数据库中的索引？

## 跳表有多快？

单链表的查找一个元素的时间复杂度为O(n)，那么跳表的时间复杂度是多少？

假如链表中有 n 个元素，我们每两个节点建立一个索引，那么第 1 级索引的结点个数就是 n/2 ，第二级就是 n/4，第三级就是 n/8, 依次类推，也就是说**第 k 级索引的结点个数是第 k-1 级索引的结点个数的 1/2，那么第k级索引的节点个数为 n 除以 2 的 k 次方，即 n/(2^k)。**

假设索引有 h 级，最高级的索引有 2 个结点。通过上面的公式，我们可以得到 n/(2^h) = 2，得到 h=log2n - 1，包含原始链表这一层的话，跳表的高度就是 log2n，假设每层需要访问 m 个结点，那么总的时间复杂度就是O(m\*log2n)。而每层需要访问的 m 个结点，m 的最大值不超过 3，这里为什么是 3 ，可以自己试着走一个。
![](https://upload-images.jianshu.io/upload_images/12989993-6ed52ddce59ccc27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因此跳表的时间复杂度为O(3log2n) = O(log2n)


## 跳表有多占内存？
天下没有免费的午餐，时间复杂度能做到 O(logn) 是以建立在多级索引的基础之上，这会导致内存占用增加，那么跳表的空间复杂度是多少呢？

假如有 n 个元素的链表，第一级索引为 n/2 个，第二级为 n/4 个，第三级为 n/8 个，......，最后一级为 2 个。这几级索引的结点总和就是n/2+n/4+n/8…+8+4+2=n-2。所以，跳表的空间复杂度是 O(n)。也就是说，如果将包含 n 个结点的单链表构造成跳表，我们需要额外再用接近 n 个结点的存储空间。那我们有没有办法降低索引占用的内存空间呢？

假如每 3 个节点抽取一个作为索引，同样的方法，可以计算出空间复杂度为 O(n/2) ,已经节约一半的存储空间了。

实际上，在软件开发中，我们不必太在意索引占用的额外空间。在讲数据结构和算法时，我们习惯性地把要处理的数据看成整数，但是在实际的软件开发中，原始链表中存储的有可能是很大的对象，而索引结点只需要存储几个指针，并不需要存储对象，所以当对象比索引结点大很多时，那索引占用的额外空间就可以忽略了。

## 跳表如何实现

跳表这种拿空间换时间的思想非常巧妙。那么如何编程实现一个跳表的数据结构呢？

**其实，知道与践行之间隔着巨大的鸿沟，知道那么多的算法，可是仍写不出牛逼的代码。还是要多写多练，不然就会被说 talk is cheap,show me the code。**

编码之前，应该思考一下跳表应支持的功能：
1、插入一个元素
2、删除一个元素
3、查找一个元素
4、查找一个区间的元素
5、输出有序序列

其实 redis 中有序集合支持的核心操作也就是这几个。这里说下为什么 redis 使用跳表而不使用红黑树。

1、红黑树在查找区间元素的效率没有跳表高，其他操作时间复杂度一致。
2、相比红黑树，跳表的实现还是简单的，简单就意味着不容易出错，bug 少，稳定，易读，易维护。
3、跳表更加灵活，通过改变索引构建策略，有效平衡效率和内存消耗。

### 0、跳表的数据结构 （python)

**链表结点**
```python
class ListNode:
    def __init__(self, data = None):
        self._data = data
        self._forwards = []   # 存放类似指针/引用的数组，占用空间很小
```
这里的 \_data 是 ListNode 的变量，前而加 \_data 表示这是一个私有变量，虽然你能在类的外部修改它，但你最好不要这样做。（Python 在编码规范上并不阻止你做一些破坏（灵活），全靠你自觉）
\_data 这里是做比较用的，在实际应用中，你可以这样写：

```python
class ListNode:
    def __init__(self, key=None, value  = None):
        self._key = key
        self._value = value
        self._forwards = []   # 存放类似指针/引用的数组，占用空间很小
```
这里的 \_key 就相当于上述的 \_data。

**跳表的类**

```python
class SkipList:

    _MAX_LEVEL = 4 

    def __init__(self):
        self._level_count = 1
        self._head = ListNode()
        self._head._forwards = [None] * self._MAX_LEVEL

    def find(self, value):
        '''
        查找一个元素，返回一个 ListNode 对象
        '''
        pass

    def find_range(self, begin_value, end_value) :
        '''
        查找一个元素，返回一组有序 ListNode 对象
        '''
        pass

    def insert(self, value):
        '''
        插入一个元素，返回 True 或 False
        '''
        pass
        
    def delete(self, value):
        '''
        删除一个元素，返回 True 或 False
        '''
        pass

    def _random_level(self, p = 0.5):
        '''
        返回随机层数
        '''
        pass

    def pprint(self):
        '''
        打印跳表
        '''
        pass
```

跳表实际上长这样：

![](https://upload-images.jianshu.io/upload_images/12989993-fa8a69f065a42740.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图中 0 代表的是原始链表的指针，姑且按指针来理解，虽然Python中并没有指针的概念。1 代表第一层索引的指针，2 代表第二层索引，依次类推。这样设计的好处是一个节点在内存中只存放一次，而多存放几个指针并不占用太多存储空间。


### 1、插入一个元素
在链表中插入一人元素非常简单，但在跳表中，还要维护索引，如果不维护索引，两个索引节点下的数据可能会变得非常多，极端情况下，跳表会退化成单链表，查找一个结点的时间复杂度退化为O(n)，为了解决这个问题，采用随机数的方法，每次在 0 到 最大层数 m 之间选一个随机数 k，每次插入结点时同时更新  0 到 k 之间的索引信息。

```python
    def _random_level(self, p = 0.5):
        level = 1
        while random.random() < p and level < self._MAX_LEVEL:
            level += 1
        return level
```
插入节点要判断节点是否已经存在跳表中。

```python
    def insert(self, value):
        '''
        插入一个结点，成功返回 True，失败返回 False
        '''
        level = self._random_level()
        if self._level_count < level: self._level_count = level
        new_node = ListNode(value)
        new_node._forwards = [None] * level
        update = [self._head] * level     # update 保存插入结节的左边的节点

        p = self._head
        for i in range(level - 1, -1, -1):
            while p._forwards[i] and p._forwards[i]._data < value:
                p = p._forwards[i]
            if p._forwards[i] and p._forwards[i]._data == value:
                #说明已经存储该节点，不需要再插入
                return False 
            update[i] = p     # 找到插入的位置

        for i in range(level):
            new_node._forwards[i] = update[i]._forwards[i]   # new_node.next = prev.next
            update[i]._forwards[i] = new_node     # prev.next = new_node
        return True
        
```

### 3、删除一个元素

删除操作是插入操作的逆操作，先找到待删除的元素，再删除之。

```python
    def delete(self, value):
        update = [None] * self._level_count
        p = self._head
        for i in range(self._level_count - 1, -1, -1):
            while p._forwards[i] and p._forwards[i]._data < value:
                p = p._forwards[i]
            update[i] = p
        
        if p._forwards[0] and p._forwards[0]._data == value:
            for i in range(self._level_count - 1, -1, -1):
                if update[i]._forwards[i] and update[i]._forwards[i]._data == value:
                    update[i]._forwards[i] = update[i]._forwards[i]._forwards[i]     # Similar to prev.next = prev.next.next
            return True
        else:
            return False
```

### 4、查找一个元素
查找操作从最高层的索引开始，逐步向下层查找，类似于二分查找算法，时间复杂度为O(logn)，非常高效。假如有下面的跳表
```python
head 3: ->27->33->39
head 2: ->12->27->33->39
head 1: ->3->6->12->18->21->24->27->30->33->39
head 0: ->3->6->9->12->18->21->24->27->30->33->36->39
```
如果查找 33 ，则从最高层 head 3 开始，遍历 27，33 两个结点就返回查找成功。如果查找 36 则遍历 27，33，39，39，39，36 共 6 个结节就返回查找成功。下面是代码实现：

```python
    def find(self, value):
        '''
        查找一个元素，返回一个 ListNode 对象
        '''
        p = self._head
        for i in range(self._level_count - 1, -1, -1):   # Move down a level
            while p._forwards[i] and p._forwards[i]._data < value:
                p = p._forwards[i]   # Move along level
            if p._forwards[i] and p._forwards[i]._data == value:
                return p._forwards[i]
        #到这一步，说明没有找到
        return None
```
### 5、查找一个区间的元素。
这一步在上一步的基础上写非常简单，先查找到区间的小元素，然后在有序链表上顺序遍历，直到元素比区间的大元素大时在终止遍历即可。

```python
    def find_range(self, begin_value, end_value) :
        '''
        查找一个元素，返回一组有序 ListNode 对象
        '''
        p = self._head
        begin = None
        for i in range(self._level_count - 1, -1, -1):   # Move down a level
            while p._forwards[i] and p._forwards[i]._data < begin_value:
                p = p._forwards[i]   # Move along level
            if p._forwards[i] and p._forwards[i]._data >= begin_value:
                begin = p._forwards[i]

        if begin is None:
            return None #没有找到
        else:
            result = []
            while begin and begin._data <= end_value :
                result.append(begin)
                begin = begin._forwards[0] 
            return result
```
### 6、遍历输出跳表

这一步没难度，从顶层往下遍历即可，就两个 while 循环搞定 

```python
    def pprint(self):
        '''
        打印跳表
        '''
        skiplist_str = ''
        i = self._level_count -1 
        while i >= 0:
            p = self._head
            skiplist_str = f'head {i}: '
            while p:
                if p._data:
                    skiplist_str += '->' + str(p._data)
                p = p._forwards[i]
            print(skiplist_str)
            i -= 1
```

### 验证一把
```python
if __name__ == "__main__":
    l = SkipList()
    for i in range(0,40,3):
        l.insert(i)
    l.pprint() 
    if l.delete(15):
        print("delete 15 success.")
    l.pprint() 
    if not l.delete(16):
        print("delete 16 fail.")
    l.pprint() 
    print("find 9 : ",l.find(9)._data)
    print("find data between 4 and 10:")
    for d in l.find_range(4,10):
        print(d._data,end ='->')
```
执行结果如下所示：

```python
head 3: ->27->33->39
head 2: ->12->27->33->39
head 1: ->3->6->12->15->18->21->24->27->30->33->39
head 0: ->3->6->9->12->15->18->21->24->27->30->33->36->39
delete 15 success.
head 3: ->27->33->39
head 2: ->12->27->33->39
head 1: ->3->6->12->18->21->24->27->30->33->39
head 0: ->3->6->9->12->18->21->24->27->30->33->36->39
delete 16 fail.
head 3: ->27->33->39
head 2: ->12->27->33->39
head 1: ->3->6->12->18->21->24->27->30->33->39
head 0: ->3->6->9->12->18->21->24->27->30->33->36->39
find 9 :  9
find data between 4 and 10:
6->9->
```
如果仍看不懂，就开启你的 debug，分析每一个变量的变化状态 。

其实写不出实现跳表的代码也无妨，其实每一种语言都已经实现了跳表，只要掌握其核心思想，直接拿来用就可以了。

下面是 python 关于跳表的三方库，你不需要重复造轮子了。

1、[skiplist （纯 C 编写） ](https://bitbucket.org/mojaves/pyskiplist/src/)
2、[pyskip 纯Python](https://github.com/toastdriven/pyskip/)
3、[py-skiplist 纯Python](https://github.com/ZhukovAlexander/py-skiplist)

系列文章：

[工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/)
[Python-排序-冒泡排序-优化](https://somenzz.github.io/2018/12/10/python_bubble_sort/)
[Python-排序-选择排序-优化](https://somenzz.github.io/2018/12/12/python_select_sort/)
[Python-排序-插入排序-优化](https://somenzz.github.io/2018/12/15/python_insert_shell_sort/)
[Python-排序-归并排序-哨兵的妙用](https://somenzz.github.io/2018/12/17/python_merge_sort/)
[Python|算法|快速排序|如何在O(n)查找第K大元素](https://somenzz.github.io/2018/12/18/python_quick_sort/)
[Python-排序-有哪些时间复杂度为O(n)的排序算法？](https://somenzz.github.io/2018/12/25/python_linear_sort/)
[Python-算法-二分法解决妹子的遇到难题](https://somenzz.github.io/2019/01/10/python_binary_search/)






（完）

加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat2.jpg)
