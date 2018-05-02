---
layout: post
title: "Good Practices with MySQL"
date: 2017-01-12 21:48:18 +0800
comments: true
tags: MySQL
categories: [Database, MySQL]
---

In this blog, I will try my best to summary helpful experience with MySQL. Without other declaration, the deault engine which I used is innodb.

### Explain your query statement is a kind of virtue

If you are not confident with what have done in your query statements, please add `explain` key word before your query statement and let the interepter tell you how MySQL will excute the statement.

That's always right to explain and assert what you let the search engine done with your statements. 

If you don't know the meaning of the returned values of `explain` from MySQL, I will recommend you to read all about `Appendix D. Using Explain` in `<<High Performance MySQL>>`. You can also google some helpful documents about that.

That's the first step of all to optimize your MySQL statements.

More advanced, you can profile you statement like this: 

```
mysql> set profiling=1;
mysql> select filed from table;
mysql> ... more mysql statements and operation on MySQL ...
mysql> show profiles;
mysql> SHOW PROFILE FOR QUERY 1;
```

You can also profile you SQL statements with the help of the IDE. In MySQL-Workbench, the duration time is how long the statment excuted in the MySQL-Server and the fetch time is how long the results are transformed in the network.

[SHOW PROFILE Syntax](https://dev.mysql.com/doc/refman/5.7/en/show-profile.html)

<!-- more -->

### Do CRUD operations in batch 

Don't insert items from service program into databse one by one. Do it in batch.

* Proposal soluton 1

``` java
int insert(User user);

```

``` XML
    <!-- Bad practice -->
    <insert id="insert" parameterType="sims.model.User" >
    insert into users (email, username, password, userType)
    values (#{email,jdbcType=VARCHAR}, #{username,jdbcType=VARCHAR},
     #{password,jdbcType=VARCHAR}, #{userType, jdbcType=INTEGER})
    </insert>

```
    
* Proposal soluton 2

``` java
int insertUsers(@Param("users") List<User> users);

```

``` XML
<!-- Good practice
    Proposal 1:
    Use INSERT IGNORE rather than INSERT. If a record doesn't duplicate
    an existing record, MySQL inserts it as usual. If the record is a 
    duplicate, the IGNORE keyword tells MySQL to discard it silently 
    without generating an error.

    Proposal 2:
    Use REPLACE rather than INSERT. If the record is new, it's inserted
    just as with INSERT. If it's a duplicate, the new record replaces the
    old one
-->
<insert id="insertUsers">
    insert ignore into users (email, username, password, userType)
    values
    <foreach collection="users" item="element" index="index" open="(" separator="),
         ("  close=")">
        #{element.email,jdbcType=VARCHAR},
        #{element.username,jdbcType=VARCHAR},
        #{element.password,jdbcType=VARCHAR},
        #{element.userType, jdbcType=INTEGER}
    </foreach>
</insert>

```
Please pay attention to those guy who write database access statements in a for-loop. They are trouble maker :)

Try your best to avoid to do DAO operation in for-loop, otherwise you do it intention and try to avoid OMM for reading all rows from a table.


#### Performance Comparision

Operation :  

> insert 100 items into the table. 

Console output:  
> Solution One cost time: 9667 ms
Solution Two cost time: 495  ms

With the help of MyBatis, we can insert multi items in a list into database, but not insert them one by one. ( Hibernate could do the same thing)


### Use index as you can as possible
Make sure that __both fileds__ of two tables which you reference to join __have index__ on that fields. Otherwise, you may get into trouble about the performance of query process in MySQL.

### How to rate an index ?
Rate an index with respect to a given query by `Three Star` system. This technique is covered in `<<Relational Database Index Design and the Optimizers>>`. 

* First Star
Rows referenced by your query are grouped together in the index.

* Second Star
Rows referenced by your query are ordered in the index the way you want them.

* Third star:
The index contains all columns referenced by your query(covering index).

### Tuning MySQL: my.cnf

Sometimes, you find that the query statments are very slow and the expected return data are not more than 100k rows.

You may try to tune your MySQL server and modify the `my.cnf` file which is loaded by MySQL for initialization. Familiar with this confiuration file will help you to avoid this common pitfall.

#### Basic settings
Here are 3 MySQL performance tuning settings that you should always look at. If you do not, you are very likely to run into problems very quickly.

`innodb_buffer_pool_size`: this is the #1 setting to look at for any installation using InnoDB. The buffer pool is where data and indexes are cached: having it as large as possible will ensure you use memory and not disks for most read operations. Typical values are 5-6GB (8GB RAM), 20-25GB (32GB RAM), 100-120GB (128GB RAM).

`innodb_log_file_size`: this is the size of the redo logs. The redo logs are used to make sure writes are fast and durable and also during crash recovery. Up to MySQL 5.1, it was hard to adjust, as you wanted both large redo logs for good performance and small redo logs for fast crash recovery. Fortunately crash recovery performance has improved a lot since MySQL 5.5 so you can now have good write performance and fast crash recovery. Until MySQL 5.5 the total redo log size was limited to 4GB (the default is to have 2 log files). This has been lifted in MySQL 5.6.

Starting with innodb_log_file_size = 512M (giving 1GB of redo logs) should give you plenty of room for writes. If you know your application is write-intensive and you are using MySQL 5.6, you can start with innodb_log_file_size = 4G.

`max_connections`: if you are often facing the ‘Too many connections’ error, max_connections is too low. It is very frequent that because the application does not close connections to the database correctly, you need much more than the default 151 connections. The main drawback of high values for max_connections (like 1000 or more) is that the server will become unresponsive if for any reason it has to run 1000 or more active transactions. Using a connection pool at the application level or a thread pool at the MySQL level can help here.

#### How do you calculate mysql max_connections variable?

The basic formulas are:

    Available RAM = Global Buffers + (Thread Buffers x max_connections)
    max_connections = (Available RAM - Global Buffers) / Thread Buffers

To get the list of buffers and their values:

    SHOW VARIABLES LIKE '%buffer%';

Here's a list of the buffers and whether they're Global or Thread:

`Global Buffers`: key_buffer_size, innodb_buffer_pool_size, innodb_log_buffer_size, innodb_additional_mem_pool_size, net_buffer_size, query_cache_size

`Thread Buffers`: sort_buffer_size, myisam_sort_buffer_size, read_buffer_size, join_buffer_size, read_rnd_buffer_size, thread_stack


### Use NOT NULL If You Can

Unless you have a very specific reason to use a NULL value, you should always set your columns as NOT NULL.

First of all, ask yourself if there is any difference between having an empty string value vs. a NULL value (for INT fields: 0 vs. NULL). If there is no reason to have both, you do not need a NULL field. (Did you know that Oracle considers NULL and empty string as being the same?)


NULL columns require additional space and they can add complexity to your comparison statements. Just avoid them when you can. However, I understand some people might have very specific reasons to have NULL values, which is not always a bad thing.

From MySQL docs:
> "NULL columns require additional space in the row to record whether their values are NULL. For MyISAM tables, each NULL column takes one bit extra, rounded up to the nearest byte."

If you have do query operation on the table which are created by someone(maybe it's yourself). It hasn't a primary key and never set NOT NULL constraint with filed.

I will recommend you to use __having id is not NULL__ clause to filter the NULL rows which are selected by your query statments.

----
Recommended Resources:
* [Official Documents Of MySQL](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)
* `<<High Performance MySQL>>`
* `<<MySQL技术内幕: SQL编程>>`
* [B+Tree index structures in InnoDB](https://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/)
* `<<阿里巴巴Java编程规范>> SQL规约相关章节`