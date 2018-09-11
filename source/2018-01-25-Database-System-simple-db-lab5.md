---
title: 'Database System: simple-db lab5'
date: 2018-01-25 23:41:51
tags: simple-db
---

整个lab6都是关于B+树索引的内容。包括MySQL在内的很多数据库以及操作系统中的文件系统的储存结构都把B+树当做底层数据的储存结构。

### 什么是B+树

![images](./images/img_for_2018_09/B+Tree_demo.png)

B+树与二叉树类似，是树形数据结构的一种，B+树是一个多(叉)分枝树，是B树的一种改进型数据结构。

与B树的区别: 将B树的相邻叶子节点通过双向指针互联起来，并且内部节点不包含具体的数据值，内部节点仅做数据区间划分，用于辅助数据查找，而不包含具体数据。而B树在1972年被发明的时候，并不区分这些，key+value被存放在所有节点中，并不区分内部节点和外部节点。

关键特性:
* Self-Balance 自平衡，每一个叶子结点的高度是相同的
* 每一个内部节点(除Root节点外)，其余节点都最起码有一半是有数据的
* 每一个内部节点，如果有k个阈值划分点(keys)，则对应有k+1个子节点(children)

![images](./images/img_for_2018_01/BTree_Overview.jpg)

### 为什么用B+树(该数据结构体的特性)

根本的原因在于目前数据的持久化存储介质还是机械磁盘（寻道寻址磁道柱头那种），数据的访问的时间消耗很大。为了更快速的访问磁盘数据，提升磁盘IO的使用效率（磁盘IO性能你是提升不了的，软件能改的就是从使用的角度提升使用效率，所以我这里特意强调了是使用效率），人们通过将一部分数据查询的全局概要关键信息加载到RAM中，而实际的全体数据还是放在磁盘上，通过RAM中的快速定位查找到目标数据，然后仅需要从磁盘获取目标数据页即可，而不需要扫描整个磁盘。这，就是出发点。

而B+树在实际使用的过程中，内部节点几乎都会被加载到RAM中，在InnoDB中，3层高度的B+树即可储存百万级别的数据规模。后面我会详细介绍，B+树如何做到的。

![images](./images/img_for_2018_01/BTree_InPractice.jpg)

B+树的节点分为两个类型:

Internal 内部节点
Leaf 叶子节点
BTreeInternalPage的内存布局: 
Parent Pointer 4byte | child category 1byte | headers | keys | children | padding

keys == fields 即区间划分阈值， 实际上 keys申请的多内空间多了一个，而且多了的那个空着。
numSlots == keys 和 children 数组实际的数量。 而keys[0] = null

children 数组实际记录的是 pageNo，即页码数
keys 数组实际记录的是划分阈值的具体的值

BTreeLeafPage 的内存布局：
Parent Pointer | Left Sibling | Right Sibling | headers | tuples |

每个Tuple占用的内存大小 = tuple本身的大小 * 8 bits + 1bits的header中占用的大小
叶子节点最大tuple的数量: (数据页的大小 * 8 bits - （左右叶子节点指针+父指针）* 8 bits ) / 每个Tuple占用的内存大小 向下取整

### 什么是索引？
索引是原数据表某些列的数据拷贝，以一种更便于/有效的数据访问方式组织起来的一份拷贝(replication)。DBMS保证索引列中的数据和原数据表中的数据完全一致。

DBMS的的工作之一（查询优化器）就是找出某个表的几个索引之中谁最适合执行查询计划，效率最优。

索引是不是越多越好呢？当然不是，索引的本质是数据的冗余，一张表有过多的索引会为DBMS带来性能上的负担，DBMS为了保持数据一致性，在并发数据更新和访问的时候不得不保证多份冗余数据的一致性，冗余数据越多，维护的开销也就越大。从储存和维护两个角度出发，索引多不是越多越好。


Reference:
* [15445 Lab5 实验指导书](https://github.com/penguiner/course-info-2017/blob/master/lab5.md)
* [B树的动态插入删除demo](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html)