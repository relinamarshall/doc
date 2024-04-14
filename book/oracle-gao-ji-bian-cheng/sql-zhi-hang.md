---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# SQL执行

## Oracle架构基础

术语**"Oracle数据库"**即用来指存储在硬盘上的内部存有数据的数据库文件，也指用来管理这些文件的内存结构。事实上，术语**"数据库"**归属与数据文件而**"实例"**则归属于内存结构。一个实例由系统**全局内存区域(system global area,SGA)**以及一系列的后台进程组成。每一个连接到数据库的用户都是通过一个客户端进程来进行管理的。客户端进程是与服务器进程相联结的，每一个服务器进程都会被分配一块私有的内存区域，称为**程序共享内存区域(program global area)**或**进程共享内存区域(process global area,PGA)**。

<figure><img src="../.gitbook/assets/image (11) (1).png" alt=""><figcaption><p>Oracle实例和数据库关系图</p></figcaption></figure>

## SGA-共享池

共享池是内存中最关键的部分之一，特别是对于SQL的执行来说。你写SQL语句并不仅仅只会影响这句SQL语句本身。数据库中正在执行的所有SQL语句结合在一起将会因为它们对共享池的影响而对总的性能和可扩展性带来巨大的影响。

**共享池是Oracle缓存程序数据的地方**。执行过的每一句SQL语句在共享池中都存有解析后的内容。共享池中存储这些语句的地方称为库**高速缓存(library cache)**。在解析每一句语句之前，Oracle都会检查库高速缓存看其中是否已经存在同样的语句。如果存在，Oracle就会直接从缓存中读取并使用该信息而不是将同样的语句再解析一遍。对于你运行的任何PL/SQL代码，情况也是这样的。非常妙的一点是，不管有多少个用户想要执行同样的SQL语句，Oracle通常都会只解析该语句一次，然后在所有用户之间共享。

SQL语句并不是存放在共享池中的唯一内容，Oracle所使用的系统参数也将存放在其中。在一块被称为**数据字典高速缓存(dictionary cache)**的区域，Oracle还会存储所有的数据库对象信息。一般来说，Oracle将你能想到的几乎所有东西都存在共享池中，这使得共享池成为一块非常繁忙和重要的内存区域。

因为分配给共享池的内存区域是有限的，所以当新语句执行时，原先已经加载的语句就不能长时间地放在其中。有一种算法叫做**最近最少使用算法(Least Recently Used，LRU)**，它可以用来管理共享池中的对象，类似于一种先进先出(First In First Out，FIFO)系统，其基本思想是保留那些使用最频繁的以及最近使用的语句。与直接的先进先出方法不同的是，同样一个语句被使用的频繁程度将会影响它在共享池中保留的时间，将用新的信息覆盖较旧的区域。

写SQL语句的时候，需要牢记于心的一点是:为了最高效地使用共享池，语句需要可以共享。如果你所写的每一句语句都是唯一的，基本上就违背了设立共享池的初衷。语句共享性越差，你将看到对于响应时间的影响也就会越大。在下节中将展示解析的过程究竟需要多高的代价。

## 库高速缓存

在执行每一句SQL语句时必然会发生的事情就是它必须被解析并载入到库高速缓存中。**库高速缓存是共享池中用来保存之前已经解析过的语句的区域**。解析包括验证语句的语法、检验提及的对象，以及确认该对象的用户权限。如果这些检查都通过了，下一个步骤就是要看看这个语句之前是不是执行过。如果是，Oracle将取回之前解析的信息并重用。这种类型的解析被称为是**软(soft)解析**。如果该语句之前没有被执行过，那么Oracle就将执行所有的工作来为当前的语句生成执行计划，并且将它存在缓存中以便将来重用。这种类型的解析被称为**硬(hard)解析**。

硬解析比软解析需要Oracle做的工作要多得多。每次进行硬解析的时候，在能够实际执行语句之前，Oracle必须收集它所需的所有信息。为了得到所需信息，Oracle需要对数据字典进行一连串的查询。想要看到Oracle在硬解析过程中都干了些什么，最简单的办法就是打开扩展的SQL追踪,执行一个语句然后查看追踪数据。扩展的SQL追踪能抓取执行过程中所发生的每一个活动因此你不仅能看到你所执行的语句，还能看到Oracle必须要执行的每一个语句。

<table><thead><tr><th width="173">表</th><th width="114">查询次数</th><th>目的</th></tr></thead><tbody><tr><td>access$</td><td>1</td><td>一个依赖对象对于其父对象所使用的权限</td></tr><tr><td>ccol$</td><td>10</td><td>约束列特有的数据</td></tr><tr><td>cdef$</td><td>3</td><td>约束特有的定义数据</td></tr><tr><td>col$</td><td>1</td><td>表中列特有的数据</td></tr><tr><td>dependency$</td><td>1</td><td>对象间依赖性</td></tr><tr><td>hist_head$</td><td>12</td><td>柱状图头数据</td></tr><tr><td>histgrm$</td><td>3</td><td>柱状图说明</td></tr><tr><td>icol$</td><td>6</td><td>索引列</td></tr><tr><td>ind$,ind_stats$</td><td>1</td><td>索引，索引统计</td></tr><tr><td>obj$</td><td>8</td><td>对象</td></tr><tr><td>objauth$</td><td>2</td><td>表授权</td></tr><tr><td>seg$</td><td>7</td><td>所有数据库段的映射</td></tr><tr><td>syn$</td><td>1</td><td>同义词</td></tr><tr><td>tab$,tab_stats$</td><td>1</td><td>表，表统计</td></tr><tr><td>user$</td><td>2</td><td>用户定义</td></tr></tbody></table>

在硬解析过程中总共需要执行**59次**对系统对象的查询。而同样一个语句的软解析不需要对系统对象进行任何查询，因为在初始的硬解析过程中所有的事情都已经做过了。**硬解析需要占用的运算时间是0.060374秒，而软解析为0.000095秒**。因此，软解析比硬解析要更令人向往。不要总是觉得硬解析无关紧要，你也看到了，它确实是很要紧的。

## 完全相同的语句

为了确定一条语句是不是之前执行过,Oracle会去检查库高速缓存看是不是存在同样的语句。你可以通过查询**v$sql视图**来查看当前存放在库高速缓存中的语句。这个视图列出了共享SQL区域的统计信息，并且包含了最初输入的SQL文本每个子成员的一行。

{% code lineNumbers="true" %}
```sql
select * from productinfo where productid = '6'

SELECT * FROM PRODUCTINFO WHERE PRODUCTID = '6'

SELECT /*a_comment*/ * FROM PRODUCTINFO WHERE PRODUCTID = '6'

--发现有三条
SELECT 
    SQL_TEXT,SQL_ID,CHILD_NUMBER,HASH_VALUE,ADDRESS,EXECUTIONS
FROM V$SQL 
WHERE UPPER(SQL_TEXT) LIKE '%PRODUCTINFO%'
```
{% endcode %}

尽管这3条语句返回完全相同的结果，但Oracle认为它们是不同的。这是因为，在执行一条语句时，Oracle会首先将**字符串转换为散列值**。这个散列值就作为该语句存放到库高速缓存时的关键字。当其他语句执行时，它们的散列值就会与已有的散列值做比较来寻找匹配。那么，既然都返回相同的结果，为什么这3条语句会产生不同的散列值呢?这是因为它们并不是严格一致的。**小写字母与大写字母是不同的**。在一个语句中**添加与不添加注释也是不同的**。任何不同都会使得语句具有不同的散列值，从而导致Oracle对语句进行硬解析。

这也就是为什么在你的SQL语句中使用绑定变量而不是常量会如此重要的原因。当使用绑定变量时，即使你改变了绑定变量的值，Oracle还是可以共享这个语句。

{% code title="SQLPlus" lineNumbers="true" %}
```sql
SQL> variable v_id varchar2(20) 
SQL> exec :v_id := '1'
SQL> select * from productinfo where productid = :v_id;

SQL> exec :v_id := '2'
SQL> select * from productinfo where productid = :v_id;

SQL> exec :v_id := '3'
SQL> select * from productinfo where productid = :v_id;
```
{% endcode %}

```sql
--发现只有一条
SELECT SQL_TEXT,SQL_ID,CHILD_NUMBER,HASH_VALUE,ADDRESS,EXECUTIONS
FROM V$SQL 
WHERE UPPER(SQL_TEXT) LIKE '%PRODUCTINFO%';
```

如果执行查询的时候直接使用实际值(1,2,3)，那么将会是3条不同的语句。记住利用**绑定变量**的优势及使用同样的SQL语句来编写SQL。需要更少的硬解析就意味着你的应用将会表现得更好并且可扩展性更强。

最后还需要理解的重要的一点是锁存器。**锁存器是Oracle为了读取存放在库高速缓存或者其他内存结构中的信息时必须获得的一种锁**。锁存器可以保护库高速缓存避免被两个同时进行的会话修改，或者一个会话正要读取的信息被另一个会话修改而导致的损坏。在读取库高速缓存中的任何信息之前，Oracle都会获得一个锁存器，其他所有会话都必须等待，直到该锁存器被释放才能获得锁存器以完成工作。

**锁存器与典型的锁是不同的，它并不是一个队列。**Oracle为了检查你要执行的SQL语句是否已经存在而要在库高速缓存中获取一个锁存器，它将会检查锁存器是否空闲。如果锁存器是空闲的，它将获取该锁存器做它需要做的工作，然后释放锁存器。但是，如果锁存器已经被使用了，Oracle将会做一件被称为**自旋(spinning)**的事情。想象一下，这就好像一个小朋友坐在车的后座上一次又一次地问“我们到了吗?"。Oracle基本上将会**迭代轮询**，继续去看锁存器是否可用。在这段时间里，Oracle积极使用CPU去进行检查，而你的查询实际上是“被挂起的，在锁存器可用之前不会做任何事情。

如果在一段时间的自旋之后锁存器仍然不可用(**Oracle将会不断轮询直到隐藏参数\_spin\_count指定的次数，默认值为2000次**)，该请求就会被暂时停止，你的会话也不得不排到其他需要使用CPU的会话后面去。它需要再次等待轮到它使用CPU的时间来检查锁存器是否可用。这个迭代过程将会不断重复直到锁存器可用。你不仅要排队等待锁存器可用，而且在排队等待时候另外一个会话完全有可能获得了锁存器。如果很多个会话同时都需要获得锁存器的话，就会非常耗费时间。

需要记住的主要一点是锁存器是串行的。Oracle需要获得锁存器的频率越高，就越有可能发生争夺，也就等待更长的时间。这对性能和可扩展性的影响是巨大的。因此，正确编写你的代码使其较少使用锁存器(也就是硬解析)是非常关键的。

## SGA-缓冲区缓存

缓冲区缓存是系统全局内存区域(SGA)最大的部分之一。在数据库块从硬盘中读取出来后或写入硬盘之前，它用来存储数据库块。块是Oracle进行操作的最小单位。块中含有表数据行或索引条目，一些块还会包含用来排序的临时数据。Oracle必须读取块来获得SQL语句需要的数据行。块的典型大小有4KB、8KB或16KB，块大小的唯一限制因素取决于你所使用的操作系统。&#x20;

每个块都有一定的结构。块中的一些区域含有Oracle用来进行管理的块本身的信息。有表示块的类型的信息(表、索引等)、稍许关于该块的事务处理信息、在磁盘中该块所处的地址信息存储数据在该块中的表的信息，以及块中所包含的行数据信息。块中剩下的部分用于存放实际的数据或者是新数据可以存进来的空白空间。关于缓冲区缓存是如何被划分为多个池以及具有可变的块大小的还有更多细节。为了讨论简单起见，这里仅讨论一个较大的具有单一块大小的默认缓冲池。

Oracle使用最近最少使用(LRU)算法来管理其中的信息。缓冲区缓存同样使用LRU列表来使Oracle知道哪些块是最近使用过的，以便在需要的时候为新的块腾出所需空间。除了LRU列表以外，Oracle还在缓冲区缓存中为每个块维护了一个**接触计数器(touch count)**。这个计数器表明了该块被使用的频繁程度，接触计数高的块要比接触计数低的块在缓存中保留的时间长。

同样类似于共享池的是，在验证块是否在缓冲区缓存中以及更新LRU信息和接触计数的时候都必须获得锁存器。有一种方法可以有助于Oracle使用较少的锁存器，这就是在写SQL语句的时候使得该语句在获取能够满足查询需要的数据行时访问尽可能少的数据块。关于如何才能做到这一点的讨论将会贯穿本书余下的章节。但现在请记住，如果在写SQL语句时唯一关心的就是能够得到功能上正确的答案，你可能写出需要数据块访问效率较低的语句，从而需要更多地使用锁存器。需要的锁存器越多，也就越可能发生争夺，你的应用响应时间越长，可扩展性越差。

执行一句数据块不在缓冲区缓存中的查询会需要Oracle访问操作系统以获取这些块，然后在将结果集返回给你之前把它们放入缓冲区缓存中。一般来说，任何一个能够满足SQL查询的包含数据行的块都必须出现在缓冲区缓存中。当Oracle确定一个数据块已经在缓冲区缓存中的时候，这样的访问被认为是一次**逻辑读取**。如果该块必须从磁盘中获取，则被认为是**物理读取**。因为块已经在内存中了，完成一次逻辑读取的响应时间要比物理读取短。下面是3种不同场景下多次执行同一句语句的区别。

> 语句执行之前清空了共享池和缓冲区缓存。这意味着该语句将会被硬解析，查询所需数据的块(以及为了完成硬解析所需的所有关于系统对象的查询)需要从磁盘上物理读取。

{% code title="SQLPlus" lineNumbers="true" %}
```sql
ALTER SYSTEM SET EVENTS 'IMMEDIATE TRACE NAME FLUSH_CACHE';
ALTER SYSTEM FLUSH SHARED_POOL;
SET AUTOTRACE TRACEONLY STATISTICS;
SELECT * FROM SCOTT.EMP WHERE DEPTNO = 20;

统计信息
----------------------------------------------------------
        922  recursive calls
          0  db block gets
        182  consistent gets
         24  physical reads
          0  redo size
       1238  bytes sent via SQL*Net to client
        524  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
         25  sorts (memory)
          0  sorts (disk)
          5  rows processed

SET AUTOTRACE OFF;
```
{% endcode %}

> 如果仅清空缓冲区缓存

{% code title="SQLPlus" lineNumbers="true" %}
```sql
ALTER SYSTEM SET EVENTS 'IMMEDIATE TRACE NAME FLUSH_CACHE';
SET AUTOTRACE TRACEONLY STATISTICS;
SELECT * FROM SCOTT.EMP WHERE DEPTNO = 20;

统计信息
----------------------------------------------------------
          0  recursive calls
          0  db block gets
          8  consistent gets
          6  physical reads
          0  redo size
       1238  bytes sent via SQL*Net to client
        524  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          5  rows processed

SET AUTOTRACE OFF;
```
{% endcode %}

> 共享池和缓冲区缓存都不清空

{% code title="SQLPlus" lineNumbers="true" %}
```sql
SELECT * FROM SCOTT.EMP WHERE DEPTNO = 20;

统计信息
----------------------------------------------------------
          0  recursive calls
          0  db block gets
          8  consistent gets
          0  physical reads
          0  redo size
       1238  bytes sent via SQL*Net to client
        524  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          5  rows processed

SET AUTOTRACE OFF;
```
{% endcode %}

从统计信息中看到,当执行一个查询仅需要进行软解析并且从缓冲区缓存中读取数据块时，执行任务所耗占的资源是最少的。

## 查询转换

在进展到执行计划的开发过程之前，会有一步被称为**查询转换**的步骤。该步骤发生在一个查询进行完语法和权限检查之后,优化器为了决定最终的执行计划而为不同的计划计算成本预估之前。换句话说，转换和优选是两种不同的任务。

在你的查询通过了**语法和权限检查之后**,查询就进入了转换查询块的转换阶段。**查询块**是通过SELECT关键字来定义的。例如，`select from employees where department_id=60`这个查询只有一个查询块_，_而`select from employees where department_id in(select department_id from departments)`有两个查询块。各个查询块要么嵌在另一个查询块中，要么以某种方式与另一个查询块相联结。查询书写的方式决定了查询块之间的关系。**查询转换的主要目的就是确定如果改变查询的写法会不会提供更好的查询计划**。

查询转换可能会重写你的查询，这意味着你写的SQL语句并不一定是最终执行计划的依据。虽然这通常是有益的，因为查询转换器能够优化执行计划，但也可能导致你期望的行为与实际不符，特别是在涉及特定部分执行顺序时。因此，了解查询转换的工作方式对于编写能够产生预期行为的SQL语句至关重要。

查询转换器可能会改变你最初所写查询的结构，只要这样的改变不会影响结果集。任何可能会导致结果集与原有查询语法不同的改变都不在考虑之列。最常进行的改变就是将独立的查询块转换为直接联结。

{% code lineNumbers="true" %}
```sql
select * from employees 
where department_id in (select department_id from departments);

--可能会被转换为下面的语句
select e.* from employees e,departments d 
where e.department_id = d.department_id;
```
{% endcode %}

结果集没有改变，但是从优化器的角度来看转换后的版本执行计划的选择将会更好。通过查看执行计划来了解是否发生了查询转换。

以下几种基本的转换能够应用到特定的查询中：

* 试图合并
* 子查询解嵌套
* 谓语前推
* 使用物化视图进行查询重写

## 视图合并

**视图合并是一种能将内嵌或储存式视图展开为能够独立分析或者与查询剩余部分合并成总体执行计划的独立查询块的转换。**改写后的语句基本上不包含视图。一个如`select * from my_view`的语句将会被改写为好像直接输入了视图源一样。视图合并常常发生在当外部查询块的谓语包括下列项的时候。&#x20;

* 能够在另一个查询块的索引中使用的列。&#x20;
* 能够在另一个查询块的分区截断中所使用的列。&#x20;
* 在一个联结视图中能够限制返回行数的条件。

大多数人相信视图总是被作为独立的查询块对待并总是有自己的子查询计划,并且会在与其他查询块联结之前执行。但由于有查询转换的存在，这是不正确的。实际情况是有时视图会被独立分析并有自己的子查询计划，但通常情况下，将视图与查询的其他部分合并会对性能有很大的好处。例如，下面的查询根据视图是否进行了合并所使用的资源也会有所不同。

{% code lineNumbers="true" %}
```sql
select * 
from order o,(select sales_req_id from orders) o_view
where o.sales_req_id = o_view.sales_req_id(+)
and o.order_total>100000;
```
{% endcode %}

> 试图合并执行计划比较

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption><p>进行试图合并</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption><p>不进行试图合并</p></figcaption></figure>

第2个不进行视图合并的执行计划中视图是单独来进行处理的。该计划还通过在第3行中使用**VIEW关键字**来表明视图是保持“原样”的。通过单独处理视图，在与外部的orders表联结之前就要对orders表进行全表扫描。然而，在使用视图合并的版本中，计划运算合并为一个计划而不是让内嵌视图保持独立。这就使得所选的对于索引的访问操作效率更高，并且需要处理更少的行(26行-104行)。这个例子使用的还是一个很小的表，因此可以想象如果在查询中包含很大的表的话将会做多少工作。对视图进行合并的转换使得总体执行计划变得更佳。

这里还存在一种误解，在一个查询中，内嵌视图或标准视图将会与查询的其他部分分开并被首先考虑。这可能源于我们所接受的关于数学运算执行顺序的教育。让我们来看下面的例子。

`6 + 4 /2 = 8`

`（6 + 4）/ 2 = 5`

第二个例子中的圆括号使得加法运算先运行，而在第一个例子中根据运算顺序除法将会先做。我们所知当使用圆括号的时候其中的运算将会先发生。但是SQL语言并不遵循与数学表达式同样的规则。使用圆括号将某个查询块与其他部分分开并不能保证该查询块将会单独或首先执行。如果在所写的语句中包含了一个内嵌视图，假如想让该视图被单独考虑，你可能需要在查询块中加入**NO\_MERGE**提示以防止它被重写。事实上，我能够在代码中生成不合并视图的执行计划也正是因为使用了**NO\_MERGE**提示。通过这个提示，我就能够告诉查询转换器我想要让o\_view查询块被独立于其他外部查询块来单独考虑。使用提示的查询实际上看起来像这样:

{% code lineNumbers="true" %}
```sql
select * 
from order o,
    (select /*+ NO_MERGE*/ sales_req_id from orders) o_view
where o.sales_req_id = o_view.sales_req_id(+)
and o.order_total>100000;
```
{% endcode %}

还有其他一些情况也会阻止视图合并的发生。如果一个查询块包含解析函数或聚合函数、集合运算(例如**UNION**、**INTERSECT**、**MINUS**)，**ORDER BY**子句或者使用了**ROWNUM**，视图合并将会被禁止或限制。即使出现了上面的某些情形，你也可以通过使用**MERGE**提示来强制执行视图合并。如果你通过该提示来强制进行了视图合并，你必须确认查询的结果集在视图合并之后仍然是正确的。如果没有发生视图合并，那有可能是因为视图合并将会导致查询结果不同。一旦使用了这个提示，也就表明你认为视图合并不会影响结果。下面展示的是一个含有聚合函数的不进行视图合并的语句，以及MERGE提示的使用是如何强制视图合并发生的例子。

> MERGE提示

{% code lineNumbers="true" %}
```sql
SELECT e1.last_name,e1.salary,v.avg_salary
FROM employees e1,(
    SELECT department_id,avg(salary) 
    FROM employees e2 GROUP BY department_id
) v
WHERE e1.department_id=v.department_id AND e1.salary > v.avg_salary;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption><p>聚合函数阻止视图合并</p></figcaption></figure>

{% code lineNumbers="true" %}
```sql
SELECT /*+MERGE(v)*/ e1.last_name,e1.salary,v.avg_salary
FROM employees e1,(
    SELECT department_id,avg(salary) 
    FROM employees e2 GROUP BY department_id
) v
WHERE e1.department_id=v.department_id AND e1.salary > v.avg_salary;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption><p>MERGE显示的视图合并</p></figcaption></figure>

## 子查询解嵌套

子查询解嵌套与视图合并的相似之处在于子查询也是通过一个单独的查询块来表示的。可合并的视图与可以解嵌套的子查询之间的主要区别在于它们的位置是不同的: 子查询位于WHERE子句，由转换器进行解嵌套的审查。**最典型的转换就是将子查询转变为表联结。**如果一个子查询没有解嵌套,将会为它生成一个独立的子计划并作为总的执行计划的一部分按照优化执行速度的次序依次执行。

当子查询不相关的时候，转换查询是非常直接的。

> 不相关子查询的解嵌套转换

{% code lineNumbers="true" %}
```sql
set autotrace traceonly explain
select * from employees 
where department_id in (select department_id from departments);
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption><p>不相关子查询的解嵌套转换</p></figcaption></figure>

本例中的子查询就是简单地通过转化为表联结来合并到主查询中。该查询的执行计划就好像是按照如下语句来得出的:

{% code lineNumbers="true" %}
```sql
select e.*
from employees e,departments d
where e.department_id = d.department_id;
```
{% endcode %}

通过使用**NO\_UNNEST**提示，我可以强制该查询按照所书写的方式进行优选，也就意味着将会为子查询单独生成一个子执行计划

{% code lineNumbers="true" %}
```sql
set autotrace traceonly explain
select * from employees 
where department_id in (
    select /*+NO_UNNEST*/ department_id from departments
);
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (7) (1) (1) (1) (1).png" alt=""><figcaption><p>子查询独立生成执行计划</p></figcaption></figure>

这两种执行计划的主要区别就是不进行查询转换将会选用**FILTER**运算而不是**NESTED LOOPS**连接。后面会对两种运算进行详细讲解。在这里只要注意FILTER运算是以较低效率进行两个表的匹配及联结的典型代表。如果你查看第一步的谓语信息，你可以看到子查询是保持原封不动的。这个“原版”查询在执行的时候对于**employees**表中的每一行数据，子查询必须使用**employees**表中的**department\_id**这一列来与子查询所返回的**department\_id**来进行联结。由于**employees**表中有107行，每一行都将执行一次子查询。因为Oracle使用了一种很好的被称为子查询缓存的优化功能，这两种执行计划孰优孰劣还不好说，但你很可能会看到为每一行执行一次查询的效率要比表联结的效率低。在后面的章节中详细讨论这些运算，并且评论为什么**NESTED LOOPS**联结比**FILTER**运算的效率要高。 当包含联结子查询的时候，子查询解嵌套转换会变得更复杂一些。

> 联结子查询的解嵌套转换

{% code lineNumbers="true" %}
```sql
select 
    outer.employees_id,
    outer.last_name,
    outer.salary,
    outer.department_id
from employees outer
where outer.salary > (
    select avg(inner.salary)
    from employees inner
    where inner.department_id = outer.department_id
);
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption><p>联结子查询的解嵌套转换执行计划</p></figcaption></figure>

注意在这个例子中子查询是如何转换为一个内嵌视图，然后与其外的查询合并且相联结的。相关列变成了联结条件而子查询的剩余部分用来生成内嵌视图。经过重写后的该查询的版本将会像下面这样:

{% code lineNumbers="true" %}
```sql
select 
    outer.employees_id,
    outer.last_name,
    outer.salary,
    outer.department_id
from employees outer,(
    select department_id,avg(inner.salary)
    from employees
    group by department_id
) inner
where inner.department_id = outer.department_id ;
```
{% endcode %}

子查询解嵌套的行为是由隐藏参数**\_unnest subquery**控制的,在Oracle9及后续版本中其默认值为TRUE。该参数被特指为控制相关子查询解嵌套行为的参数。与视图合并类似，从Oracle10版本开始，转换后的查询将会由优化器进行复核，然后根据成本的评估来确定解嵌套后的版本是不是成本更低。

## 谓语前推

谓语前推用来将谓语从一个内含查询块中应用到不可合并的查询块中。目标就是允许索引的使用或者让其他对于数据集的筛选在查询中能够更早地进行。一般来说，将不需要的数据行尽可能早地过滤掉都是个好主意。如果一个谓语可以通过将它前推到不可合并查询块中更早的执行在剩下的执行计划中所需要抓取的数据就会更少。更少的数据意味着要做的事情也更少，要做的事情更少则所花的时间也更少。

> 谓语前推

{% code lineNumbers="true" %}
```sql
set autotrace traceonly explain

select 
    e1.last_name, e1.salary,v.avg_salary
from employees e1,(
    select department_id,avg(inner.salary)
    from employees
    group by department_id
) v
where e1.department_id = v.department_id 
AND e1.salary > v.avg_salary
AND e1.department_id = 60;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (15) (1).png" alt=""><figcaption><p>谓语前推1</p></figcaption></figure>

{% code lineNumbers="true" %}
```sql
select 
    e1.last_name, e1.salary,v.avg_salary
from employees e1,(
    select department_id,avg(inner.salary)
    from employees
    where rownum > 1
    group by department_id
) v
where e1.department_id = v.department_id 
AND e1.salary > v.avg_salary
AND e1.department_id = 60;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (16) (1).png" alt=""><figcaption><p>谓语前推2</p></figcaption></figure>

注意第一个执行计划中的第6步。`WHERE department_id = 60`这个谓语被推进到视图中，使得可以仅计算一个部门的平均薪水。当这个谓语如第二个执行计划所示不被推进时，需要计算每个部门的平均薪水，然后当外部的查询块与内部查询块相联结的时候，再将所有`department_id`不为60的数据行剔除掉。你可以通过行数估算也可以通过第二个计划的执行成本看出，优化器认识到该计划不得不等待应用这个谓语而需要做更多的工作，因此是更昂贵并且更费时的运算。

需要指出的是在上例中用了一个小窍门来阻止谓语前推的发生。在第二个查询中所用的**rownum**伪列(增加了`WHERE rownum > 1`的谓语)扮演了禁止谓语前推的角色。**rownum不仅会禁止谓语前推，而且也会禁止视图合并**。使用rownum就如同在查询中加入了**N0\_MERGE**和**NO\_PUSH\_PRED**提示。在本例中，它使得可以指出当不发生谓语前推时的坏影响，但同时想要确定你认识到了使用**rownum**在决定执行计划的时候也会影响优化器进行选择。在使用**rownum**的时候你一定要小心，它将会使得所在的查询块既不能进行视图合并也不能前推谓语。

谓语前推不需要你做任何特别的动作就会发生，除非使用**rownum**或者**NO\_PUSH\_PRED**。这正是你所需要的!尽管在有些极端情况下谓语前推会不具优势，但这种情况少之又少。因此，一定要检查执行计划以确保谓语前推如你所期望的那样发生了。

## 使用物化视图进行查询重写

查询重写是一种发生在当一个查询或查询的一部分已经被保存为一个物化视图,转换器重写该查询以使用预先计算好的物化视图数据而不需要执行当前查询的转换。物化视图与普通视图的区别在于查询已经被执行并将结果集存入了一张表中。这样做的好处就是预先计算了查询的结果并且在特定查询执行的时候可以直接调取该结果。也就是说所有的确定执行计划、执行查询以及收集所有数据的工作都已经做完了。因此，当同样的查询再一次执行的时候，就不需要再做一遍了。查询转换器会将查询与可用的物化视图相匹配,然后重写该查询以直接从物化结果集中选取查询数据。

> 使用物化视图进行查询重写

{% code lineNumbers="true" %}
```sql
set autotrace traceonly explain;

select p.prod_id,p.prod_name,t.time_id,t.week_ending_day,
    s.channel_id,s.promo_id,s.cust_id,s.amount_sold
from sales s ,products p,times t
where s.time_id = t.time_id AND s.prod_id = p.prod_id;

set autotrace off;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (17) (1).png" alt=""><figcaption><p>使用物化视图进行查询重写1</p></figcaption></figure>

{% code lineNumbers="true" %}
```sql
create materialized view sales_time_product_mv
enable query rewrite as
select p.prod_id,p.prod_name,t.time_id,t.week_ending_day,
    s.channel_id,s.promo_id,s.cust_id,s.amount_sold
from sales s ,products p,times t
where s.time_id = t.time_id AND s.prod_id = p.prod_id;

set autotrace traceonly explain;
select p.prod_id,p.prod_name,t.time_id,t.week_ending_day,
    s.channel_id,s.promo_id,s.cust_id,s.amount_sold
from sales s ,products p,times t
where s.time_id = t.time_id AND s.prod_id = p.prod_id;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (19) (1).png" alt=""><figcaption><p>使用物化视图进行查询重写2</p></figcaption></figure>

{% code lineNumbers="true" %}
```sql
select /*+rewrite(sales_time_product_mv)*/
    p.prod_id,p.prod_name,t.time_id,t.week_ending_day,
    s.channel_id,s.promo_id,s.cust_id,s.amount_sold
from sales s ,products p,times t
where s.time_id = t.time_id AND s.prod_id = p.prod_id;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (20) (1).png" alt=""><figcaption><p>使用物化视图进行查询重写2</p></figcaption></figure>

为了保持例子的简单性，我使用了一个**REWRITE**提示来打开查询重写转换。你同样也可以让查询重写自动发生。当发生重写时，执行计划仅仅列出了对物化视图的完全访问而不是初始生成结果集的时候所需要执行的整个运算集。你可以想象，对于具有很大结果集的复杂查询，时间的节省是非常显著的，尤其是当查询包含聚合的话。关于查询重写及物化视图的更多信息，高级查询重写的内容请自行搜索文章查看。

## 确定执行计划

当发生硬解析的时候，Oracle将会确定哪个执行计划对于该查询是最优的。**执行计划就是Oracle访问你的查询所使用的对象并返回相应结果数据将会采用的一系列步骤**。为了确定该计划，Oracle将要收集并使用很多信息。Oracle确定执行计划所用到的最关键的信息之一就是统计信息。可以针对对象(如表和索引)收集统计信息，也可以收集系统统计信息。系统统计信息向Oracle提供数据块读取的平均速度以及其他更多的信息。所有这些信息都用来帮助Oracle检查查询执行的不同场景并且确定哪一个能实现最佳的性能。

在Oracle进行完一个SQL语句的语法和权限检查之后,它会使用从数据字典中收集的统计信息来计算为了得到查询所需要的结果可能会使用到的每一个运算以及运算组合的成本值。成本是Oracle用来对同一查询的不同执行计划进行比较的一个内部值，成本最低的选项被认为是最佳的。例如，一个语句可以通过使用全表扫描或索引来执行。通过使用统计信息、参数以及其他信息，Oracle就会确定哪种方法将会具有最快的执行时间。 由于Oracle在确定执行计划过程中的一个主要目标就是为所解析出的SQL语句选择一系列具有最短响应时间的运算，统计信息越准确，Oracle也就越可能计算出最佳执行计划。

**优化器是Oracle内核中的代码路径，负责为查询确定最佳执行计划**。因此，当谈到统计信息的时候，实际上说的是优化器是如何来使用统计信息的。

> Employees表的统计信息

{% code lineNumbers="true" %}
```sql
-- 查看表的统计信息
SELECT * FROM ALL_TAB_STATISTICS
WHERE TABLE_NAME = 'LOG_USER';

select * from USER_TABLES where table_name ='LOG_USER';

select * from ALL_TABLES where table_name ='LOG_USER'

select * from DBA_TABLES where table_name ='LOG_USER';

-- 查看表的所有字段统计信息
SELECT COLUMN_NAME, NUM_DISTINCT, NUM_NULLS, DENSITY, LOW_VALUE, HIGH_VALUE
FROM ALL_TAB_COL_STATISTICS
WHERE TABLE_NAME = 'LOG_USER';

select * from all_tab_cols where table_name ='LOG_USER';

-- 查看表的所有索引统计信息
SELECT INDEX_NAME, DISTINCT_KEYS, LEAF_BLOCKS, CLUSTERING_FACTOR
FROM ALL_INDEXES
WHERE TABLE_NAME = 'LOG_USER';

select * from ALL_INDEXES  where table_name ='LOG_USER';

select * from ALL_IND_COLUMNS where table_name ='LOG_USER';
```
{% endcode %}

就像在棒球中的统计信息一样，优化器所使用的统计信息具有一定的预测性质。例如，一个棒球选手的平均击球率为0.333，你可以预期他每3次击球会有一次击中。事实并不一定总是这样的，但这是大多数人所信赖的指标。同样，优化器也依赖于num\_distinct列的统计信息来计算一个列中的值出现的频率有多高。默认情况下，总是假设所有值出现的比例与其他任何值都是一样的。如果你正在看一个列color的num\_distinct统计信息，并且该值设定为10，这就意味着优化器会认为有10种可能的颜色，每种颜色在表的全部行中出现的比率为1/10。因此，假设优化器正在解析下面的查询:&#x20;

`select * from widgets where color ='BLUE'`&#x20;

优化器可以选择读取整个表(**全表访问运**算)或者它可以选择使用索引(**通过索引ROWID来访问表**)。但它是如何来确定哪种方式是最好的呢?是通过使用统计信息来确定的。例如使用表明widgets表中有多少行的统计信息(`numrows=1000`)以及表明color列有多少种不同颜色的统计信息(`num distinct=10`)。此处的数学运算是非常简单的:

查询应返回的结果行数=`(1/num distinct)xnumrows`=`(1/10)x1000 =100`&#x20;

如果你仔细考虑，你会发现这是非常有意义的。如果在表中有1000行，有10种不同的颜色，而你的查询仅需要找出那些颜色为蓝色的行，你将仅需要找出1/10的数据，也就是100行。这个计算得到的值被称为选择比。通过将唯一数值划分为1，你将会确定每一个单独值的选择比。

实际的计算确实是更复杂的，但希望这个简单的例子能够帮助你搞明白优化器是如何只进行一些非常直接的计算的。但如果所使用的值不准确的话即便是如此简单的一些事情也会受到很大的影响。

如果在优化器解析查询的时候统计信息已经过时了或者缺失了会怎么样呢?例如,假设统计信息所表明的不是1000行和10种不同的颜色，而是表中有100行数据和1种颜色。使用这两个值，查询需要返回的行经过计算也是100x(1/1x100)。需要返回的数据行数与最初的计算是一样的,但事实上真的是一回事吗?不是,大不相同。在第一个计算中,优化器假设1000行的10%将会返回,而在第二种情况下所返回的100行代表了表中的所有行(至少按照统计信息是这样的)。 理解统计信息的重要性将会有助于你识别那些并不是与你所写的SQL语句相关而是源于统计信息的性能问题。可能你做的所有事情都是正确的,但如果统计信息是错误的或者不够准确，不能准确反映你的数据的实际情况,你需要能够迅速查明--而不是花上好几个小时甚至几天的时间在代码上来试着去解决一个实际上不是代码问题的问题。

在这个例子中，你写了下面这样一个非常简单的查询语句:&#x20;

`select * from car_purchases where manufacturer 'Ford’ and make = 'Focus’`

该查询使用了一张存有所有美国款汽车购买信息的表。为了满足这个例子，假设每种品牌的车仅由一家公司生产。也就是说只有福特会有一款车叫福克斯。那么，这个查询的问题出在哪儿呢?它当然会返回正确的结果集,但仅返回正确的结果集并不是唯一需要回答的问题。你还需要确定优化器是否能够准确理解该查询所给出的数据。那么，让我们来看看统计信息:

`num rows(carpurchases):1000,000`

`num distinct(manufacturer):4`

`num distinct(make):1000`

因为查询中有两个不同的条件(或称为**谓语**)，首先需要计算出它们各自的选择比。生产商的选择比为1/4或0.25。品牌的选择比为1/1000或0.001。由于谓语是通过AND连接的，这个条件组合的两个选择比将会通过各自的选择比相乘得出。因此最终的选择比将会是0.00025(0.25x0.001)。这就是说优化器会确定该查询将会返回250行(0.00025x1000000)。

应为已经假设对于某一个确定的品牌只有一个制造商会生产。也就是说其他3家制造商都不会生产福克斯，包含制造商选择比的计算是有瑕疵的。事实是我们知道所有的福克斯车型都必须是由福特生产的。将条件`where manufacturer='Ford'`包含在查询中使得总的选择比降低到25%。在这个例子中，真正的选择比仅仅是车型这一列的选择比。如果仅有这一个谓语，选择比将会是1/1000或0.001，优化器所计算出的需要返回的数据行数应该是1000行而不是250行。这也就是说优化器所给出的答案被“除以”了4。你可能会看着250行与1000行的区别,然后想“那么有何高见?相差的也不是很多呀，不是吗?”让我们回到棒球的例子，来使用相同的逻辑看看能不能让你看出更多区别。如果一个选手通常的平均击中率是0.333，你又加进去了另一个没有意义的条件将他的平均击中率乘以一个0.25的因子，将会发生什么?突然之间，一个高薪的职业运动员看上去就像一个平均击中率为0.083x(0.333x0.25)的业余球员!

数字会改变一切—并不仅仅是在棒球中。优化器所进行的计算将会严重影响执行计划选择的运算。这些选择可能导致响应时间从几秒钟变为几个小时。在这个具体的例子中，你已经看到了当优化器不知道你所做的一些事情的时候将会发生什么。优化器所能做的就是插入所有的统计信息来得到一个结果。如果你知道一些优化器不知道的关于你的数据的信息，一定要写出相应的SQL代码来避免优化器误入歧途。

## 执行计划并取得数据行

在优化器确定了执行计划并保存到库高速缓存中以备日后重用之后，实际上，下一个步骤就是执行计划并取得满足你查询的数据行。在后面的章节中我将阐述更多的与计划运算以及如何阅读和理解执行计划输出的相关内容，但现在，让我们来谈谈在执行计划选定之后将会发生什么。

一个执行计划就是告诉Oracle对于每一个表对象使用哪种访问方法以及什么联结方法和联结顺序来将多个表联结到一起的一系列指令。执行计划中的每个步骤产生一个行源，然后与另一个行源相联结，直到所有的对象都被访问和联结。满足查询条件的行必须从数据库返回给应用。对于任何大小的结果集，需要返回的数据行很可能都不是在一次往返的过程中就传递给应用的。数据包将会从数据库通过网络传递给应用，直到最终所有的行都到达用户/调用者。

当你执行一个SQL查询，返回给你的是一个简单的由满足查询的数据行组成的响应，实际上是由一系列单独执行的调用完成的。为了完成响应，你的查询将会完成解析、绑定、执行、提取的步骤。在一个查询执行过程中可能有一个或多个提取调用，每次返回满足查询结果所需的一部分数据行。下图展示出了当一条SELECT语句执行的时候“后台”实际所进行的步骤。

<figure><img src="../.gitbook/assets/image (12) (1).png" alt=""><figcaption><p>在一句SELECT语句执行表象之下</p></figcaption></figure>

每次调用时客户端和数据库之间的网络往返回路将会影响语句总的响应时间。除了FETCH以外，其他的数据库调用类型在一次查询过程中都只会发生一次。正如前面所提到的，Oracle需要执行足够次数的FETCH调用来获取并返回满足查询所需要的所有结果。

一次FETCH调用将会访问缓冲器缓存中的一个或多个数据块。每次访问一个数据块的时候，Oracle都会从该块中取出数据行然后在一次回路中返回给客户端。一次返回的行数是一个可配置的值称为列大小(**Arraysize**)。列大小是一个网络回路中一次可以传输的可能行数。如果数据行太大以至于一个包都装不下，Oracle将会将这些行分解到多个包中，但即使是这样，也只需要个FETCH调用来提供特定数目的行。

列大小配置值是通过编程设置的。如何来实现将取决于你所使用的应用调用环境。在SQLPlus中，默认列大小值是15，可以通过使用**SET ARRAYSIZE N**命令改变数组大小。JDBC的默认值是10，可以使用**((0rac1eConnection)conn).setDefaultRowPrefetch(n)**来更改。请一定要看看你的应用的列大小配置值并在必要时增加。具有较大的数组大小的好处有两点:减少FETCH调用的次数以减少网络往返。可能看上去并不怎么样,但其影响之大可能吓你一跳。在下面代码块中展现了同样一个查询的逻辑读取次数是如何通过改变列大小来减少的。注意在自动追踪输出中**逻辑读取是使用consistent gets**标出的

<figure><img src="../.gitbook/assets/image (13) (1).png" alt=""><figcaption><p>列大小设置时如何影响逻辑读取的</p></figcaption></figure>

即使对于这个小的仅有664行的结果集，增大列大小所带来的不同也是显而易见的。我将设置值从15增大到45，**逻辑读取的次数就从52次降低到22次**，**网络往返次数从46次减少到16次**!这个改变与SQL语句本身没有任何关系，而与Oracle如何能够访问并返回数据行相关。

## SQL执行总览

<figure><img src="../.gitbook/assets/image (14) (1).png" alt=""><figcaption><p>SQL语句执行时所发生的步骤汇总</p></figcaption></figure>

这是一张简化了的图，但封装了所有的步骤。从一张大图的角度来说，每一个查询都必须完成解析、执行以及提取的步骤。DML语句(INSERT、UPDATE及DELETE)仅需要进行解析和执行。除了这些步骤之外，使用绑定变量的语句作为解析的一部分还需要包含一个步骤来读取绑定值。

## 小结

理解SQL语句是如何执行的会使你能够写出更高效的语句。优化器位于所有你所写的SQL语句的核心位置。在写SQL语句的时候时刻考虑优化器将会使你获得超出想象的帮助。理解优化器是获得的所有知识中最受益的内容之一。所以，如果你本来是想要从语法和特定的SQL代码开始学习的话也不要感到失望，当学习本书的整个旅程结束的时候你会发现所有这些都是很值得的。
