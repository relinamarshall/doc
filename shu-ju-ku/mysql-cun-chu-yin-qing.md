---
description: >-
  存储引擎（Storage Engine），是MySQL 的专用称呼，数据库行业老大哥 Oracle，以及 SQL Server，PostgreSQL
  等都没有存储引擎的说法。
---

# MYSQL存储引擎

## <mark style="color:blue;">1、前言</mark>

MySQL 区别于其他数据库的重要特点就是，其插件式（pluggable）的表存储引擎，存储引擎的功能是接收上层传下来的指令，然后对表中的数据进行读取或写入的操作。提醒一下，**存储引擎操作的对象是表（table），而不是数据库（database）。**

**MySQL 5.5 版本之后开始采用 InnoDB 为默认存储引擎，之前版本默认的存储引擎为 MyISAM。**

MySQL 8.0 支持 9 种存储引擎，但只有 InnoDB 支持事务（Transactions），分布式事务（XA），保存点（Savepoints），即事务回滚所需要的功能。

## <mark style="color:blue;">2、存储引擎介绍</mark>

### **2.1、Federated**

Federated 存储引擎，提供了访问远程 MySQL数据库服务器上表的方法，本地并不存放数据，数据全部放到远程服务器上，本地需要保存表的结构和远程服务器的连接信息。

### **2.2、Memory**

内存型数据库引擎，所有的数据都存储在内存中，表结构保存在磁盘上，因此它的读写效率很高，但 MySQL 服务重启之后数据会丢失。它不支持事务、不支持外键。MEMORY 支持 Hash 索引或 B 树索引，其中 Hash 索引是基于 key 查询的，因此查询效率特别高，但如果是基于范围查询的效率就比较低了。而 MyISAM 和 InnoDB 两种存储引擎是基于 B+ 树的数据结构实现的。

非常适用于存储临时数据的临时表。其数据只存储于内存中，读写当然非常快，但使用时要考虑内存消耗。

### **2.3、Performance\_schema**

Performance\_schema 存储引擎，是 MySQL 数据库系统专用引擎，用户不能创建这种存储引擎的表。

系统默认数据库 performance\_schema 中的表就是采用这种存储引擎。数据库 performance\_schema 用于监控 MySQL 在一个较低级别的运行过程中的资源消耗、资源等待等情况。

### **2.4、Blackhole**

Blackhole 存储引擎，充当一个 “黑洞”，接受数据，但将其扔掉，不存储数据，类似于 Linux 系统中的 /dev/null 文件。

这么特别的黑洞存储引擎，主要作用在于：Replication 场景实现中继或过滤，验证转储文件语法，测量开启 binlog 日志所带来的额外开销，查找和存储引擎无关的其他方面的性能瓶颈。

### **2.5、CSV**

CSV 存储引擎，会在 MySQL 安装目录 data 文件夹中，和该表所在数据库名相同的目录生成一个 **.CSV** 文件，它可以将 CSV 类型的文件当做表进行处理，相比其他存储引擎的文件内容，可以直接查看和编辑。

### **2.6、Archive**

Archive 存储引擎，仅仅支持最基本的插入（insert）和查询（select）两种功能。Archive 拥有很好的压缩机制，比 MyISAM、InnoDB 存储引擎更加节约存储空间。

可以用于：日志记录，打卡记录，天气信息记录等不需要数据更新的场景。

### **2.7、MyISAM**

MyISAM 是 MySQL 5.5 之前默认的数据库引擎，读取效率较高，占用数据空间较少，但不支持事务、不支持行级锁、不支持外键等特性。因为不支持行级锁，因此在添加和修改操作时，会执行锁表操作，所以它的写入效率较低。

MyISAM 引擎保存了单独的索引文件 .myi，且它的索引是直接定位到 OFFSET 的，而 InnoDB 没有单独的物理索引存储文件，且 InnoDB 索引寻址是先定位到块数据，再定位到行数据，所以 MyISAM 的查询效率是比 InnoDB 的查询效率要高。但它不支持事务、不支持外键，所以它的适用场景是**读多写少**，且对完整性要求不高的业务场景。

### **2.8、MRG\_MyISAM**

MRG\_MyISAM 存储引擎，是一组 MyISAM 的组合，也就是说，他将 MyISAM 引擎的多个表聚合起来，但是他的内部没有数据，真正的数据依然是 MyISAM 引擎的表中，但是可以直接进行查询、删除更新等操作。

### **2.9、InnoDB**

InnoDB 是 MySQL 5.5 之后默认的存储引擎，它支持事务、支持外键、支持崩溃修复和自增列。如果对业务的完整性要求较高，比如张三给李四转账，需要减张三的钱，同时给李四加钱，这时候只能全部执行成功或全部执行失败，此时可以通过 InnoDB 来控制事务的提交和回滚，从而保证业务的完整性。

它的缺点是读写效率较差、占用的数据空间较大。但由于其支持事务处理、外键、支持崩溃修复能力和并发控制。

**如果需要对事务的完整性要求比较高，要求实现并发控制，需要频繁的更新、删除操作的数据库，那选择 InnoDB 有很大的优势。**

## <mark style="color:blue;">3、存储引擎配置</mark>

存储引擎的设置粒度是表级别的，也就是每张表可以设置不同的存储引擎，可以使用以下命令来查询某张表的存储引擎。

```
 show create table t1;
```



<figure><img src="../.gitbook/assets/mysql查询数据库引擎" alt=""><figcaption><p>mysql查询数据库引擎</p></figcaption></figure>

创建表时设置存储引擎

```
 create table t1 (i int) engine = INNODB;
```

<figure><img src="../.gitbook/assets/mysql建表时配置引擎" alt=""><figcaption><p>mysql建表时配置引擎</p></figcaption></figure>

修改表的存储引擎

```
 alter table t1 engine = MyISAM;
```

<figure><img src="../.gitbook/assets/mysql修改表引擎" alt=""><figcaption><p>mysql修改表引擎</p></figcaption></figure>
