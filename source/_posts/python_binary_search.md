---
layout: post
title: "Python-算法-二分法解决妹子的遇到难题"
date: 2019-01-10 15:38:57
comments: true
reward: true
tags: 
	- Python
	- 二分查找 
---



有一个妹子，每天都会在朋友圈放一张自拍照，由于颜值不错，赢得不少朋友的点赞，妹子很开心。直到有一天，闺蜜告诉她在陌陌上看到了她同样的照片，妹子听后非常吃惊，也非常生气，也不知道是哪个贼 X 盗图，告知闺蜜自己只发朋友圈，这肯定是盗图，听说陌陌是约泡的软件，自己都没注册过。闺蜜说，我还不了解你么，但是别人可不了解，当务之急要找出这个贼 X。

<!-- more -->


妹子点头，但是怎么找呢？直接问陌陌上的那个人肯定是得不到正确答案的，而且会打草惊蛇，贼 X 应该在微信好友里，有500多好友，怎么找呢？

妹子之前加入了一个程序员众多的社群，便向社群里求助，然后群里聊得热火朝天，争先恐后的献计，不过程序员脑的脑子就是好使，有人就给出了很高效的解决方案：

>二分查找法，现对微信好友分两组，第一天发照片时设置对其中的一组（A组）可见，对另一组（B组）不可见，然后观察陌陌，如果陌陌上有相同照片，说明贼在 A 组，否则在 B 组，第二天在贼所在的组继续进行二分，第三天继续，这样第九天就可以找到那个贼（2^9=512）。

>还有提出优化方案的，分三组，每次对其中两组发两张不同照片，或者分四组，极端情况下分 500 组，发 499 张不同照片，当天就可以找出贼。

妹子大喜。

妹子最后使用四分查找法在第5天就找到了那个贼，痛骂了一番，然后怒删之。

作为程序员的你对 **二分查找算法** 一定很熟悉了，不过你知道为什么在算法实现中我们倾向于使用二分查找而不是三分、四分来加快速度呢？ 类似的比如经典的归并排序、快速排序，都是采用分治思想的二分法来实现的。

为了回答这个问题，我们先使用 Python 实现一下二分查找算法:

查找给定元素t，我们先在数据序列（0...n）的中间位置mid查找，如果找到就返回mid，如果中间元素比给定元素大，就在(0..mid-1)的序列中再次进行二查找，否则就在(mid...n)的序列中再二分查找，直到最后的序列区间变为1，此时仍未找到就返回 -1，下面是使用递归和循环的两种方法。

#### 二分查找 - 循环方式
```python
ef binarySearch(sorted_data_list,tartget):
    '''
    循环方式实现二分查找
    '''
    length = len(sorted_data_list)
    low = 0
    high = length -1 
    while low <= high:
        mid =  low + (high-low)//2 #之所以不用 (low+hight)//2 是为了防止之和数据大而溢出
        if sorted_data_list[mid] == tartget:
            return mid
        elif sorted_data_list[mid] > tartget:
            high = mid - 1 
        else:
            low = mid + 1
    #未找到就返回 -1 
    return -1
```
这段代码很简单，但是有**两个容易出错**的地方，一处是

```python
mid =  low + (high-low)//2
```
是为了大数相加的值太大而溢出，不论你所用的编程语言是否有这种溢出检查，建议都这样编写，养成良好习惯。还有一处是 while 循环退出条件是 low <= high 而不是 low < high，否则可能造成返回错误的结果。二分查找算法也很容易改写递归：

#### 二分查找 - 递归方式
```python
def binarySearch2(low,high,sorted_data_list,tartget):
    '''
    递归实现二分查找
    '''
    if low > high:
        return -1
    mid =  low + (high-low)//2 #之所以不用 (low+hight)//2 是为了防止之和数据大而溢出
    if sorted_data_list[mid] == tartget:
        return mid
    elif sorted_data_list[mid] > tartget:
        return binarySearch2(low,mid-1,sorted_data_list,tartget) 
    else:
        return binarySearch2(mid+1,high,sorted_data_list,tartget) 
```
运行结果如下所示：

```python
if __name__ == "__main__":
    sorted = [1,3,5,8,23,29,344,1113]
    print(sorted)
    print(f"29 的下标：{binarySearch(sorted,29)}")
    print(f"1 的下标：{binarySearch2(0,len(sorted),sorted,1)}")
```
输出如下结果
```python
[1, 3, 5, 8, 23, 29, 344, 1113]
29 的下标：5
1 的下标：0
```
接下来我们来实现下 3 分查找，3 分查找可以认为分成 3 个区间来查找，递归实现如下所示：

#### 三分查找算法
```python
def binarySearch3(low,high,sorted_data_list,tartget):
    '''
    递归实现三分查找
    '''
    if low > high:
        return -1
    mid1 =  low + (high-low)//3 
    mid2 =  high - (high-low)//3 
    if sorted_data_list[mid1] == tartget:
        return mid1
    elif sorted_data_list[mid2] == tartget:
        return mid2
    elif sorted_data_list[mid1] > tartget:
        return binarySearch3(low,mid1-1,sorted_data_list,tartget) 
    elif sorted_data_list[mid1] < tartget < sorted_data_list[mid2]:
        return binarySearch3(mid1+1,mid2-1,sorted_data_list,tartget) 
    else:
        return binarySearch3(mid2+1,high,sorted_data_list,tartget) 
```
三分查找算法的时间复杂度为 logn 以 3 为底的对数。  

对比二分查找和三分查找的 if else 的代码行数就可以看出，三分查找算法的比较次数更多，是 5 次，而二分查找算法只需要 3 次，同理可以推导出 4分查找算法是 7 次，5 分查找算法是 11 次。 

n 分查找算法 (n >=3)，看上去比二分查找算法的时间复杂度低，但是由于比较次数的增多，算法在运行的过程中保存更多的中间变量，在更多分支递归时CPU也需要保留更多的现场，因此实际使用时这些查找算法的耗时反而会比较高，同时也会增加代码的复杂度。类似的，为什么用二叉树而不是三叉、四叉树。



系列文章：

[工作后，为什么还要学习数据结构与算法](https://somenzz.github.io/2018/12/09/algorthms/)
[Python-排序-冒泡排序-优化](https://somenzz.github.io/2018/12/10/python_bubble_sort/)
[Python-排序-选择排序-优化](https://somenzz.github.io/2018/12/12/python_select_sort/)
[Python-排序-插入排序-优化](https://somenzz.github.io/2018/12/15/python_insert_shell_sort/)
[Python-排序-归并排序-哨兵的妙用](https://somenzz.github.io/2018/12/17/python_merge_sort/)
[Python|算法|快速排序|如何在O(n)查找第K大元素](https://somenzz.github.io/2018/12/18/python_quick_sort/)
[Python-排序-有哪些时间复杂度为O(n)的排序算法？](https://somenzz.github.io/2018/12/25/python_linear_sort/)




（完）

加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat2.jpg)
