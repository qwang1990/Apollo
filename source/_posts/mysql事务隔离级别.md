---
title: mysql事务隔离级别
date: 2018-02-28 20:14:03
categories: 
- 后端 
tags: 
- 数据库 
---

# InnoDB事务模型
## 事务隔离级别
事务隔离是数据库的基础。隔离是ACID中的I，隔离级别是用来调整性能，可靠性，一致性和当多个事务同时改变或查询一个结果时的复现性的。

InnoDB提供了四中隔离级别， READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE。默认的隔离界别是REPEATABLE READ。

使用 SET TRANSACTION 用户可以改变一个回话的隔离级别或全部后续链接的隔离级别。也可以在命令行或option文件中使用 --transaction-isolation 来设置默认的隔离级别。

InnoDB对每种隔离级别使用不同的锁策略。 REPEATABLE READ 可以给你带来高一致性，你也可以使用 READ COMMITTED，甚至 READ UNCOMMITTED 来放松对一致性的要求。当然你也可以用SERIALIZABLE增强约束，它一般用于特殊场景下，比如[XA](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_xa)或解决并发死锁问题。

下面描述了MySQL支持的四种隔离界别，顺序是按照使用频率来的:

- REPEATABLE READ
这个是InnoDB的默认隔离级别。读一致性，在一个事务内每次读的都是第一次读的快照。这意味着如果你在同一个事务里发起多次的普通select操作它们是一致的。

对于locking read(SELECT with FOR UPDATE 或 LOCK IN SHARE MODE)，更新，删除，锁依赖于该语句是否是使用唯一检索条件或范围检索条件的唯一索引。
	
	- 如果是唯一索引并且是唯一的检索条件，InnoDB仅仅使用index record锁，而不用gap锁。
	- 如果是其他检索条件，InnoDB使用index锁，next-key或gap-key来阻塞其他回话对该区间的插入行为。

- READ COMMITTED
每次读都是设置并且读取自己的快照(即便是在同一个事务内)。

对于locking read (SELECT with FOR UPDATE or LOCK IN SHARE MODE)，update，delete，InnoDB仅仅有index锁，不会有gaps锁，所以其他事务可以随意在上锁的索引记录附近插入记录。gap锁仅仅用于外键约束和duplicate-key检查。

因为gap锁不在了，所以可能会有幻读问题(phantom problems)，因为其他事务可以在区间插入新的记录。

使用 READ COMMITTED 有如下影响：
	
	- 对于update或delete，InnoDB仅仅锁住那些它需要update或delete的行。那些没有匹配上的行上的record locks会在where条件计算完后释放。这会大大降低死锁可能，但依然可能发生。
	- 对于update，如果该行已经锁定了，InnoDB执行“semi-consistent”读，它返回最近的一次提交版本给mysql，让mysql确定改行是否符合update的where条件。如果匹配(一定会被更新)，mysql就会再次读取该行，这次InnoDB要么锁住它，要么等待锁住它。


看一下下面一个例子:

```mysql
	CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
	INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
	COMMIT;
```
在这个例子中，table没有索引，所以检索和扫描用的是隐藏的clustered索引。

假设有一个回话执行如下update操作:

```mysql
	# Session A
	START TRANSACTION;
	UPDATE t SET b = 5 WHERE b = 3;
```

假设又有一个回话做下面的update操作:

```mysql
	# Session B
	UPDATE t SET b = 4 WHERE b = 2;
```

当使用默认的REPEATABLE READ隔离级别的时候，第一个update会给它读的每一行上x-lock，并且不会释放它们:

```mysql
	x-lock(1,2); retain x-lock
	x-lock(2,3); update(2,3) to (2,5); retain x-lock
	x-lock(3,2); retain x-lock
	x-lock(4,3); update(4,3) to (4,5); retain x-lock
	x-lock(5,2); retain x-lock
```

此时第二个update会阻塞，知道第一个结束或回滚:

```mysql
	x-lock(1,2); block and wait for first UPDATE to commit or roll back
```

如果是READ COMMITTED隔离级别，第一个update会给它读过的每一行上x-lock，但是会释放那些不需要修改的。

```mysql
	x-lock(1,2); unlock(1,2)
	x-lock(2,3); update(2,3) to (2,5); retain x-lock
	x-lock(3,2); unlock(3,2)
	x-lock(4,3); update(4,3) to (4,5); retain x-lock
	x-lock(5,2); unlock(5,2)
```

此时第二个update来了，InnoDB会执行“semi-consistent”读，返回每一行的最后一个提交的版本，然后确定改行是否需要修改:

```mysql
	x-lock(1,2); update(1,2) to (1,4); retain x-lock
	x-lock(2,3); unlock(2,3)
	x-lock(3,2); update(3,2) to (3,4); retain x-lock
	x-lock(4,3); unlock(4,3)
	x-lock(5,2); update(5,2) to (5,4); retain x-lock
```

使用READ COMMITTED隔离级别和启用 innodb_locks_unsafe_for_binlog 配置的效果一样，除了:

	-  innodb_locks_unsafe_for_binlog是全局设置，而隔离级别可以使全局，全部回话甚至是一个回话。
	-  innodb_locks_unsafe_for_binlog只能服务启动的时候设置，但是隔离界别可以随时更改。

READ UNCOMMITTED
随意读，可能会有脏读

SERIALIZABLE
这个级别和REPEATABLE READ很想，只是InnoDB给每个select隐式的加上了 SELECT ... LOCK IN SHARE MODE。 其实就是全部事情都线性执行了。

关于幻读的问题，网上有很多说法，有说RR级别能防止幻读的，有说不行的。但是主要取决于你对幻读的理解，官网上说RR模式下能避免幻读。先看一下官网对幻读的定义:

幻读是指在一个事务内同样的查询在不同的时间得到不同的结果。

假设在id列上有索引，你想读取并且锁住全部大于100的记录用于以后更新:

```mysql 
	SELECT * FROM child WHERE id > 100 FOR UPDATE;
```

其实就是在执行上面语句的时候，利用next-key的机制不仅仅锁住了index record，而且还有gap lock。他会阻止其他事务想该表中插入任务id大于100的记录。

有一种别的说法
在如下情况下会出现“幻读”：

```
	transaction1:
	START TRANSACTION;
	SELECT * FROM child WHERE id > 100 ;

	transaction2:
	START TRANSACTION;
	INSERT INTO child(id,a) VALUES (103,2);
	COMMIT;

```

此时再次在transaction1中执行	SELECT * FROM child WHERE id > 100 ; 不会出现刚刚插入的行。
但是如果在transaction1中执行  update child set a = 2; 然后在执行select就会发现出现了刚刚插入的行。

