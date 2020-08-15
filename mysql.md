# MySQL

### 1.简介

MySQL 是一种关系型数据库，在Java企业级开发中非常常用，因为 MySQL 是开源免费的，并且方便扩展。默认端口号3306

### **2.名词释义**

**索引：用于快速进行数据查找的数据结构，MySQL常用索引有b+树索引（查询速度稳定）和hash索引（快，不适合范围查询）**

**联合索引：MySQL可以使用多个字段同时建立一个索引,叫做联合索引.在联合索引中,如果想要命中索引,需要按照建立索引时的字段顺序挨个使用,否则无法命中索引.**

### **3.索引**

### 4.日志

#####  MySQL的binlog有有几种录入格式?分别有什么区别?

有三种格式,statement,row和mixed.

- statement模式下,记录单元为语句.即每一个sql造成的影响会记录.由于sql的执行是有上下文的,因此在保存的时候需要保存相关的信息,同时还有一些使用了函数之类的语句无法被记录复制.
- row级别下,记录单元为每一行的改动,基本是可以全部记下来但是由于很多操作,会导致大量行的改动(比如alter table),因此这种模式的文件保存的信息太多,日志量太大.
- mixed. 一种折中的方案,普通操作使用statement记录,当无法使用statement的时候使用row.

此外,新版的MySQL中对row级别也做了一些优化,当表结构发生变化的时候,会记录语句而不是逐行记录.

### **9.优化**

**尽量避免null**

.

### **10.其他**

**三大范式**

第一范式: 每个列都不可以再拆分. 第二范式: 非主键列完全依赖于主键,而不能是依赖于主键的一部分. 第三范式: 非主键列只依赖于主键,不依赖于其他非主键. 

# InnoDB

### 2.存储引擎对比

MyISAM和InnoDB

InnoDB支持行锁、MVCC、事务、支持外键。

MyISAM使用表锁，不支持事务和外键，支持全文索引。缓冲池只缓存索引文件，不缓存数据文件。

### 3.innodb体系结构

后台线程、内存

内存中的缓冲池：

读取页操作时，将磁盘读到的页数据放到缓冲池，下一次命中缓冲池时，直接读取缓冲池。更新时，先更新缓冲池，再刷新到数据库（checkpoint）。

缓冲池中缓存：索引页、数据页、undo页、插入缓冲、自适应哈希索引、锁信息、数据字典信息。

使用LRU对缓冲池做内存管理

**checkpoint**

用于缩短数据库的恢复时间、缓冲池不够用时，刷新脏页到磁盘、重做日志不可用时，刷新脏页

### 4.innodb关键特性

**插入缓冲**

针对使用辅助索引插入数据费时间的问题，如果缓冲池没有命中索引页，先把数据放入insert buffer中，然后以一定频率合并到辅助索引页。该辅助索引不能是唯一索引

**两次写**

对缓冲池脏页刷新到磁盘时，不直接写磁盘，先复制到doublewrite buffer，doublewrite buffer分两次分别写入共享表空间的物理磁盘和磁盘，

**自适应哈希索引**

**异步IO**

### 5.表

**主键**

不指定则：先判断是否有非空的唯一索引，否则自动创建主键

**逻辑存储结构**

所有数据逻辑的放在表空间，表空间→段→区→页

表空间存放数据，索引，插入缓冲bitmap页

### 6.索引

innode使用b+树索引

b树、b+树区别

b+树仅在叶子节点存放数据，所以每次查询操作都要走到叶子阶段。

b+树索引仅能找到被查找舒张所在数据页，然后把页读到内存，再查询。

聚集索引和非聚集索引都使用b+树，非聚集索引叶子节点存放的是对应的聚集索引。

### 7.锁机制

管理对共享资源的并发访问

**lock和latch**

锁和闩（锁的时间很短，没有死锁检查机制）

锁分为共享锁和排它锁

**一致性非锁定读**

通过行多版本控制的方式读取当前执行时间数据库中行的数据。如果读取的行正在执行delete或update，**不会等待行上的锁释放，而是读取一个快照版本。**在read commit隔离级别下，读取的是最新的快照版本。repeatable read隔离级别下读取的是事务开始前快照版本。

**一致性锁定读**

在需要对读取的操作加锁以确保数据逻辑一致性。

**自增长与锁**

为了提高插入的并发性能，锁不是在事务完成后释放，而是完成后对自增长值插入的SQL语句后立即释放。

**锁算法**

行锁、间隙锁、next-key lock（行锁+间隙锁，锁定一个范围，包含记录本身）

**死锁**

两个或以上事务在执行过程中，因争夺锁资源而陷入互相等待的现象。

数据库使用等待图进行死锁检测。在事务请求锁并发生等待时，判断是否存在回路，如果存在在有死锁，回滚undo量最小的事务。

### **8.事务**

事务的四大特性

ACID

原子性、一致性、隔离性以及持久性。

**事务分类**

扁平事务、带有保存点的扁平事务、连事务、嵌套事务、分布式事务。

扁平事务：普通事务，全执行或全回滚。

带有保存点的：允许回滚到同一事务中较早的一个状态，保存点 会随着系统崩溃而丢失。

链事务：在提交一个事务时，释放不需要的数据对象，将必要的处理上下文隐式传给下一个开始的事务。提交与开启下一事务为一原子操作。

**事务实现**

redo log保证事务的原子性和持久性，undo log保证事务的一致性。

redo记录页的物理修改操作，undo是逻辑日志，

**redo**

事务提交时，将事务的所有日志写入到重做日志文件进行持久化，才算完成提交。

**undo**

undo存放在数据库内部undo段，位于共享表空间。undo用于将数据库逻辑的恢复到原来的样子。
