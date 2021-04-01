---
type : posts
title : 数据结构与算法 - 线性表 - 链表
categories: [数据结构与算法,] 
series: [数据结构与算法笔记,]
date : 2019-12-18
url: /posts/2019-12-18-link.html 
tags : [数据结构与算法, 链表]
---

> 《数据结构与算法-王争》学习笔记，记录备查

## 基本概念

链表，通过“指针”将一组凌乱的内存块串联起来使用的数据结构。

## 链表的分类

### 单链表

**构成**

```
-> <data|next> -> <data|next> -> <data|next> -> NULL 
```
- 节点，内存块
- 后继指针，记录下个节点地址的指针
- 头节点，第一个节点
- 尾节点，最后一个节点。最后一个节点的指向是空地址NULL。

**查找**

因为链表的地址空间非连续，无法像数组一样根据下标寻址公式来计算地址，只能从头挨个遍历。查询某值的时间复杂度都是O(n)。

**插入**

插入时，我们只需关注节点的指针指向即可，所以时间复杂度为O(1)。

**删除**

删除时，同插入，移动节点指针即可。时间复杂度为O(1)（此处，并没有算节点的查找所花费的时间，仅为删除操作。如果加上查找，则时间复杂度为O(n)）。

### 循环链表

循环链表，是一种特殊的单链表，即最后一个节点的后继指针保存了头结点的内存地址。大致如下：

```
        _______________________________________ 
       ↓                                      |
-> <data|next> -> <data|next> -> <data|next> --
```

方便查找头节点，其他操作时间复杂度同单链表。

### 双向链表

顾名思义，双向链表，除了后继指针外，还有前驱指针保存前一个节点的内存地址。大致如下：

```
-> <prew|data|next> <-> <prew|data|next> <-> <prew|data|next>
```

**查找**

因为双向链表支持双向遍历，在需要查找前驱节点的连续操作时，时间复杂度可以降为O(1)。

**插入**

插入时同单链表，只需关注指针指向即可，时间复杂度为O(1)。

**删除**

双向链表和单链表一样，单说删除操作时，仅仅是移动了节点的指针，时间复杂度为O(1)。但是，删除操作时，我们为了操作指针，需要知道删除节点的前一个节点。单链表和双向链表略有不同：

- 对于单链表，则需要从头遍历，找到后继指针是我们删除节点地址的节点。时间复杂度为O(n)。
- 对于双向链表，则可直接通过前驱指针来获取前驱节点，时间复杂度为O(1)。

### 双向循环链表

双向循环链表，就是把循环链表和双向链表结合起来的一种数据结构。

双向链表和单链表相比，多了存储前指针的空间，但是他的操作时间复杂度确大大减少。这便是**空间换时间**的思想，在实际的生产中，我们要根据实际资源情况来合理的使用**空间换时间**或**时间换空间**的思想。

## 应用

### 数组和链表的性能对比

![](/static/imgs/complexity/link.jpg)

- 数组内存空间连续，对CPU的缓存机制友好，CPU可以于都数组的数据，提高访问效率。链表内存不连续，无法连用CPU的缓存机制提高访问效率。
- 数组内存固定，扩容耗时严重。链表没有大小限制，可随意动态扩容。

### 链表应用

- 链表适合插入、删除操作频繁的业务场景
- 双向链表比单链表的插入、删除简单、高效。
- 循环链表合适要处理的数据具有环形结构特点，如约瑟夫问题。

## 常面试点及代码编写技巧

### 考点

- 单链表的反转
- 链表中环的检测。
- 连个有序链表合并。
- 删除链表倒数第N个节点。
- 求链表的中间节点。

### 链表编写技巧

- 理解指针和引用的含义：指针和引用都存储了所致对象的内存地址。**将某个变量赋值给指针，实际上就是将这个变量的地址赋值给指针，或者反过来指针中存储了存储了这个变量的内存地址，指向了这个变量，通过指针就能找到这个变量。**

- 警惕指针丢失和内存泄露：插入节点时，注意操作的顺序；删除节点时，记着释放内存空间。
- 多用「哨兵节点」简化难度：链表的头节点和尾节点的操作时，哨兵节点可以简化操作。哨兵节点不存储数据。
  - 带头链表：有哨兵节点的链表
  - 不带头链表：无哨兵节点的链表

- 留意边界条件处理：几个常见的边界情况
  - 空链表
  - 只包含一个结点的链表
  - 只包含两个结点的链表
  - 代码逻辑在头结点和尾结点是否正常

- 编写代码时，举例画图可以帮助思考。

## 参考

- 常见的缓存淘汰策略：
  - FIFO（First In，First Out）
  - 最少使用策略 LFU（Least Frequently Used）
  - 最近最少使用策略 LRU（Least Recently Used）