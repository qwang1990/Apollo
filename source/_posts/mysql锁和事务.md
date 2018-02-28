---
title: mysql锁和事务
date: 2018-02-27 22:30:13
categories: 
- 后端 
tags: 
- 数据库 
---

# MySQL InnoDB 锁
版本 5.7

## 共享锁和排它锁

InnoDB实现了两种标准的行锁，共享锁(shared lock)和排它锁(exclusive lock)。
- 共享锁允许持有它的事务读取改行
- 排它锁允许持有它的事务更新或删除改行

如果事务T1持有行r的共享锁，者别的事务T2对行r的请求会按如下处理:
- 如果是s锁的请求，者立刻得到授权，并且此时T1和T2都会持有行r的s锁
- 若果是x锁的请求者不能被直接授权

如果事务T1持有的是行r的排他锁，其他任何事务的任务请求都不会得到授权，其他事务要等待T1释放行r上的锁。

## 意向锁(intention locks)
InnoDB支持多种粒度的锁，它可以让行锁和表锁共存。比如说， LOCK TABLES ... WRITE 会对指定表上x锁。InnoDB使用意向锁来实现多个粒度的锁。意向锁是表级别的锁。
它代表着事务可能会在后面对表中的行申请s或x锁。我们有两种意向锁:

- 意向共享锁(IS) 表明事务有对表中的某一行设置共享锁的意向。
- 意向排他锁(IX) 表明事务有队表中的某一行设置排它锁的意向。

比如 SELECT ... LOCK IN SHARE MODE 设置了一个IS锁, SELECT ... FOR UPDATE 设置了一个IX锁。

意向锁的规则如下:

- 一个事务想要在表中的某一行获得s锁的前提是，它首先获得IS锁或比IS锁更强的锁。
- 一个事务想要在表中的某一行获得x锁的前提是，它首先获得IX锁。

![image](https://user-images.githubusercontent.com/13915081/36726906-48ea6232-1bf6-11e8-96fe-8fc465edf17c.png)

当事务申请的锁和当前锁的关系为compatible时，它可以获得该锁，但如果是conflicts，该事物必须等待当前所释放。如果事务所需的锁和当前锁conflicts，并且因为死锁的原因导致无法获得，就会出错。

除全表请求(LOCK TABLES ... WRITE)外，意向锁不会阻塞任何事。它的主要目的是表明有事务将要锁某行。

## 记录锁(Record Locks)
记录锁会锁定一个索引记录。比如，SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE 会阻止其他任何事务插入，更新或删除t.c1=10的行。
记录锁一定会锁住索引记录，即使这个表没有定义索引。这种情况下，InnoDB会创建一个隐含的clustered索引，然后为记录所使用这个索引。

## 间隙锁(Gap Locks)
间隙锁是一个锁住索引之间，或第一个索引之前，或最后一个索引之后的记录。比如 SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE会阻止其他事务插入c1值为15的记录，无论这个值是否存在，因为在间隙内的所有值都被锁了。

间隙锁可以跨越一个索引值，多个索引值甚至是0个索引值。

间隙锁是性能和并发性的一个权衡，它用于一些隔离级别上。

当使用唯一索引来检索唯一行的时候不需要间隙锁。(它不包括检索条件是一个多值唯一索引中的一部分值的情况)比如，如果id是唯一索引，这下面这个语句只会在id为100的记录上加index锁，而不会在它之前的间隙里加间隙锁。

```sql 
	SELECT * FROM child WHERE id = 100;
```

如果id没有索引或是一个非唯一的索引，上面的语句会锁住之前的间隙。

值得注意的是不同的事务可以持有冲突的间隙锁。比如，事务A可以持有一个间隙共享锁(gap S-lock),事务B可以在同一个间隙中持有间隙排它锁(gap X-lock)。？？

在InnoDB中间隙锁是“purely inhibitive”的，它意味着间隙锁只阻止事务插入到该间隙。它不阻止其他事务在同一个gap上获取锁。因此，间隙共享锁和间隙排它锁作用相同。

间隙锁可以显示的禁止。你可以吧事务隔离级别设为READ COMMITTED或者启用innodb_locks_unsafe_for_binlog变量(这个已经被废弃了)。在这种情况下间隙锁只会被用于外键约束检测和重复key值检测了。

## Next-Key Locks
Next-Key锁是索引记录上的record锁和索引近路之前的gap锁的组合。

InnoDB是以下面的方式来执行级别的锁，当搜索或扫描一个表的索引时，它会在遇到的索引记录上加s或x锁。所以行锁实际上就是index-record锁。索引记录上的next-key锁也会影响所以记录之前的区间。因此next-key锁是index-record锁加上索引之前的间隙。如果一个会话在索引记录R上有s或x锁，其他会话不能在索引顺序在R之前的区间中插入新的值。

假设一个索引有值10，11，13，20。可能的next-key如下：

```sql
	(negative infinity, 10]
	(10, 11]
	(11, 13]
	(13, 20]
	(20, positive infinity)
```

最后一种情况下，next-key仅仅锁了一个间隙。默认情况下，InnoDB工作在 REPEATABLE READ 事务隔离级别下，此时它使用next-key来检索和遍历索引。它可以避免phantom行（后面会讲到）。


## 插入意向锁(REPEATABLE READ)
插入意向锁是insert上的gap锁，优先级高于行插入。这个锁标明希望用如下方式插入记录的意向:当多个事务插入同一个索引间隙，如果它们插入的不是同一个位置，它们就不需要等待其他事务。假设在4和7之间个index record， 两个事务尝试插入5和6，在获取x锁之前，它们都会获得4和7间隙中的插入意向锁。

下面一个例子说明事务获得插入意向锁的优先级高于被插入记录上的排他锁。假设我们有两个客户端A和B。
A创建了一个表有两个索引记录 90和102，然后开启了一个事务，获取Id大于100的x锁。这个x锁包含一个小于102的gap锁。

```sql
	mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
	mysql> INSERT INTO child (id) values (90),(102);

	mysql> START TRANSACTION;
	mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
	+-----+
	| id  |
	+-----+
	| 102 |
	+-----+
```

这时B开启了一个事务，插入一个值到该gap中。这个事务先获得插入意向锁，然后等待获取x锁。
```sql
	mysql> START TRANSACTION;
	mysql> INSERT INTO child (id) VALUES (101);
```

插入语句的事务数据如下:

```sql
	RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
	trx id 8731 lock_mode X locks gap before rec insert intention waiting
	Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
	 0: len 4; hex 80000066; asc    f;;
	 1: len 6; hex 000000002215; asc     " ;;
	 2: len 7; hex 9000000172011c; asc     r  ;;...
```

## 自增锁(AUTO-INC Locks)
自增锁是一个特殊的表级别的锁，它出现在当一个事务想要插入一个有AUTO_INCREMENT列的表的时候。在这种情况下，如果一个事务想要在该表中插入值，其他想插入数据的表必须等待它完成。

 innodb_autoinc_lock_mode可以控制自增锁的算法，它允许你权衡自增值的可以测性和插入操作的并发性。。