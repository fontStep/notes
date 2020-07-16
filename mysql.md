# 事务

## 概念

事务指的是满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚

每个操作就相当于一个事务

## ACID

### 原子性（Atomicity）

#### 定义

事务被视为不可分割的最小单元，同个事务中的所有操作要不同时成功,要不同时失败

#### 实现原理

回滚可以用回滚日志（Undo Log）来实现，回滚日志记录着事务所执行的修改操作，在回滚时反向执行这些修改操作。

Undo Log属于逻辑日志,它记录的是sql执行相关的信息,当发生回滚时，innodb会根据Undo Log的内容做相反的操作，对于insert语句，会执行相反的delete语句，对于delete语句，会执行相反的insert语句，对于update语句会执行相反的update语句

### 一致性（Consistency）

#### 定义

一致性是指事务执行结束后，**数据库的完整性约束没有被破坏，事务执行的前后都是合法的数据状态。**数据库的完整性约束包括但不限于：实体完整性（如行的主键存在且唯一）、列完整性（如字段的类型、大小、长度要符合要求）、外键约束、用户自定义完整性（如转账前后，两个账户余额的和应该不变）。

#### 实现

可以说，一致性是事务追求的最终目标：原子性、持久性和隔离性，都是为了保证数据库状态的一致性。此外，除了数据库层面的保障，一致性的实现也需要应用层面进行保障。

实现一致性的措施包括：

- 保证原子性、持久性和隔离性，如果这些特性无法保证，事务的一致性也无法保证
- 数据库本身提供保障，例如不允许向整形列插入字符串值、字符串长度不能超过列的限制等
- 应用层面进行保障，例如如果转账操作只扣除转账者的余额，而没有增加接收者的余额，无论数据库实现的多么完美，也无法保证状态的一致

### 隔离性（Isolation）

#### 定义

**与原子性，持久性侧重于研究事务本身不同,隔离性研究的是不同事务之间的相互影响**事务内部的操作与其他事务是隔离的，并发执行的各个事务之间不能互相干扰。严格的隔离性，对应了事务隔离级别中的Serializable (可串行化)，但实际应用中出于性能方面的考虑很少会使用可串行化。

#### 锁机制

##### 行锁与表锁

关于锁的概念默认在innodb存储引擎下,按照锁的颗粒度可以 分为表锁和行锁，表锁在操作数据的时候会锁定整张表，并发性能较差，行锁只会锁定需要操作的数据，并发性相对较好，但是由于加锁本身需要消耗资源（获取锁，检查锁，释放锁等），因此在数据较多的情况下使用表锁可以节约大量的资源

##### 查看锁的信息

```mysql
select * from information_schema.innodb_locks; #锁的概况
show engine innodb status; #InnoDB整体状态，其中包括锁的情况
```



```mysql
CREATE TABLE `account` (
  `id` bigint(255) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `accout_name` varchar(20) DEFAULT '' COMMENT '账户名称',
  `balance` decimal(10,0) DEFAULT NULL COMMENT '余额',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4
```

使用两个不同的事务来模拟锁等待的问题

```mysql
#在事务A中执行：
start transaction;
update account SET balance = 1000 where id = 1;
#在事务B中执行：
start transaction;
update account SET balance = 2000 where id = 1;
```

当事务A和事务B同时对accout表中ID = 1的记录进行更新操作时,

事务A会对ID=1的记录进行加X锁操作，事务B只有等到事务A释放X锁之后才能进行接下来的事务操作，直到获取锁超时

![image-20200712182726340](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712182726340.png)

![image-20200712183511785](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712183511785.png)

##### 脏读、不可重复读和幻读

脏读 当前事务（A）中可以读取到其他事务（B）中未提交的事务，这种情况下只要事务B回滚事务，事务A相对来说读取的是错误的错误，称为脏数据

查看当前会话对应的事务隔离级别，将对应的事务隔离级别调整为未提交度



```mysql
# 事务B将ID为1的数据的balamce修改为200
 set session transaction isolation level read uncommitted;
 select @@tx_isolation;
 start transaction;
 update account SET balance = 200 where id = 1;
```

![image-20200712184603244](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712184603244.png)

```mysql
 # 事务A读取ID=1的记录会读取到事务B未提交的数据
 set session transaction isolation level read uncommitted;
 start transaction;
 select * from account where id = 1;
```



![image-20200712184851334](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712184851334.png)

| 时间 | 事务A                                     | 事务B                                          |
| ---- | ----------------------------------------- | ---------------------------------------------- |
| T1   | start transaction;                        | start transaction;                             |
| T2   |                                           | update account SET balance = 200 where id = 1; |
| T3   | select * from account where id = 1;[脏读] |                                                |
| T4   |                                           | commit;                                        |

不可重复读：在事务A中先后两次读取同一个数据，两次读取的结果不一样，这种现象称为不可重复读。脏读与不可重复读的区别在于：前者读到的是其他事务未提交的数据，后者读到的是其他事务已提交的数据

```mysql
set session transaction isolation level read committed;
```

| 时间 | 事务A                                           | 事务B                                         |
| ---- | ----------------------------------------------- | --------------------------------------------- |
| T1   | start transaction;                              | start transaction;                            |
| T2   | select * from account where id = 1;             | update account SET balance =300 where id = 1; |
| T3   |                                                 | commit;                                       |
| T4   | select * from account where id = 1;(不可重复读) |                                               |

幻读：在事务A中按照某个条件先后两次查询数据库，两次查询结果的条数不同，这种现象称为幻读。不可重复读与幻读的区别可以通俗的理解为：前者是数据变了，后者是数据的行数变了

| 时间 | 事务A                                                        | 事务B                                                        |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| T1   | start transaction;                                           | start transaction;                                           |
| T2   | select * from account where id <5 ;<br /><br /><br />1  zhangsan 300 |                                                              |
| T3   |                                                              | INSERT INTO `test`.`account`( `accout_name`, `balance`) VALUES ( 'lisi', 2000); |
| T4   |                                                              | commit;                                                      |
| T5   | select * from account where id <5</br><br/>1  zhangsan 300 </br> 2 lisi 2000 |                                                              |
|      |                                                              |                                                              |

### 持久性（Durability）

#### 定义

一旦事务提交，他对数据库做的修改是永久性

#### 实现原理

innoDB作为mysql的存储引擎，数据是存放在磁盘中的，但如果每次读写数据都需要磁盘IO,效率会很低，为此，InnoDB提供了缓存（Buffer Pool）Buffer Pool中包含了磁盘中部分数据页的映射，作为访问数据库的缓冲，当从数据中读取数据时，会首先从Buffer Pool中读取，如果Buffer Pool中没有，则从磁盘中读取后放入Buffer Pool，当像数据库中写入数据时，会首先写入Buffer Pool Buffer Pool 中修改的数据会定期刷新到磁盘中（这一过程称为刷脏）

Buffer Pool的使用大大提高了读写数据的效率，但是也带来了新的问题：如果Mysql宕机，而此时Buffer Pool中修改的数据还没有刷新到磁盘，就会导致数据的丢失，事务的持久性就无法保证

redo log被引用来解决这个问题：当数据被修改时，除了修改Buffer Pool中的数据，还会在redo log日志文件中记录此次操作,当事务提交时，会调用sync接口对redo log进行刷盘，如果mysql宕机，重启时可以读取redo log中的数据，对数据库进行恢复，redo log采用的是WAL（Write-ahead logging，预写式日志），所有修改先写入日志，再更新到Buffer Pool，保证了数据不会因为mysql宕机而丢失数据，从而满足了持久性要求

既然redo log也需要在事务提交时将日志写入磁盘，为什么它比直接将Buffer Pool中修改的数据写入磁盘(即刷脏)要快呢？主要有以下两方面的原因：

（1）刷脏是随机IO，因为每次修改的数据位置随机，但写redo log是追加操作，属于顺序IO。

（2）刷脏是以数据页（Page）为单位的，MySQL默认页大小是16KB，一个Page上一个小修改都要整页写入；而redo log中只包含真正需要写入的部分，无效IO大大减少。

#### redo log与binlog

我们知道，在MySQL中还存在binlog(二进制日志)也可以记录写操作并用于数据的恢复，但二者是有着根本的不同的：

（1）作用不同：redo log是用于crash recovery的，保证MySQL宕机也不会影响持久性；binlog是用于point-in-time recovery的，保证服务器可以基于时间点恢复数据，此外binlog还用于主从复制。

（2）层次不同：redo log是InnoDB存储引擎实现的，而binlog是MySQL的服务器层(可以参考文章前面对MySQL逻辑架构的介绍)实现的，同时支持InnoDB和其他存储引擎。

（3）内容不同：redo log是物理日志，内容基于磁盘的Page；binlog的内容是二进制的，根据binlog_format参数的不同，可能基于sql语句、基于数据本身或者二者的混合。

（4）写入时机不同：binlog在事务提交时写入；redo log的写入时机相对多元：

- 前面曾提到：当事务提交时会调用fsync对redo log进行刷盘；这是默认情况下的策略，修改innodb_flush_log_at_trx_commit参数可以改变该策略，但事务的持久性将无法保证。
- 除了事务提交时，还有其他刷盘时机：如master thread每秒刷盘一次redo log等，这样的好处是不一定要等到commit时刷盘，commit速度大大加快。

## mvcc

