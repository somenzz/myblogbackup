---
layout: post
title: "二叉树的Python实现"
date: 2019-01-23 20:38:57
comments: true
reward: true
tags: 
	- Python
	- 二叉树
---


学过数据结构的同学一定对树这种数据结构非常熟悉了，树是一种非常高效的非线性存储结构，学好树对理解一些复杂的算法非常有帮助。
<!-- more -->
树有以下内容需要掌握：
- 树、二叉树
- 二叉查找树
- 平衡二叉查找树、红黑树
- 递归树

作为一名 Python 程序员，如果把基础的数据结构与算法都自己亲自实现一遍，那么你已经比 90% 的 Python 程序员更优秀了。

今天的我们的目标是使用 Python 来实现一棵二叉树。 二叉查找树、平衡二叉查找树、红黑树、递归树后面也会实现，请保持关注。

先来梳理下概念：树，可以很形象的理解，有根，有叶子，对应在数据结构中就是根节点、叶子节点，同一层的叶子叫兄弟节点，邻近不同层的叫父子节点，非常好理解。

二叉树，就是每个节点都至多有二个子节点的树。

满二叉树，就是除了叶子节点外，每个节点都有左右两个子节点，这种二叉树叫做满二叉树。

完全二叉树，就是叶子节点都在最底下两层，最后一层叶子节都靠左排列，并且除了最后一层，其他层的节点个数都要达到最大，这种二叉树叫做完全二叉树。

二叉树即可以使用链式存储，也可以使用数组来存储，而完全二叉树是使用数据存最省内存的一种结构。

接下来我们使用 Python 实现链式存储的二叉树。

思路：
1、先定义一个节点 node 类，存储数据 item 和左右节点指针
2、再实现二叉树 binary_tree 的类，类应至少有以下属性和方法：
属性：有一个根节点（root) , 它是 node 类。

方法：
- 插入一个元素（逐层向下插入）。
- 删除一个元素（按照二叉查找树的方式删除，先查找待删除节点的父节点。 如果父节点不为空， 判断 item 的左右子树，如果左子树为空，那么判断 item 是父节点的左孩子，还是右孩子，如果是左孩子，将父节点的左指针指向 item 的右子树，反之将父节点的右指针指向 item 的右子树，如果右子树为空，那么判断 item 是父节点的左孩子，还是右孩子，如果是左孩子，将父节点的左指针指向 item 的左子树，反之将父节点的右指针指向 item 的左子树，如果左右子树均不为空，寻找右子树中的最左叶子节点 x ，将 x 替代要删除的节点。 ，如果父节点为空，说明不存在该元素 ，删除元素，返回 True ，未删除元素, 返回 False。
- 获取一个元素的父节点。
- 层序遍历：按层输出二叉树的元素
- 中序遍历：先左子树，再根节点，最后右子树
- 前序遍历：先根节点，再左子树，最后右子树
- 后序遍历：先左子树，再右子树，最后根节点。

按下来就是编程实现了，请动手自己实现一把。

先实现一个 node 类：

```python

class Node(object):
    def __init__(self, item):
        self.item = item
        self.left = None
        self.right = None

    def __str__(self):
        return str(self.item)
```
这里的 \_\_str\_\_ 方法是为了方便打印。在 print 一个 Node 类时会打印 \_\_str\_\_ 的返回值,例如：print(Node(5)) 就会打印出字符串 5 。

实现一个二叉树类：

```python
class Tree(object):
    def __init__(self):
        # 根节点定义为 root 永不删除，做为哨兵使用。
        self.root = Node('root')

    def add(self, item):
        node = Node(item)
        if self.root is None:
            self.root = node
        else:
            q = [self.root]

            while True:
                pop_node = q.pop(0)
                if pop_node.left is None:
                    pop_node.left = node
                    return
                elif pop_node.right is None:
                    pop_node.right = node
                    return
                else:
                    q.append(pop_node.left)
                    q.append(pop_node.right)

    def get_parent(self, item):
        '''
        找到 Item 的父节点
        '''
        if self.root.item == item:
            return None  # 根节点没有父节点
        tmp = [self.root]
        while tmp:
            pop_node = tmp.pop(0)
            if pop_node.left and pop_node.left.item == item:
                return pop_node
            if pop_node.right and pop_node.right.item == item:
                return pop_node
            if pop_node.left is not None:
                tmp.append(pop_node.left)
            if pop_node.right is not None:
                tmp.append(pop_node.right)
        return None

    def delete(self, item):
        '''
        从二叉树中删除一个元素
        先获取 待删除节点 item 的父节点
        如果父节点不为空，
            判断 item 的左右子树
            如果左子树为空，那么判断 item 是父节点的左孩子，还是右孩子，如果是左孩子，将父节点的左指针指向 item 的右子树，反之将父节点的右指针指向 item 的右子树
            如果右子树为空，那么判断 item 是父节点的左孩子，还是右孩子，如果是左孩子，将父节点的左指针指向 item 的左子树，反之将父节点的右指针指向 item 的左子树
            如果左右子树均不为空，寻找右子树中的最左叶子节点 x ，将 x 替代要删除的节点。
        删除成功，返回 True
        删除失败, 返回 False

        '''
        if self.root is None:  # 如果根为空，就什么也不做
            return False

        parent = self.get_parent(item)
        if parent:
            del_node = parent.left if parent.left.item == item else parent.right  # 待删除节点
            if del_node.left is None:
                if parent.left.item == item:
                    parent.left = del_node.right
                else:
                    parent.right = del_node.right
                del del_node
                return True
            elif del_node.right is None:
                if parent.left.item == item:
                    parent.left = del_node.left
                else:
                    parent.right = del_node.left
                del del_node
                return True
            else:  # 左右子树都不为空
                tmp_pre = del_node
                tmp_next = del_node.right
                if tmp_next.left is None:
                    # 替代
                    tmp_pre.right = tmp_next.right
                    tmp_next.left = del_node.left
                    tmp_next.right = del_node.right

                else:
                    while tmp_next.left:  # 让tmp指向右子树的最后一个叶子
                        tmp_pre = tmp_next
                        tmp_next = tmp_next.left
                    # 替代
                    tmp_pre.left = tmp_next.right
                    tmp_next.left = del_node.left
                    tmp_next.right = del_node.right
                if parent.left.item == item:
                    parent.left = tmp_next
                else:
                    parent.right = tmp_next
                del del_node
                return True
        else:
            return False

    def traverse(self):  # 层次遍历
        if self.root is None:
            return None
        q = [self.root]
        res = [self.root.item]
        while q != []:
            pop_node = q.pop(0)
            if pop_node.left is not None:
                q.append(pop_node.left)
                res.append(pop_node.left.item)

            if pop_node.right is not None:
                q.append(pop_node.right)
                res.append(pop_node.right.item)
        return res

    def preorder(self, root):  # 先序遍历
        if root is None:
            return []
        result = [root.item]
        left_item = self.preorder(root.left)
        right_item = self.preorder(root.right)
        return result + left_item + right_item

    def inorder(self, root):  # 中序序遍历
        if root is None:
            return []
        result = [root.item]
        left_item = self.inorder(root.left)
        right_item = self.inorder(root.right)
        return left_item + result + right_item

    def postorder(self, root):  # 后序遍历
        if root is None:
            return []
        result = [root.item]
        left_item = self.postorder(root.left)
        right_item = self.postorder(root.right)
        return left_item + right_item + result
```

运行测试：

```python
if __name__ == '__main__':
    t = Tree()
    for i in range(10):
        t.add(i)
    print('层序遍历:', t.traverse())
    print('先序遍历:', t.preorder(t.root))
    print('中序遍历:', t.inorder(t.root))
    print('后序遍历:', t.postorder(t.root))

    for i in range(10):
        print(i, " 的父亲", t.get_parent(i))

    for i in range(0, 15, 3):
        print(f"删除 {i}", '成功' if t.delete(i) else '失败')
        print('层序遍历:', t.traverse())
        print('先序遍历:', t.preorder(t.root))
        print('中序遍历:', t.inorder(t.root))
        print('后序遍历:', t.postorder(t.root))
```

注意这里使用了根节点 root 作为哨兵，永不删除，简化了 delete 函数的实现，这样也最不容易出错，否则还要判断待删除的节点是否是根节点。

执行结果如下：

```python
层序遍历: ['root', 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
先序遍历: ['root', 0, 2, 6, 7, 3, 8, 9, 1, 4, 5]
中序遍历: [6, 2, 7, 0, 8, 3, 9, 'root', 4, 1, 5]
后序遍历: [6, 7, 2, 8, 9, 3, 0, 4, 5, 1, 'root']
0  的父亲 root
1  的父亲 root
2  的父亲 0
3  的父亲 0
4  的父亲 1
5  的父亲 1
6  的父亲 2
7  的父亲 2
8  的父亲 3
9  的父亲 3
删除 0 成功
层序遍历: ['root', 8, 1, 2, 3, 4, 5, 6, 7, 9]
先序遍历: ['root', 8, 2, 6, 7, 3, 9, 1, 4, 5]
中序遍历: [6, 2, 7, 8, 3, 9, 'root', 4, 1, 5]
后序遍历: [6, 7, 2, 9, 3, 8, 4, 5, 1, 'root']
删除 3 成功
层序遍历: ['root', 8, 1, 2, 9, 4, 5, 6, 7]
先序遍历: ['root', 8, 2, 6, 7, 9, 1, 4, 5]
中序遍历: [6, 2, 7, 8, 9, 'root', 4, 1, 5]
后序遍历: [6, 7, 2, 9, 8, 4, 5, 1, 'root']
删除 6 成功
层序遍历: ['root', 8, 1, 2, 9, 4, 5, 7]
先序遍历: ['root', 8, 2, 7, 9, 1, 4, 5]
中序遍历: [2, 7, 8, 9, 'root', 4, 1, 5]
后序遍历: [7, 2, 9, 8, 4, 5, 1, 'root']
删除 9 成功
层序遍历: ['root', 8, 1, 2, 4, 5, 7]
先序遍历: ['root', 8, 2, 7, 1, 4, 5]
中序遍历: [2, 7, 8, 'root', 4, 1, 5]
后序遍历: [7, 2, 8, 4, 5, 1, 'root']
删除 12 失败
层序遍历: ['root', 8, 1, 2, 4, 5, 7]
先序遍历: ['root', 8, 2, 7, 1, 4, 5]
中序遍历: [2, 7, 8, 'root', 4, 1, 5]
后序遍历: [7, 2, 8, 4, 5, 1, 'root']
```


加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat2.jpg)
