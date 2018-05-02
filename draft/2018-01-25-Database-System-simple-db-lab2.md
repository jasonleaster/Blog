---
title: 'Database System: simple-db lab2'
date: 2018-01-25 23:34:32
tags: Database, MIT
---

lab2 的核心工作就是为simple-db实现关系代数运算类，并实现一些聚合函数的功能。

我个人是比较讨厌在查询语句中写一些聚合函数达到数据获取的需求，我不喜欢数据库所在服务器做额外更多的CPU密集型的任务，我希望数据库全力集中在数据IO上就好，计算的事情交给上层的应用去做。一旦没有约束，有些开发者喜欢写大串复杂难以维护的SQL计算逻辑。~~灭了他的心都有 trouble maker~~

### 关系代数

我并不想把关系代数解释的很复杂，简单的说它就是一个基于集合的关系运算工具，运算作用于一个或者多个关系上来生成一个关系。

<img src="/images/img_for_2018_01/RelationalAlgebra.jpg" width="70%" height="250" style="display:block; margin:0 auto;"/> 

上图介绍了关系代数的七大操作。可以说所有SQL查询逻辑的“内核”都是关系代数的运算，下图介绍了一条简单的SQL查询语句的关系代数运算图。

<div>
<img src="/images/img_for_2018_01/queryPlainDemo.jpg" width="70%" height="250" style="display:block; margin:0 auto;"/>
<span style="display:block; margin:0 200px;">执行两个表A, B的join运算，筛选出两表id相等</span>
</div>

如果写过很多SQL，比较熟悉MySQL的同学来说，基本不会对关系代数有困难，不熟悉基础概念的话出门右转Wikipedia。

再介绍一下普通的product和join的区别，看CMU的ppt

实验最后介绍了如何利用现有数据加载到SimpleDB并执行简单的查询语句。Parser部分的核心功能是借用了ZQL的第三方库，sql的解析基本不用操心，作者将重点放在了如何将逻辑查询转换成物理执行计划。

### 函数依赖、范式化、反范式化

聊这三个话题背后的共同核心点是“如何创建一个好的数据库表呢？”

“好”？没有绝对的好，很多都是根据具体需求而来，好的数据表主要从**存储**和**计算**两个方面去刻画“好的数据表”:

* 没有数据冗余，或尽可能少的数据冗余
* 性能表现优异

首先谈谈数据冗余，如下图展示的学生课程信息和得分的表

sid  | cid    | room     | grade | name  | address
---- | ------ | -------- | ----- | ----- | -------
123  | 15-445 | GHC 6115 | A     | Andy  | Pittsburgh
456  | 15-721 | GHC 8102 | B     | Tupac | Los Angeles
789  | 15-445 | GHC 6115 | A     | Obama | Chicago
012  | 15-445 | GHC 6115 | C     | Waka  | Flocka Atlanta
789  | 15-721 | GHC 8102 | A     | Obama | Chicago

(cid, room) 的第1 3 4 列, (name, address) 的 第4 5 列，明显就是重复的数据了，如果这样的数据很多的话就容易带来问题，数据插入，更新，删除操作很麻烦，一个值的修改涉及到多个了数据列的修改，此外数据冗余很多的话，磁盘空间也被大量重复数据占用，存储效率不高浪费资源。

解决的方法就是表的垂直拆分。

<img src="/images/img_for_2018_01/demoCompositedStudentTable.jpg" width="70%" height="250" style="display:block; margin:0 auto;"/> 

如上图，当更新某一特定学生的地址信息时，只需要更新student表中的一条记录即可，不需要更新其他的冗余信息记录列。

让我们思考一个问题，在初始未拆分的表中，学生id(sid)能够唯一确定名称和地址信息(name, address), 但是却不能确定(cid，room)的组合。即我们能够得到sid与(name, address)存在一一映射的关系，sid与(cid, room)却不是一一映射的。

我们这里说的映射就像确定性函数一样，对于同样的输入，函数的输出是相同的，保障我们使用某一个值(key值)作为输入时，能够得到唯一与之匹配的输出值(数据列)。这就保障了不会数据冗余，函数依赖(Functional Dependency)的概念由此引出。

> 定义: X -> Y. X 的值能够函数式的确定Y的值，我们成Y依赖于X。

其意图在于说明，不存在任意两条记录，它们在X属性（或属性组）上的值相同，而在Y属性上的值不同。
如果一个数据表，所有的数据列都能通过key列(其实就是主键)唯一确定的话，这个表就没有数据冗余。所以解决数据冗余思路就是尽可能的找到数据冗余的列，然后寻找其函数依赖关系，进而进行数据表的垂直拆分。需要注意的是，拆分后的表必须能够通过关联字段列JOIN的方式，无损的重构回原来的表，不能出现JOIN之后出现了原来没有的数据。

函数依赖分`完全函数依赖`和`部分函数依赖`两种。

* 完全函数依赖: 在一张表中，若 X → Y，且对于 X 的任何一个真子集（假如属性组 X 包含超过一个属性的话），X ' → Y 不成立，那么我们称 Y 对于 X 完全函数依赖，记作 X F→ Y。

* 部分函数依赖: 假如 Y 函数依赖于 X，但同时 Y 并不完全函数依赖于 X，那么我们就称 Y 部分函数依赖于 X，记作 X  P→ Y。


说完FD，接着讲讲`范式`，即Normal Form, 我理解为其实就是表通常的样子。我们可以根据好坏程度对表通常的样子进行分级，即所谓的范式等级，第一范式，第二范式等等。大多数人在开发中一般只会用到第三范式，再高等级的范式基本上不会用到了。通过这些经验范式，我们能够构建“更好的数据表”。

节操再低，最起码第一范式不能违背。

<img src="/images/img_for_2018_01/FirstNormalForm.jpg" width="70%" height="250" style="display:block; margin:0 auto;"/> 

第一范式要求数据表的所有列必须是“原子的”，你不能在某一个列里面储存重复的数据。

第二范式则要求数据表里的所有数据都要和该数据表的键（主键与候选键）有完全依赖关系。原话是"1NF and non-key attributes fully depends on the candidate key"。如果有哪些数据只和一个键的一部分有关的话，就得把它们独立出来变成另一个数据表，得拆分。关键在于是否存在非主属性对于键的部分函数依赖，如果存在则不符合第二范式，反之则符合

例如最初提到的学生课程表的例子，如果把sid当做candidate ket，上课的房间room和学生sid是不存在依赖关系，而且room和课程cid有依赖关系，因此为了避免数据冗余，我们就得把对cid有依赖的属性拆分出来，单独构成一个数据表维护起来。

关于第三范式，其在第二范式的基础上要求，不能存在非主属性对于码的传递函数依赖。

<img src="/images/img_for_2018_01/Not_Valid_3NF_Example.jpg" width="70%" height="250" style="display:block; margin:0 auto;"/> 

上图中的表就是不满足3NF的demo，A->B, B->C, A 与 C有传递依赖的关系。要解决上面传递函数依赖的问题，需要对原始表根据依赖关系垂直拆分，并把传递依赖关系单独抽取出来，构建一个表，以此记录传递依赖关系，要达到无损join(lossless join)的话，需要三表同时JOIN。

既然范式化这么好，但是为什么当下会有一些数据库才有反范式化的设计思路呢？这就是计算和存储之间的博弈和考量。

在alibaba的Java技术规范中明确要求道:

>  【强制】超过三个表禁止 join。需要 join 的字段，数据类型保持绝对一致；多表关联查询时，
保证被关联的字段需要有索引。
说明：即使双表 join 也要注意表索引、SQL 性能。

系统的数据拆分成很多个表，如果需要重新关联起来需要这些表根据关联字段JOIN，但是如果数据量很大，涉及的表数目多，机器的计算能力跟不上则会造成查询性能低下等等问题。甚至有些数据库系统嫌弃事务管理导致性能慢，抛弃事务带来ACID的特性，嫌弃关系表JOIN执行慢，会做适当的数据冗余设计，即所谓的“反范式化”。之前我们谈到的文本型数据库(非关系型)MongDB就是“适当的反范式化的”代表。


### Join的算法比较

JOIN操作的主要目的就是根据两个表的关联关系，重新组合成一个新的表。但是这个新的表是一个"临时表", 即并没有模型或文件持久化这个表，大多数情况临时表都是暂时存在于内存中，且生命周期很短。这就涉及到性能的问题，怎么又快(时间性能的角度)，又省空间(存储的角度)的完成JOIN操作了。当下主要有三种流行的JOIN算法实现。

* 经典的Nest Loop

对于两个关系(表) R 和 S，进行 join 操作如下:

<pre>  
For each tuple r in R do
    For each tuple s in S do
        If r and s satisfy the join condition
           Then output the tuple &lt;r,s&gt;
</pre>
此外 Block nested loop 也是Simple Nest Loop的一种变形，前者的改进在于将驱动表的数据进行分组，每组内的tuple共用一次扫面从表的结果，避免每个驱动表的tuple都要对从表全表扫面。

### Lab2 实现小结

主要是实现一些关系代数操作符类的实现。
目前SimpleDB并没有实现类似于一致性检查的功能，例如主键约束(Primary Key Constraints)和外键约束(Foreign Key Constraints)，因此是可能插入内容的记录，并不保证主键或外键约束。


### 遗留的问题
1. Buffer Pool Page eviction的策略在不同lab的表现不一致，由于前面不考虑并发事务，lab2相关单元测试EvictionTest会把即使是dirty的页也回写到磁盘，但是lab5的时候，dirty页是不能回写磁盘的。导致前后单元测试试图表达的想法有冲突。


<!-- [0]: https://zh.wikipedia.org/zh-hans/关系代数_(数据库)

附: -->

<!-- 1. [MIT 6.830 lecture 3](http://db.csail.mit.edu/6.830/lectures/lec3-notes.pdf) -->
<!-- 2. [CMU 15-445 FD](http://15445.courses.cs.cmu.edu/fall2017/slides/04-functionaldependencies.pdf) -->
<!-- 3. [数据库范式 知乎回答](https://www.zhihu.com/question/24696366) -->

<!-- 2. [第二范式](https://zh.wikipedia.org/wiki/%E7%AC%AC%E4%BA%8C%E6%AD%A3%E8%A6%8F%E5%8C%96) -->

