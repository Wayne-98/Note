---
title: 高性能MySQL第六章：查询性能优化
date: {date}
tags: MySQL
categories: 高性能MySQL
---


## 为什么查询速度会慢？

* 查询的生命周期

  查询的生命周期，从客户端，到服务器，然后在服务器上进行解析，生成执行计划，执行，并返回结果给客户端。执行是查询生命周期最重要的阶段，这其中包括了大量为了检索数据到存储引擎的调用以及调用后的数据处理，包括排序、分组等等。

## 慢查询基础：优化数据访问

* 分析步骤

1. 确认应用程序是否在检索大量超过需要的数据。这通常意味着访问了太多的行，但有时候也是可能访问了太多的列

2. 确认MySQL服务器层是否在分析大量超过需要的数据行

### 是否向数据库请求了不需要的数据

1. 查询不需要的记录（加上limit）
2. 多表关联时返回全部列（查询涉及三张表的时候，使用select *）
3. 总是取出全部列（无法使用索引覆盖扫描，慎用select*）
4. 重复查询相同的数据（缓存）

### MySQL是否在扫描额外的记录

* **衡量查询开销的三个指标：**

1. 响应时间：
   * 服务时间是指数据库处理这个查询花了多少时间
   * 排队时间是指服务器因为等待某些资源而没有真正执行查询的时间。可能是等I/O操作完成，等行锁

2. 扫描的行数：
3. 返回的行数：

这三个指标都会存放在MySQL的慢日志中，所以检查慢日志记录是找出扫描行数过多的查询的好办法。

* **访问类型：**

全表扫描，索引扫描，范围扫描，唯一索引查询，常数引用

* **MySQL 应用 where ：**

1. 在索引中使用 where 条件来过滤不匹配的记录。在存储引擎层完成的。
2. 使用索引扫描(在 Extra 列中出现了 Using Index )来返回记录，直接从索引中过滤不需要的记录并返回命中结果。这是在 MySQL 服务器层完成的，但无须回表查询记录
3. 在数据表中返回数据，然后过滤不满足条件的记录(在 Extra 列中出现 Using Where)。这是在 MySQL 服务器层完成，MySQL 需要先从数据表读出记录然后过滤

## 重构查询的方式

* **一个复杂的查询还是多个简单的查询**

* **切分查询：**大查询切分成小查询，每个查询功能完全一样，每次只完成一小部分

* **分解关联查询：**

1. 对每一个表进行一次单表查询，然后将结果在应用程序中进行关联

2. 让缓存的效率更高  
3. 将查询分解后，执行单个查询可以减少锁的竞争
4. 在应用层做关联，可以更容易对数据库进行拆分，更容易做到高性能和可扩展

## 查询执行的基础

1. 客服端发送一条查询给服务器

2. 服务器先检查查询缓存，如果命中缓存，则立即返回
3. 服务器端进行SQL解析，预处理，再由优化器生成对应的执行计划
4. MySQL根据优化器生成的执行计划，调用存储引擎的API来执行查询
5. 返回结果给客户端

### MySQL客户端/服务端通信协议：半双工

### 查询缓存

### 查询优化处理

1. 语法解析器和预处理：生成“解析树”并验证解析树是否合法，验证权限
2. 查询优化器：基于成本的优化器
3. 数据和索引统计信息：统计信息在存储引擎层，服务器层在执行查询优化的时候需要调用api
4. MySQL如何执行关联查询：嵌套循环关联操作
   * MySQL先在一个表中循环取出单条数据，然后再嵌套循环到下一个表中寻找匹配的行，依次下去，直到找到所有表中匹配的行为止。然后根据各个表匹配的行，返回查询中需要的列。
     * 如果是左外连接，取出单条数据遍历下一个表的时候，如果没有匹配，则输出当前行+NULL
     * 右连接改正等价的左外连接

5. 执行计划：生成一颗查询的指令树

6. 关联查询优化器：

7. 排序优化：

   * 如果需要排序的数据量小于“排序缓冲区”，MySQL使用内存进行“快排”。如果内存不够，MySQL先将数据分块，每个独立的块使用快排然后刷到磁盘上，最后将各个排好序的块合并。

   * 排序算法：

     * 两次传输排序：读取行指针和需要排序的字段，对其进行排序，然后根据排序结果读取所需要的数据行。优点是排序的时候存储尽可能少的数据，缺点是第二次数据传输的时候又大量的随机I/O

     * 单次传输排序：先读取查询所需要的所有的列，然后再根据给定列进行排序，最后直接返回结果

   * 关联查询的时候如果需要排序：
     * orderBy 子句中的所有列全部来自关联的第一个表，那么 MySQL 在关联处理第一个表时就进行文件排序。Extra：Using filesort
     * 除此之外，MySQL 都会将关联的结果存放在一个临时表中，在所有的关联都结束的时候，再进行文件排序。Extra：Using temporary；Using filesort
     * 有limit，也会对全量数据排序，然后返回较少的数据（5.6版本之后，对这个有优化，不再是全量数据排序）

### 查询执行引擎

* 解析和优化阶段，MySQL生成对应的执行计划(数据结构)
* 查询中的每个表有一个handler的实例表示，存储引擎的接口称为handler API

### 返回结果给客户端：TCP