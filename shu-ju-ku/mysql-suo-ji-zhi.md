---
description: >-
  数据库锁定机制简单来说，就是数据库为了保证数据的一致性，而使各种共享资源在被并发访问变得有序所设计的一种规则。MySQL
  数据库由于其自身架构的特点，存在多种数据存储引擎，每种存储引擎的锁定机制都是为各自所面对的特定场景而优化设计，所以各存储引擎的锁定机制也有较大区别。
---

# MYSQL锁机制

<figure><img src="../.gitbook/assets/锁分类" alt=""><figcaption><p>锁分类</p></figcaption></figure>

## <mark style="color:purple;">1、表级锁、行级锁、页级锁</mark>

### 1.1、表级锁

表级别的锁定是 MySQL 各存储引擎中最大颗粒度的锁定机制。该锁定机制最大的特点是实现逻辑非常简单，带来的系统负面影响最小。所以获取锁和释放锁的速度很快。

当然，**锁定颗粒度大所带来最大的负面影响就是出现锁定资源争用的概率也会最高，致使并发度大打折扣。**

使用表级锁定的主要是 `MyISAM`，`MEMORY`，`CSV` 等一些非事务性存储引擎。

### 1.2、行级锁

行级锁定最大的特点就是锁定对象的颗粒度很小，由于锁定颗粒度很小，所以发生锁定资源争用的概率也最小，能够给予应用程序尽可能大的并发处理能力而提高一些需要高并发应用系统的整体性能。

虽然能够在并发处理能力上面有较大的优势，但是行级锁定也因此带来了不少弊端。

**由于锁定资源的颗粒度很小，所以每次获取锁和释放锁需要做的事情也更多，带来的消耗自然也就更大了。此外，行级锁定也最容易发生死锁。**

**使用行级锁定的主要是 `InnoDB` 存储引擎。**

### 1.3、页级锁

页级锁定是 MySQL 中比较独特的一种锁定级别。页级锁定的特点是锁定颗粒度介于行级锁定与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的并发处理能力也同样是介于上面二者之间。

使用页级锁定的主要是 `BerkeleyDB` 存储引擎。

### 1.4、总结

总的来说，MySQL 这 3 种锁的特性可大致归纳如下：

* **表级锁**：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低；
* **行级锁**：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高；
* **页面锁**：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

## <mark style="color:purple;">2、共享锁、排它锁</mark>

**InnoDB 实现了标准的行级锁**，包括两种：**共享锁（简称 s 锁）、排它锁（简称 x 锁）。**

对于共享锁而言，对当前行加共享锁，不会阻塞其他事务对同一行的读请求，但会阻塞对同一行的写请求。只有当读锁释放后，才会执行其它事物的写操作。

对于排它锁而言，会阻塞其他事务对同一行的读和写操作，只有当写锁释放后，才会执行其它事务的读写操作。

| 兼容性 | S   | X   |
| --- | --- | --- |
| S   | 兼容  | 不兼容 |
| X   | 不兼容 | 不兼容 |

**读锁会阻塞写 (X)，但是不会堵塞读 (S)。而写锁则会把读 (S) 和写 (X) 都堵塞。**

对于InnoDB在 RR（MySQL默认隔离级别）而言，对于 `update`、`delete` 和 `insert` 语句， 会自动给涉及数据集加排它锁（X）；

对于普通 select 语句，innodb 不会加任何锁。如果想在 select 操作的时候加上 S 锁 或者 X 锁，需要我们手动加锁。

```
 -- 加共享锁（S）
 select * from table_name where ... lock in share mode
 ​
 -- 加排它锁（X)
 select * from table_name where ... for update
```

用 `select … lock in share mode` 获得共享锁，主要用在需要数据依存关系时来确认某行记录是否存在，并确保没有人对这个记录进行 `update` 或者 `delete` 操作。

但是如果当前事务也需要对该记录进行更新操作，则有可能造成死锁，对于锁定行记录后需要进行更新操作的应用，应该使用 `select … for update` 方式获得排他锁。

## <mark style="color:purple;">3、加锁模式</mark>

### **3.1、记录锁（Record Locks）**

记录锁其实很好理解，对表中的记录加锁，叫做记录锁，简称行锁。比如

```
 SELECT * FROM `test` WHERE `id`=1 FOR UPDATE;
```

它会在 id=1 的记录上加上记录锁，以阻止其他事务插入，更新，删除 id=1 这一行。

需要注意的是：

* id 列必须为**唯一索引**列或**主键列**，否则上述语句加的锁就会变成**临键锁**。
* 同时查询语句必须为**精准匹配**（=），不能为 >、<、like 等，否则也会退化成**临键锁**。

在通过 **主键索引** 与 **唯一索引** 对数据行进行 UPDATE 操作时，也会对该行数据加记录锁：

```
 -- id 列为主键列或唯一索引列 
 UPDATE SET age = 50 WHERE id = 1;
```

* 记录锁是锁住记录，锁住索引记录，而不是真正的数据记录。
* 如果要锁的列没有索引，进行全表记录加锁。
* 记录锁也是排它 (X) 锁，所以会阻塞其他事务对其插入、更新、删除。

### **3.2、间隙锁（Gap Locks）**

间隙锁 是 InnoDB 在 RR（可重复读）隔离级别下为了解决 **幻读问题** 时引入的锁机制。间隙锁是InnoDB 中行锁的一种。

请务必牢记：**使用间隙锁锁住的是一个区间，而不仅仅是这个区间中的每一条数据。**

举例来说，假如 emp 表中只有 101 条记录，其 empid 的值分别是1, 2, …, 100, 101，下面的SQL：

```
 SELECT * FROM emp WHERE empid > 100 FOR UPDATE
```

当用条件检索数据，并请求共享或排他锁时，InnoDB 不仅会对符合条件的 empid 值为 101 的记录加锁，也会对 empid 大于 101（这些记录并不存在）的 “间隙” 加锁。

这个时候如果你插入 empid 等于 102 的数据的，如果那边事物还没有提交，那你就会处于等待状态，无法插入数据。

### **3.3、临键锁（Next-Key Locks）**

Next-key 锁是记录锁和间隙锁的组合，它指的是加在 **某条记录以及这条记录前面间隙上** 的锁。

也可以理解为一种特殊的间隙锁。**通过临建锁可以解决幻读的问题。**每个数据行上的非唯一索引列（该索引里面的值允许重复）上都会存在一把临键锁，当某个事务持有该数据行的临键锁时，会锁住一段 **左开右闭** 区间的数据。

需要强调的一点是，InnoDB 中行级锁是基于索引实现的，**临键锁只与非唯一索引列有关，在唯一索引列（包括主键列）上不存在临键锁。**

假设有如下表：id 主键，age 普通索引。

| id | name     | age |
| -- | -------- | --- |
| 1  | zhangsan | 10  |
| 3  | lisi     | 24  |
| 5  | wangwu   | 32  |
| 7  | zhaoliu  | 45  |

该表中 age 列潜在的临键锁有：(-∞, 10]，(10, 24]，(24, 32]，(32, 45]，(45, +∞]。

在事务 A 中执行如下命令：

```
 -- 根据非唯一索引列 UPDATE 某条记录 
 UPDATE table SET name = Vladimir WHERE age = 24; 
 ​
 -- 或根据非唯一索引列 锁住某条记录 
 SELECT * FROM table WHERE age = 24 FOR UPDATE;
```

不管执行了上述 SQL 中的哪一句，之后如果在事务 B 中执行以下命令，则该命令会被阻塞：

```
 INSERT INTO table VALUES(100, 26, 'tianqi');
```

很明显，事务 A 在对 age 为 24 的列进行 UPDATE 操作的同时，也获取了 (10, 32] 这个区间内的 **临键锁**。

这里对 **记录锁、间隙锁、临键锁** 做一个总结：

* **InnoDB** 中的 **行锁** 的实现依赖于 **索引**，一旦某个加锁操作没有使用到索引，那么该锁就会退化为表锁。
* **记录锁** 存在于包括 **主键索引** 在内的 **唯一索引** 中，锁定单条索引记录。
* **间隙锁** 存在于 **非唯一索引** 中，锁定开区间范围内的一段间隔，它是基于 **临键锁** 实现的。
* **临键锁** 存在于 **非唯一索引** 中，该类型的每条记录的索引上都存在这种锁，它是 **一种特殊的间隙锁**，锁定一段 **左开右闭** 的索引区间。

### **3.4、意向锁**

意向锁也分为 **意向共享锁（IS）** 和 **意向排他锁（IX）**。

* 意向共享(IS)锁：事务有意向对表中的某些行加共享锁（S锁）

```
 -- 事务要获取某些行的 S 锁，必须先获得表的 IS 锁。
 SELECT column FROM table ... LOCK IN SHARE MODE;
```

* 意向排他(IX)锁：事务有意向对表中的某些行加排他锁（X锁）

```
  -- 事务要获取某些行的 X 锁，必须先获得表的 IX 锁。
  SELECT column FROM table ... FOR UPDATE;
```

首先要明白四点：

* 意向共享锁（IS）和 意向排他锁（IX）都是 **表锁**。
* 意向锁是一种 **不与行级锁冲突的表级锁**，这一点非常重要。
* 意向锁是 InnoDB 自动加的， 不需用户干预。
* 意向锁是在 InnoDB 下存在的内部锁，对于MyISAM 而言没有意向锁之说。

这里就会有疑惑，既然前面已经有了共享锁（S锁）、排它锁（X锁）。那么为什么需要引入意向锁呢？它能解决什么问题呢？

可以理解 **意向锁存在的目的** 就是 **为了让 InnoDB 中的行锁和表锁更高效的共存。**

为什么这么说，来举一个例子。下面有一张表 id 是主键。

| id | name     |
| -- | -------- |
| 1  | zhangsan |
| 3  | lisi     |
| 6  | wangwu   |
| 7  | zhaoliu  |

事务 A 获取了某一行的排他锁，并未提交：

```
 SELECT * FROM users WHERE id = 6 FOR UPDATE;
```

事务 B 想要获取 users 表的表锁：

```
 LOCK TABLES users READ;
```

因为共享锁与排他锁互斥，所以事务 B 在视图对 users 表加共享锁的时候，必须保证：

* 当前没有其他事务持有 users 表的排他锁。
* 当前没有其他事务持有 users 表中任意一行的排他锁 。

为了检测是否满足第二个条件，事务 B 必须在确保 users 表不存在任何排他锁的前提下，去检测表中的每一行是否存在排他锁。很明显这是一个效率很差的做法，但是有了意向锁之后，情况就不一样了：事务 B 只要看表上有没有意向共享锁，有则说明表中有些行被共享行锁锁住了，因此，事务 B 申请表的写锁会被阻塞。这样是不是就高效多了。

这也解释就应该清楚，为什么有意向锁这个东西存在了。我们可以举个生活中的例子，再来理解下为什么需要存在意向锁。

打个比方，就像有个游乐场，很多小朋友进去玩，看门大爷如果要下班锁游乐场的门(加表锁)，他必须确保每个角落都要去检查一遍，确保每个小朋友都离开了（释放行锁），才可以锁门。

假设锁门是件频繁发生的事情，大爷就会非常崩溃。那大爷想了一个办法，每个小朋友进入，就把自己的名字写在本子上，小朋友离开，就把自己的名字划掉，那大爷就能方便掌握有没有小朋友在游乐场里，不必每个角落都去寻找一遍。例子中的“小本子”，就是意向锁，他记录的信息并不精细，他只是提醒大爷，是否有人在屋里。

这里我们再来看下 **共享(S)锁、排他(X)锁、意向共享锁(IS)、意向排他锁(IX)** 的兼容性。

|           | 意向共享锁（IS） | 意向排他锁（IX） |
| --------- | --------- | --------- |
| 意向共享锁（IS） | 兼容        | 兼容        |
| 意向排他锁（IX） | 兼容        | 兼容        |

可以看出 **意向锁之间是互相兼容的**。那你存在的意义是啥？

意向锁不会为难意向锁，也不会为难 **行级排他(X) / 共享(X)锁**，它的存在是为难 **表级排他(X) / 共享(X)锁**。

| 兼容性 | IS  | IX  | S   | X   |
| --- | --- | --- | --- | --- |
| IS  | 兼容  | 兼容  | 兼容  | 不兼容 |
| IX  | 兼容  | 兼容  | 不兼容 | 不兼容 |
| S   | 兼容  | 不兼容 | 兼容  | 不兼容 |
| X   | 不兼容 | 不兼容 | 不兼容 | 不兼容 |

注意 **这里的 排他(X) / 共享(S)锁 指的都是表锁！意向锁不会与 行级的共享 / 排他锁 互斥！**

意向锁与意向锁之间永远是兼容的，因为当你不论加行级的 X 锁或 S 锁，都会自动获取表级的 IX 锁或者 IS 锁。也就是你有 10 个事务，对不同的 10 行加了行级 X 锁，那么这个时候就存在 10 个 IX 锁。

这 10 个 IX 存在的作用是啥呢，就是假如这个时候有个事务，想对整个表加排它 X 锁,那它不需要遍历每一行是否存在 S 或 X 锁，而是看有没有存在 **意向锁**，只要存在一个意向锁，那这个事务就加不了表级排它(X)锁，要等上面 10 个 IX 全部释放才行。

### **3.5、插入意向锁**

在讲解插入意向锁之前，先来思考一个问题？

下面有张表 id 主键，age 普通索引。

| id | name | age |
| -- | ---- | --- |
| 1  | Mr   | 10  |
| 2  | Tome | 20  |
| 3  | Jon  | 30  |

首先事务 A 插入了一行数据，并且没有 commit：

```
 INSERT INTO users SELECT 4, 'Bill', 15;
```

随后事务 B 试图插入一行数据：

```
 INSERT INTO users SELECT 5, 'Louis', 16;
```

请问：

* 事务 A 使用了什么锁？
* 事务 B 是否会被事务 A 阻塞？

**插入意向锁是在插入一条记录行前，由 INSERT 操作产生的一种间隙锁。**

该锁用以表示插入意向，当多个事务在同一区间（gap）插入位置不同的多条数据时，事务之间不需要互相等待。

假设存在两条值分别为 4 和 7 的记录，两个不同的事务分别试图插入值为 5 和 6 的两条记录，每个事务在获取插入行上独占的（排他）锁前，都会获取（4，7 ] 之间的间隙锁，但是因为数据行之间并不冲突，所以两个事务之间并不会产生冲突（阻塞等待）。

总结来说，**插入意向锁** 的特性可以分成两部分：

* 插入意向锁是 **一种特殊的间隙锁** —— 间隙锁可以锁定开区间内的部分记录。
* 插入意向锁之间互不排斥，所以即使多个事务在同一区间插入多条记录，只要记录本身（**主键、唯一索引**）不冲突，那么事务之间就不会出现冲突等待。

**虽然插入意向锁中含有意向锁三个字，但是它并不属于意向锁而属于间隙锁，因为意向锁是表锁，而插入意向锁是行锁。**

现在我们可以回答开头的问题了：

* 使用插入意向锁与记录锁。
* 事务 A 不会阻塞事务 B。

**为什么不用间隙锁？**

如果只是使用普通的间隙锁会怎么样呢？我们在看事务 A，其实它一共获取了 3 把锁：

* id 为 4 的记录行的记录锁。
* age 区间在（10，15 ] 的间隙锁。
* age 区间在（15，20 ] 的间隙锁。

最终，事务 A 插入了该行数据，并锁住了（10，20 ] 这个区间。

随后事务 B 试图插入一行数据：

```
 INSERT INTO users SELECT 5, 'Louis', 16;
```

因为 16 位于（15，20 ] 区间内，而该区间内又存在一把间隙锁，所以事务 B 别说想申请自己的间隙锁了，它甚至不能获取该行的记录锁，自然只能乖乖的等待事务 A 结束，才能执行插入操作。

很明显，这样做事务之间将会频发陷入阻塞等待，插入的并发性非常之差。这时如果我们再去回想我们刚刚讲过的插入意向锁，就不难发现它是如何优雅的解决了并发插入的问题。

**总结**

* InnoDB 在 RR 的事务隔离级别下，**使用插入意向锁来控制和解决并发插入。**
* 插入意向锁是 **一种特殊的间隙锁。**
* 插入意向锁在锁定区间相同但记录行本身不冲突的情况下互不排斥。

## <mark style="color:purple;">4、乐观锁、悲观锁</mark>

### **4.1、定义**

**乐观锁**，顾名思义，乐观锁就是持比较乐观态度的锁。就是在操作数据时非常乐观，认为别的线程不会同时修改数据，所以不会上锁，但是在更新的时候会判断在此期间别的线程有没有更新过这个数据。

**悲观锁**，就是持悲观态度的锁。就在操作数据时比较悲观，每次去拿数据的时候认为别的线程也会同时修改数据，所以每次在拿数据的时候都会上锁，这样别的线程想拿到这个数据就会阻塞直到它拿到锁。

### **4.2、如何理解**

举个例子，有时候我们上公共厕所的时候要排队。如果你蹲马桶的时候开着门，外面有人排着队看着你。你会这么做吗？当然，如果在自己家里，有可能会这么干，这就是乐观锁。**虽然，能进到房间，但是有人占着坑位，该排队还是得排队。** 比如数据库提供的类似于 `write_condition` 机制，Java API 并发工具包下面的原子变量类就是使用了乐观锁的 `CAS（Compare and Swap，比较并交换）` 来实现的。

悲观锁就不同了，就相当于是进房间之后，第一件事就是把门锁上，那在门外排队等候的人不知道里面发生了什么，又着急但是又只能干等着，这就是悲观锁。比如行锁、表锁、读锁、写锁，都是在操作之前先上锁，Java API 中的 `synchronized` 和 `ReentrantLock` 等独占锁都是悲观锁思想的实现。

### **4.3、应用场景**

根据前面对两种锁的介绍，总结一下两种锁的应用场景：

* 乐观锁，它适用于读多写少的情况，也就是说减少操作冲突，这样可以省去锁竞争的开销，提高系统的吞吐量。
* 悲观锁，它适用于写多读少的情况。因为，如果还使用乐观锁的话，会经常出现操作冲突，这样会导致应用层会不断地 Retry，反而会降低系统性能。

**思考一刻：你认为秒杀场景下，并发下单和支付应该使用什么锁？**

* 使用乐观锁并发下单
* 使用悲观锁发起支付