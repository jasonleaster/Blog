---
title: 'Database System: simple-db lab1'
date: 2018-01-21 12:12:01
tags: Database, MIT
---

[MIT 6.830](http://db.csail.mit.edu/6.830/) 课程主要旨在向不了解数据的童鞋介绍数据库基础，顺便伴随着课程实验，自己体会一下如何实现一个简易的数据库系统simple-db。由于工作中会频繁使用到MySQL数据库，常常会有一些技术性的疑问无法得到深入的解答，故尝试通过这个lab加深自己对于数据库的理解。完成实验后受益匪浅，接下来我会用几篇文章，尽量简要的记录实验想要表达的数据库基础核心要点。

通过这个课程我获得的收益:
* 更加深入的理解事务，以及Java的并发控制，如何避免并发死锁，如何监控并打破死锁
* 理解基于事物日志的容灾恢复和回滚
* 在工作中通过分析执行计划能够快速的定位SQL的性能问题并提出相应的解决方案。

<!-- more -->

在此之前我已经阅读了
* `<<MySQL技术内幕 SQL编程>>` by  姜承尧
* `<<High Performance MySQL>>` by Baron Schwartz等  

在读:  
* `<<数据库系统实现>>`
仅供参考，从我个人体验角度来谈的话，如果完全没有任何数据库应用基础和Java编程基础的话，可能不适合尝试该课程。

----
### 数据库基础概念

__什么是数据库?__
- 结构化的数据集合
- 一般以一条条“记录”(records)的组织形式存在于存储器上(一般为磁盘)，并且这些记录之间的关系也会被记录和使用。

__为什么需要数据库？__
数据库的诞生是为了`数据模型`与`上层业务软件逻辑`解耦，以数据模型为应用系统的底层基石，数据持久层对上层应用暴露访问接口即可，通过分层，模块化，解耦构建成一个单独的数据维护系统即DBMS，来完成对数据模型的管理，实现“Data Independence”。并且大多数数据库是支持事务控制的，事务ACID的特性便能够使得上层应用代码的编写更加简洁。  

数据库带来的好处:
* 更高效的数据访问和数据更新
* 容灾恢复
* 数据的一致性

__那什么又是数据的独立性 (Data Independence)呢?__
> Decoupling of logical model from physical representation is known as “data independence” Can store the data in different ways on disk, don’t have to change program.

关系模型: 记录了对象(记录record)与他们属性(column)之间的关系，即数据表就是关系模型。

> Why is it called relational?
because each record is a relation between fields (“keys” capture relations)

一个比较典型的数据表demo:

<img src="/images/img_for_2018_01/student-table.jpg" width="50%" height="300" style="display:block; margin:0 auto;"/>

上面的表中有三列数据，id, name, address, 从面相对象的角度来看，这个表就像一个类，而其中的列数据，就对应类的实例。每一列就像一个类的Field或者Attribute。Field是抽象的，它可能是Int类型的，也可能是字符串类型的，或者其他系统支持的数据类型。


上面的话没有一行代码，但是都是深刻的体会。

### 谈一谈关系模型

> Those who cannot learn from history are doomed to repeat it.  -- George Santayana

在关系模型出现之前，数据库应用的开发和维护是很麻烦的一件事件，代码和数据耦合的非常紧。

前段时间有朋友抛出了一些关于数据库的疑问，对于关系型数据库中“关系”一词如何理解上一小节已经做了描述。那么为什么要用关系型数据库呢？它会带来哪些益处呢？这是历史选择的结果。

1970年代的时候，那时数据库慢慢兴起，那时关于如何定义数据模型有两个阵营，一个是以Codd为代表的关系型阵营，另一个是网络型的。核心辩论的要点有以下四点:

* Data redundancy (or how to avoid it)
* Physical data independence (programs don't embed knowledge of physical structure)
* Logical data independence (logical structure can change and legacy programs can continue to run)
* High level languages

计算机行业很多都是实现优先于标准先出来，先实现了再尝试标准化，“Talk is cheap, show me your code”。四五十年前，这两大阵营都觉得对方的想法不好实现，但是IBM的[DB2](https://en.wikipedia.org/wiki/IBM_Db2)数据库发布结束了关于数据库模型的讨论，DB2支持所有关系模型的特性，得到市场认可。其数据模型主要有以下特点:
* 所有的数据以`records` 或 `tuples`形式表现。
* 数据表是`无序集合`
* 数据库由一个或多个数据表组成
* 每个关系数据实体都有对应的“schema”用来描述这些数据实体中不同的列(columns/fields)的类型。
* 每一个field是一种基础数据类型(int, char 等等), 不能再是数据表(此处我理解为不支持递归定义)。

![images](/images/img_for_2018_01/tupleModel.jpg)

上图的左右两侧分别展示了simple-db中的tuple模块和常见的数据库建表语句，不难看出两者之间的联系和映射关系体现了关系型数据库表的特点。

__所谓“关系”，是指包含属性关系实体对象的**无序集合**。看到这里，体会至此，应该慢慢会有感受，所谓的关系型数据库，更深层次的其实是面向集合的数据库，它强调的核心是把数据当做数学集合(set)来看待，对数据表的操作是对集合的操作。__

关系模型中三个重要的概念:
* Candidate Keys: 候选键?反正每个关系(表)中必须至少有一个key能够`唯一`的定位这个记录对象, 即一一映射的关系。否则会违反关系模型。
* Primary Key: 主键，其实就是Candidate Key的一种，用来映射表中记录。一些数据库再实在是找不到已有Candidate Key的时候，会自动默认生成一个内部Id，用来当做主键，MySQL就是如此，﻿ε≡٩(๑>₃<)۶ 。一个集合，如果出现重复的数据，这个数据集就违反了集合(Set)本身的固有属性(数据唯一性)，集合论就玩不转了。
* Foreign Key: 即外键。外键指明从一个关系(表 relation/Table)的属性必须能够映射到另外一个关系(表)的记录实体(Tuple/Record)。

再谈谈非关系型的数据库，下图介绍了常见的主流数据库数据模型，其中蓝色框出的即Non-SQL非关系型数据库，典型代表如Redis(Key-Value), MongoDB(Document), HBase(Column-Family)等等，这里推荐看看CMU的15445中对于[关系代数](http://15445.courses.cs.cmu.edu/fall2017/slides/02-relationalalgebra.pdf)一节的介绍文档。

<img src="/images/img_for_2018_01/dataModels.jpg" width="30%" height="150" style="display:block; margin:0 auto;" />

本节最后总结一下，关系模型怎么解决历史上两大阵营的核心辩论要点的:
* 引入集合论，利用集合的特点实现数据去重
* 引入关系型查询语言，用户通过查询语言与数据库交互，以查询语言为‘接口’，实现了分层解耦，进一步实现数据独立性，用户不再关心底层实现即可快速完成数据库应用的开发。

### Simple-DB 架构图

TODO :)
<!-- ![images](/images/img_for_2018_01/ArchitectureOfDBMS.jpg) -->

### lab1 注意事项

lab1旨在介绍关于simple-db的一些基础概念，让参与者对Fileds, Tuple, BufferPool, Catalog, HeapFile 等等这些经常会用到的类进行介绍并设置少量的程序任务，让参与者慢慢熟悉源代码。在lab1的最后会让参与者去实现一个序列化读文件的操作符SeqScan。

其实此处实验的安排者希望通过先让读者接触基础数据的概念和格式，以及数据在系统中如何存储的，来让参与者循序渐进的熟悉数据库。在simple-db中，最基础的概念就是Tuple和Fields，数据对象以Tuple的实例存储在数据页Page当中，目前具体的实现有HeapPage，即一种很普通无序化堆式存储方式。接着把这些HeapPage中关键的信息通过 
_HeapPage#getPageData()_
方法返回字节块的形式，通过Java的文件IO相关方法储存在磁盘文件中。No Magic :)

-----
附:  

* Java编程推荐Stanford CS108  
* [6.830 课程讲义](http://db.csail.mit.edu/6.830/notes.php)  
* 值得一提的是Berkeley的CS186和6.830的实验内容几乎是一样的，都是实现simple-db  
* [宾夕法利亚州立大学的数据库管理课程 431W Database Management Systems](http://www.cse.psu.edu/~yul189/cmpsc431w/lectures.html)
* 如果有大量空闲时间推荐食用这两个课程的Video, 口味更佳~   

<!-- * [CMU 15445 Relational Algebra](http://15445.courses.cs.cmu.edu/fall2017/slides/02-relationalalgebra.pdf)   -->

-----
