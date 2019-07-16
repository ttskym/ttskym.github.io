---
layout:     post
title:      "数据库事务和锁"
subtitle:   "DataBase Transactional and Lock"
date:       2019-07-01 22:40:00
author:     "jk"
header-img: "img/post-bg-css.jpg"
catalog: true
tags:
    - DataBase
    - Mysql
    - Transactional
    - Lock
    - Spring
---
## 数据库事务

简述：数据库事务是一组对数据库的逻辑操作，不同的事务之间彼此独立隔离，并发执行，最终结果与多个事务串行执行一致。

数据库事务主要有两个使命：

- 系统崩溃时候提供容错，保证数据库内容满足定义的各种约束及正确完整。
- 提供多个事务并发操作的能力

#### ACID

- A-原子性：由多个操作组成的一个数据库事务满足执行效果的原子性，either occur, or nothing occurs.
- C-一致性：事务提交时，保证满足定义的各种约束，以及已提交事务对未来事务的可见性。
- I-隔离性：决定了事务影响的内容对于用户、系统或其他事务的可见程度，解决事务并发执行时的冲突和正确性
- D-持久性：保证已提交事务永久生效，即使系统崩溃也可以恢复。

数据库事务通过四个特性保证了上述的两个使命，其中CD主要保证保证了数据库在异常情况下的容错能力，而AI主要解决数据库事务并发的问题。

## 锁

### 两段锁 (Two-phase Locking)

两段锁是数据库中最常见的并发控制方法。两段锁要求事务访问数据库对象时，必须要先获得该对象的锁，锁类型由操作类型决定。

2PL包含两个阶段：

- 上锁：获取锁，不能释放锁
- 解锁：释放锁，不能获取锁

简单的说，就是明确区分加锁阶段和解锁阶段，加锁阶段只能加锁，解锁阶段只能解锁。每个阶段锁的数量可以变化。

![two_phrase_lock](https://img.charflow.com/2019-07-01-DataBase_Transactional_and_Lock/two_phrase_lock.jpeg)

#### when lock, when release

对记录进行更新操作或者用select for update(X)、select lock in share mode(S)时，会对记录进行加锁。

事务提交(commit) 或者 rollback时，解锁

### 乐观锁

事务中一般的select不会加锁，而是会用MVCC实现的一致性非锁定读，不同隔离级别下的一致性非锁定读对于快照数据的定义是不同的，如RC下读取最新的快照数据，而RR下读取事务开始时的快照数据。此外，在Serializable级别下不会有快照读，执行一般select时也会对记录加锁(S)。

乐观锁更新数据时会引入一个数据版本的记录机制，操作如下：

```sql
update TABLE
set value=2,version=version+1
where id=#{id} and version=#{version}
```

### 悲观锁

#### S、X、IS、IX

|      | X      | S      |
| ---- | ------ | ------ |
| X    | 不兼容 | 不兼容 |
| S    | 不兼容 | 兼容   |

|      | IS     | IX     | S      | X      |
| ---- | ------ | ------ | ------ | ------ |
| IS   | 兼容   | 兼容   | 兼容   | 不兼容 |
| IX   | 兼容   | 兼容   | 不兼容 | 不兼容 |
| S    | 兼容   | 不兼容 | 兼容   | 不兼容 |
| X    | 不兼容 | 不兼容 | 不兼容 | 不兼容 |

上述表中可分为四象限来看：

- 第一和第三象限的内容一样，都表示意向锁和SX的兼容性
  - 只有S和IS是兼容的
  - IX和X与其他任何一种锁都不兼容
- 第二象限的信息则表示意向锁都是兼容的
- 第四象限的内容则是共享锁和排他锁的兼容性

#### 一致性锁定读

mysql中的读操作可分为一致性非锁定读和一致性锁定读，前者是通过MVCC(乐观锁)实现的，后者则是通过锁(悲观锁)实现的。前者也叫快照读，后者叫做当前读或者锁读。

- SELECT ... FOR UPDATE     排他锁
- SELECT ... LOCK IN SHARE MODE 共享锁

事务提交，锁释放

### 锁算法

Innodb三种行锁算法：

- Record Lock
- Gap Lock
- Next-key Lock

具体场景加锁参考

[InnoDB locking](https://github.com/octachrome/innodb-locks)

[MySQL InnoDB存储引擎(三)：锁及事务模型](https://t.hao0.me/mysql/2016/09/10/mysql-innodb-03-lock-and-transaction-model.html)

### 锁问题与隔离级别

数据库存在两类数据丢失更新问题，一类包括脏读、不可重复读、幻读等，另一个类则是二类丢失更新。

**脏读问题**

一个事务可以读到另一个事务未提交的数据，也叫脏数据。在隔离级别Read Uncommitted 下存在。

**不可重复读问题**

事务未提交时读取的数据状态与事务提交后读取的数据状态不一致，因为有另一个事务在事务未提交期间修改了数据的状态。在Read Committed下存在。

**幻读**

幻读与不可重复读类似，只不过幻读更倾向于新增或删除数据，导致另一个事务提交前后检索数据库范围内的结果不同，mysql其实不区分不可重复读和幻读，统一为幻读问题，通过Gap Lock和Next-Key Lock解决，在Read Repeatable级别mysql解决了两个问题，但是在数据库标准中，幻读问题是在隔离级别Serilizable级别中将所有事务串行化解决的。

#### 丢失更新

丢失更新更多的是一种应用逻辑的问题。即一个事务读取数据的状态并缓存，而另一个事务修改了该数据状态后，第一个事务并未更新缓存，仍然用之前的缓存数据进行计算，并将结果更新到数据库，这就会覆盖前一个事务的结果，造成逻辑错误。

解决办法是将这种情况下的事务进行串行化，对事务中的操作进行加锁。

```sql
BEGIN;
SELECT something;
think about it for a while
UPDATE that something;
COMMIT;
```

上面这样的写法就会造成丢失更新。select 应该替换为如下写法：

```sql
SELECT something  FOR UPDATE;
```

### 死锁

```sql
BEGIN;    -- in one connection
UPDATE thing_1;
UPDATE thing_2;
COMMIT;

BEGIN;    -- in another connection, at the "exact same time"
UPDATE thing_2;
UPDATE thing_1;
COMMIT;
```

## Spring 事务管理

[可能是最漂亮的Spring事务管理详解](https://juejin.im/post/5b00c52ef265da0b95276091)

### 事务传播行为

[Spring事务传播行为详解](https://juejin.im/entry/5a8fe57e5188255de201062b)

[Ref]

[MySQL InnoDB存储引擎(三)：锁及事务模型](https://t.hao0.me/mysql/2016/09/10/mysql-innodb-03-lock-and-transaction-model.html)

[MySQL: will a transaction lock the row?](https://dba.stackexchange.com/questions/166242/mysql-will-a-transaction-lock-the-row?newreg=2c20159723ab4584b9e6f3b1f0d15498)

[可能是最漂亮的Spring事务管理详解](https://juejin.im/post/5b00c52ef265da0b95276091)