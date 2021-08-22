## Spring教程：事物详解（一）初探事物



## 引言

很多coder在不理解事务的原理甚至连基本概念都不清楚的情况下，就去使用数据库事务，是极容易出错，写出一些自己不能掌控的代码。网上很多文章要不就是概念，或者一点源码，或者一点测试验证，都不足以全面了解事务，所以本文出现了，本系列Spring事务详解包含四部分：

第一章 讲概念，对事务的整体有一个了解。

第二章 从源码来看底层实现机制。

第三章 实例测试验证。

第四章 总结提高。



## 一、 背景

### 1.1 拜神

Spring事务领头人叫 `Juergen Hoeller`，先混个脸熟哈，他写了几乎全部的Spring事务代码。读源码先拜神，掌握他的源码的风格，读起来会通畅很多。最后一节咱们总结下这个大神的代码风格。

![](https://images2018.cnblogs.com/blog/584866/201809/584866-20180906151913868-1478211392.png)

### 1.2 事务的定义

**事务（Transaction**）是数据库区别于文件系统的重要特性之一。目前国际认可的数据库设计原则是**ACID特性**，用以保证数据库事务的正确执行。Mysql的innodb引擎中的事务就完全符合ACID特性。

Spring对于事务的支持，分层概览图如下：

<img src="img/%E6%88%AA%E5%B1%8F2021-08-04%20%E4%B8%8B%E5%8D%884.41.32.png" style="zoom:50%;" />

## 二、事务的ACID特性

- **原子性（Atomicity）：一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚**。主要涉及InnoDB事务。相关特性：事务的提交，回滚，信息表。
- **一致性（consistency）：数据库总是从一个一致性的状态转换到另一个一致性的状态。在事务开始前后，数据库的完整性约束没有被破坏**。例如违反了唯一性，必须撤销事务，返回初始状态。
- **隔离性（isolation）：每个读写事务的对象对其他事务的操作对象能相互分离，即：事务提交前对其他事务是不可见的，通常内部加锁实现**。主要涉及事务，尤其是事务隔离级别，相关特性：隔离级别、innodb锁的底层实现细节。
- **持久性（durability）：一旦事务提交，则其所做的修改会永久保存到数据库**。涉及到MySQL软件特性与特定硬件配置的相互影响，相关特性：4个配置项：双写缓冲开关、事务提交刷新log的级别、binlog同步频率、表文件；写缓存、操作系统对于 `fsync()` 的支持、备份策略等。

## 三、事务的属性定义

要保证事务的ACID特性，Spring给事务定义了6个属性，对应于声明式事务注解 `@Transational` （`org.springframework.transaction.annotation.Transactional`）中6个成员属性。

- **事务名称**：用户可手动指定事务的名称，当多个事务的时候，可区分使用哪个事务。对应注解中的属性`value`、`transactionManager`
- **隔离级别**: 为了解决数据库并发事物带来的问题，采用分级加锁处理策略。 对应注解中的属性 `isolation`
- **超时时间**: 定义一个事务执行过程多久算超时，以便超时后回滚。可以防止长期运行的事务占用资源.对应注解中的属性 `timeout`
- **是否只读**：表示这个事务只读取数据但不更新数据, 这样可以帮助数据库引擎优化事务.对应注解中的属性 `readOnly`
- **传播机制**: 对事务的传播特性进行定义，共有7种类型。对应注解中的属性 `propagation`
- **回滚机制**：定义遇到异常时回滚策略。对应注解中的属性 `rollbackFor、noRollbackFor、rollbackForClassName、noRollbackForClassName`

其中==隔离级别==和==传播机制==比较复杂，我们来细细的品一品。

### 3.1 隔离级别

#### 3.1.1 现象

 事物的隔离级别（isolation level）定义了一个事务可能受其他并发事务影响的程度。 在并发事务中，经常会引起以下问题：

* **脏读（Dirty reads）**——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。

- **不可重复读（Nonrepeatable read）**——不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
- **幻读（Phantom read）**——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

#### 3.1.2 MySQL底层支持（InnoDB事物模型）

当多个事务同时执行sql操作时，隔离级别用于平衡InnoDB的性能、可靠性、并发性、结果的可再现性。可以通过 `set_transaction` 进行单个用户连接的隔离级别设置。通过show variables like ‘tx_isolation’查看当前使用的隔离级别。加上server启动参数--transaction -isolation 或者在 配置文件中设定server level的隔离级别。InnoDB使用不同的锁策略来实现对不同事务隔离级别的支持。具体如下：

| 隔离级别                   | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别（Spring独有的）                 |
| ISOLATION_READ_UNCOMMITTED | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读 |
| ISOLATION_READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生 |
| ISOLATION_REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 |
| ISOLATION_SERIALIZABLE     | 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的 |



### 3.2 传播行为

**事务的传播行为（propagation behavior）**是指一个事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。Spring在 `org.springframework.transaction` 包下的`TransactionDefinition`接口定义了七种传播行为：

| 传播行为                  | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |



## 参考资料

* [spring事务详解（一）初探事务.https://www.likecs.com/default/index/url?u=aHR0cHM6Ly9pLmNuYmxvZ3MuY29tL0VkaXRQb3N0cy5hc3B4P3Bvc3RpZD05NTQ5NTM1](https://www.likecs.com/default/index/url?u=aHR0cHM6Ly9pLmNuYmxvZ3MuY29tL0VkaXRQb3N0cy5hc3B4P3Bvc3RpZD05NTQ5NTM1)

* [Mysql技术内幕-InnoDB存储引擎]()
* [InnoDB Locking and Transaction Model.MySQL官网](https://dev.mysql.com/doc/refman/5.6/en/innodb-locking-transaction-model.html)

