---
title: "MySQL之MVCC机制"
subtitle: ""
layout: post
author: "Aug"
header-style: text
tags:
  - MySQL
---


# MVCC机制

> **Multiversion concurrency control (MCC or MVCC)**, is a concurrency control method commonly used by database management systems to provide concurrent access to the database and in programming languages to implement transactional memory. 
>
> MVCC机制的实现依赖于undo log和read view

## 当前读和快照读

- 快照读：读取的是记录的可见版本（有可能是历史版本），不用加锁。简单的select属于快照读
- 当前读：读取的是记录的最新版本，并且当前读返回的记录都会加上锁，保证其他事务不会并发修改这条记录。特殊的读操作，insert/update/delete操作属于当前读

## 一致性非锁定读

​		一致性非锁定读(consistent nonlocking read)是指InnoDB存储引擎通过多版本控制(MVCC)读取当前数 据库中行数据的方式。如果读取的行正在执行DELETE或UPDATE操作，这时读取操作不会因此去等待行上锁的释放。相反地， InnoDB会去读取行的一个最新可见快照。 快照存储于undo log，而undo log存储于共享表空间中的回滚段中。

## undo log

![](https://tva1.sinaimg.cn/large/006tNbRwly1g9jj4pkzloj30vl0u0gyq.jpg)

​	事务进行过程中，每次sql语句执行，都会记录undo log和redo log，然后更新数据形成脏页，然后 redo log按照时间或者空间等条件进行落盘，undo log和脏页按照checkpoint进行落盘，落盘后相应的 redo log就可以删除了。此时，事务还未COMMIT，如果发生崩溃，则首先检查checkpoint记录，使用 相应的redo log进行数据和undo log的恢复，然后查看undo log的状态发现事务尚未提交，然后就使用 undo log进行事务回滚。事务执行COMMIT操作时，会将本事务相关的所有redo log都进行落盘，只有 所有redo log落盘成功，才算COMMIT成功。然后内存中的数据脏页继续按照checkpoint进行落盘。如 果此时发生了崩溃，则只使用redo log恢复数据。 

![](https://tva1.sinaimg.cn/large/006tNbRwly1g9jj5p9f2ij30xy0bstbc.jpg)



## ReadView

> 事务开启时当前所有活跃事务的一个集合
>
> RR隔离级别：开启事务的时候，只生成一次ReadView，一直维持到事务提交，中间每次查询都不会重建ReadView，从而实现了可重复读
>
> RC隔离级别：在事务中每执行一次查询，都会重新生成ReadView，所以不可重复读

**数据结构**

- creator_trx_id：当前事务id
- up_limit_id：最先开始的活跃事务id
- low_limit_id：离当前事务最近的活跃事务id
- trx_ids：所有活跃事务id列表

**数据版本访问规则**：

- trx_id=creator_trx_id，即当前事务在访问它自己修改过的记录，可访问。
- trx_id<up_limit_id，即该版本的事务已提交，可访问。
- trx_id>low_limit_id，即该版本的事务在当前事务创建之后创建，不可访问。
- up_limit_id < trx_id < low_limit_id，即该版本在中间，此时需要看一下trx_ids列表，如果在里面，则为活跃事务，不可访问；如果不在，则说明该事务已提交，可访问