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

# 索引

索引对于高效获取数据行、强制唯一性以及引用约束的有效实现都是很关键的结构。Oracle数据库提供了适合不同应用访问方法需求的多种索引。索引类型的有效选择以及索引列的关键选择对于实现最优性能有着很高的重要性。不恰当的或不正确的索引策略可能会导致性能问题。在本章中，将讨论索引的基本实现、各种不同的索引类型、应用实例以及选择最优索引类型的策略。`Oracle数据库11gR2`中可以使用的索引基于其所使用的算法从广义上可以分为3类:**B-树索引**、**位图索引**以及**索引组织表**。

位图索引的实现适用于不经常进行更新、插入和删除的列。它们更适合于具有较少唯一值(**Distinct Value**)的静态列。一个典型的例子就是在数据仓库应用中。在一张包含人口统计信息的表中的性别列是一个很好的例子,因为对于这一列只有很少的唯一值。稍后我将在本章中详述这一点。

{% hint style="info" %}
本章中引用的所有表都是引自0racle公司示例脚本SH模式中的对象。
{% endhint %}

{% embed url="https://github.com/oracle-samples/db-sample-schemas/releases/tag/v12.2.0.1" %}
示例数据库下载地址
{% endembed %}

<pre class="language-sh" data-title="示例数据库安装脚本" data-overflow="wrap" data-line-numbers><code class="lang-sh">#登录
sqlplus system/<a data-footnote-ref href="#user-content-fn-1">systempw</a>@<a data-footnote-ref href="#user-content-fn-2">connect_string</a>;

#将__SUB__CWD__替换为path
perl -p -i.bak -e 's#__SUB__CWD__#'<a data-footnote-ref href="#user-content-fn-3">path</a>'#g' *.sql */*.sql */*.dat

#安装示例数据库
@<a data-footnote-ref href="#user-content-fn-4">path</a>/mksample <a data-footnote-ref href="#user-content-fn-5">password</a> <a data-footnote-ref href="#user-content-fn-6">password</a> <a data-footnote-ref href="#user-content-fn-7">password</a> <a data-footnote-ref href="#user-content-fn-8">password</a> <a data-footnote-ref href="#user-content-fn-9">password</a> <a data-footnote-ref href="#user-content-fn-10">password</a> <a data-footnote-ref href="#user-content-fn-11">password</a> <a data-footnote-ref href="#user-content-fn-12">password</a> users temp <a data-footnote-ref href="#user-content-fn-13">path</a>/log <a data-footnote-ref href="#user-content-fn-14">connect_string</a>
</code></pre>

**B-树索引**在各类应用中得到了广泛的使用。有很多种索引类型如**分区索引**、**压缩索引**、**基于函数的索引**都实现为**B-树索引**。特殊的索引类型如**索引组织表**以及索引组织表上的**次级索引**同样也实现为**B-树索引**。

## 理解索引

全表扫描访问路径就一定是不好的吗?不一定。一种访问路径的效率对于不同的SQL语句构造、应用数据、数据的分布以及环境都是不同的。没有一种访问路径适用于所有的执行计划。在某些情况下，全表扫描访问路径要好过基于索引的访问路径。我将讨论索引的选用、选择索引列所需要考虑的因素以及对空值子句的特殊考虑。

### 什么时候使用索引

一般来说，如果SQL语句中声明的谓语是选择性的，即应用所声明的谓语将会获取很少的数据行，那么基于索引的访问路径性能将会更好。典型的基于索引的访问路径通常包含下面3步。

* (1)遍历索引树并在将SQL语句中的谓语应用到索引列后收集叶子块的行编号。
* (2)使用行编号从表数据块中获取数据行。
* (3)在所获取的数据行上应用其余的谓语来得出最终结果集。

如果在第1步中返回了大量的行编号,第2步访问表数据块的代价就会更高。对于来自索引叶子块的每一个行编号，都需要访问表数据块。并且这可能会导致多次物理IO从而引起性能问题同时,从物理上来说1次只能访问1个表数据块,进一步放大了性能问题。例如,考虑下面代码清单中仅使用一个谓语`country='Spain'`来访问Sales表的SQL语句,第5步返回的数据行数估计为7985行因此，预计要从执行步骤中获取7985个行编号并且表数据块必须至少访问7985次以取出数据记录。如果数据块还没有在缓冲区缓存中，其中一些表数据块的访问就可能会导致物理IO。因此 对于这个特定的例子，基于索引的访问路径性能可能更差。

在下列代码清单的第1个SELECT语句中，你使用提示`index(s sales_fact_c2)`来强制使用基于索引的访问路径，优化器估算该基于索引的访问路径成本为723。下一个没有这个提示的`SELECT`语句的执行计划显示优化器估算的全表扫描访问路径成本为316。显然，全表扫描访问路径估算的成本更低，也就更适合于这个SQL语句。

> 索引访问路径

{% code lineNumbers="true" %}
```sql
drop index sales_fact_c2;
create index sales_fact_c2 on sales_fact(country);

select /*+ index (s sales_fact_c2) */ count(distinct(region))
from sales_fact s where country = 'Spain';

select count(distinct(region)) from sales_fact s 
where country = 'Spain';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption><p>强制基于索引访问</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption><p>不强制索引访问</p></figcaption></figure>

来看另一个SELECT语句。在下列代码清单中,所有的3列都在SQL语句的谓语中进行了声明。因为这些谓语选择性更强，优化器估算这个SELECT语句将会返回9行,执行计划的成本为3。在下一条SELECT语句的执行过程中你强制使用了全表扫描，该执行计划的成本为315。对于这条SQL语句，基于索引的访问性能更优。

> 索引访问路径2

{% code lineNumbers="true" %}
```sql
select product,year,week from sales_fact
where product='Xtend Memory' and year = 1998 and week = 1;

select /*+ full(sales_fact) */ product,year,week from sales_fact
wh1ere product='Xtend Memory' and year = 1998 and week = 1;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>索引扫描</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>全表扫描</p></figcaption></figure>

显然，没有一个执行计划对于所有SQL语句都是好的。即使对于同一个语句，根据数据的分布以及底层硬件的不同，执行计划的行为也不同。如果数据分布改变了，执行计划就可能具有不同的成本。这恰恰就是为什么你需要收集反映数据分布的统计信息，以便优化器可以选择最优的执行计划。

此外，**全表扫描和快速全扫描进行多块读取调用**，而**索引范围扫描或索引唯一扫描进行单块读取**。多块读取的效率要比逐块进行的单块读取高得多。优化器的计算将这一区别也考虑了进去。从而能够恰当地选用基于索引的访问路径或全表访问路径。一般来说，OLTP应用将会主要使用基于索引的扫描路径而数据仓库应用将主要使用全表扫描路径。

最后要考虑的一点是并行。如果查询的谓语选择性并不是特别强，就可以使用并行来对查询进行调优，使其执行更快。一个使用并行全表扫描的执行计划的成本可能比串行索引范围扫描成本更低，从而优化器会选择更优的并行执行计划。

### 列的选择

选择进行索引的最佳列对于提高SQL访问性能是非常关键的。对于索引列的选择应该与SQI语句中所使用的谓语相匹配。下面这些就是选择最优索引列时需要考虑的内容。

* 如果应用代码访问某张表的时候在某一列上使用了**等式**或**范围谓语**，考虑对这一列进行索引就是一个很好的策略。对于多列索引，**引导列应该是在大多数谓语中被使用的列**。例如，如果你需要选择在c1列和c2列上进行索引，那么引导列就应该是在大多数谓语中使用的列。
* 考虑谓语的基数以及列的选择度也是很重要的。例如，如果某个列只有两个唯一值并且是均匀分布的，那么这一列可能就不适合建立**B-树索引**，因为在这一列上使用等式谓语将会获取50%的数据行。另一方面，如果这个列有两个唯一值但不是均匀分布的，也就是说有一个值仅在很少的数据行中出现且经常使用这个不常出现的列值来访问表，这种情况下就最好在这一列上建立索引。
* 一个例子就是一张`work-in-progress`表上经过处理的具有3个唯一值(`P`、`N`和`E`)的`Processed`列。应用通过谓语`Processed='N’`来访问这张表。**在`Processed`列中仅有几行状态为`'N'`的未处理数据，因此通过索引来访问是最优的**。**但谓语为`Processed='Y'`的查询就不应该使用索引，因为使用这个谓语几乎所有行都将被取出**。可以使用直方图信息来使得优化器可以根据使用的是常量或绑定变量来选择最优执行计划。
* 考虑列的排序，并安排好索引中列值的顺序以使其与应用访问模式相适应。例如，在SH模式下的`Sales`表中，`Prod_id`列的选择度为`1/72`，而`Cust_id`列的选择度为`1/7059`。看上去似乎`Cust_id`列是进行索引更好的候选因为该列的选择度较低。但是,如果应用声明了`Prod_id`列上的等式谓语而没有在谓语中声明`Cust_id`列，那么`Cust_id`列就不必进行索引，即使`Cust_id`列具有更好的选择度。如果应用在`Prod_id`列和`Cust_id`列都应用了谓语，那么最好是在这两列上都建立索引并将`Cust_id`列作为引导列。**需要考虑的是列是否在谓语中使用,而不是完全依赖于列的选择度。**
* 你还需要考虑索引的成本。插入、删除以及更新(更新索引列)都需要维护索引，意味着如果在`Sales`表中插入了一行，那么就需要在索引中加入一对与这一行数据相匹配的新值。如果索引列需要进行大量更新的话，这个索引的维护成本就更高，因为建有索引的列的更新会导致索引内部发生删除和插入。这也有可能会引入额外的资源争夺点。
* 考虑列的长度。建有索引的列越长，索引也就越大。索引的成本就可能会超过由索引带来的全部好处。较大的索引尺寸也会增加撤销(`Undo`)和重做(`Redo`)区的大小。
* 在多列索引中，如果引导列只有很少的唯一值，考虑将该索引建立为压缩索引。这些索引的尺寸将会变得更小，因为压缩索引中不保存重复值。本章后面将会讨论压缩索引。
* 如果谓语在索引列上使用函数,这一列上的索引就不会被选用。例如,谓语`to_char(prod_id)=:B1`就在`Prod_id`列上应用了一个`to_char`函数。不太可能为这个谓语选用`Prod_id`列上的常规索引，需要在`to_char(prod_id)`列上来建立一个基于函数的索引。
* 不要在需要大幅修改的列上建立位图索引。**位图索引的内部实现更适合于只有很少唯一值的只读列。**如果索引列进行了更新，位图索引的大小可能会迅速增大。对一个位图索引的过多修改还可能会导致大量的锁资源争夺。位图索引在数据仓库应用中的使用更普遍。

{% hint style="warning" %}
基数被定义为一个谓语或执行步骤预期获取的数据行数。考虑一个假设列值均匀分布的列上的简单等式谓语。通过表中的数据行数与表中唯一值的个数相除来算出基数。例如，在`Sales`表中，有`918K`行数据而`Prod_id`列有`72`个唯一值，因此`Prod_id`列上的等式谓语基数就是`918 K/72=12750`。那么，换句话说，谓语`Prod_id=:b1`预计将取出`12750`行数据。具有较低基数的列更适合作为索引的候选,因为索引的选择度要更好。对于每个值都是唯一的列，等式谓语的基数为1。**选择度是一个从0到1之间的度量值，简单定义为`1/NDV`，其中`NDV`表示唯一值的数目。**因此，一个谓语的基数可以定义为选择度乘以表中数据行数。
{% endhint %}

### 空值问题

在SQL语句中经常会声明`IS NULL`谓语。空值不存储在某个单独列的索引中,因此谓语`IS NULL`将不会使用单列索引。但空值是存储在多列索引中的。通过使用另一个虚拟列来创建多列索引,就可以在`IS NULL`子句中启用索引。

下列代码清单中，在列`n1`上建立了一个单列索引`T1_N1`。优化器没有为具有`n1 is null`谓语的`SELECT`语句选择索引访问路径。另一个索引`t1_n10`建立在表达式`(n1,0)`上，优化器选择了使用这个索引的访问路径，因为空值存储在这个多列索引中。在索引中加入一个虚拟零值，索引的大小仍然保持较小。

> 空值的处理

{% code lineNumbers="true" %}
```sql
create table t1(n1 number,n2 varchar(100));

insert into t1 select object_id,object_name 
from DBA_OBJECTS where rownum <101; 

create index t1_n1 on t1(n1);

select * from t1 where n1 is null;

PLAN_TABLE_OUTPUT                                                         |
--------------------------------------------------------------------------+
Plan hash value: 3617692013                                               |
--------------------------------------------------------------------------|
| Id  | Operation         | Name | Rows  | Bytes | Cost (%CPU)| Time     ||
--------------------------------------------------------------------------|
|   0 | SELECT STATEMENT  |      |     1 |    65 |     3   (0)| 00:00:01 ||
|*  1 |  TABLE ACCESS FULL| T1   |     1 |    65 |     3   (0)| 00:00:01 ||
--------------------------------------------------------------------------|

create index t1_n10 on t1(n1,0);

select * from t1 where n1 is null;

PLAN_TABLE_OUTPUT                                                                     |
--------------------------------------------------------------------------------------+
Plan hash value: 1744381638                                                           |
                                                                                      |
--------------------------------------------------------------------------------------|
| Id  | Operation                   | Name   | Rows  | Bytes | Cost (%CPU)| Time     ||
--------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT            |        |     1 |    65 |     2   (0)| 00:00:01 ||
|   1 |  TABLE ACCESS BY INDEX ROWID| T1     |     1 |    65 |     2   (0)| 00:00:01 ||
|*  2 |   INDEX RANGE SCAN          | T1_N10 |     5 |       |     1   (0)| 00:00:01 ||
--------------------------------------------------------------------------------------|
```
{% endcode %}

## 索引结构类型

Oracle数据库提供了多种索引类型以适应应用访问路径。这些索引类型按照索引结构大体上可以分为3大类。

### B-树索引

B-树索引实现类似于**倒置的树型结构**，包括**根节点**、**分支节点**和**叶子节点**，并且使用树遍历算法来搜索列值。**叶子节点**中包含**一对(值，行编号)值**，**值对应于索引键列**，**行编号则表示行在表数据块中的物理位置**。**分支节点**包含**叶子节点目录**以及存储在其中的**叶子节点的值范围**。根节点包含**分支节点目录**以及这些**分支节点所包括的值范围**。&#x20;

下图示出了一个数值类型列的B-树索引结构。为了便于理解，这张图对索引结构进行了概括，实际的索引结构要复杂得多。**索引的根节点保存分支节点地址以及分支块中所访问值的范围。分支节点保存叶子节点地址以及叶子块中的值范围**。

使用索引来搜索某个列值通常会导致使用**索引范围扫描**或**索引唯一扫描**。**这样的一个搜索从索引树的根节点开始，遍历分支节点，然后遍历到叶子节点。行编号从叶子节点的(列值，行编号)值对中获取，然后使用该行编号来从表数据块中获取数据行。**如果没有索引，搜索一个键值将不可避免地要对表进行全表扫描。

在下图中，如果`SQL`语句使用谓语`n1=12000`来查找一个列值`12000`，将会从根节点开始进行索引范围扫描，遍历到第2个分支节点，因为第2个分支节点保存的值范围为**11001**到**22000**。然后遍历到第4个叶子节点，因为该叶子节点保存的列值范围为**11001**到**16000**。由于索引按照排序后的顺序保存列值，索引范围扫描很快从叶子节点中找出了与`n1=12000`相匹配的列值，读取了与这个列值相联结的行编号，并且使用这个行编号从表中访问到数据行。行编号是表数据块中数据行物理位置的指针。

B-树索引适合于具有**较低选择度的列**。如果列的选择度不够低，索引扫描就会较慢。并且，选择度不够的列将会从叶子块中取出大量的行编号从而导致对表进行过多的单数据块访问。

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>B-属索引结构</p></figcaption></figure>

### 位图索引

位图索引的组织结构和实现方式与B-树索引不同，**使用位图来表示列值的行编号**。位图索引不适合需要进行大量更新的列或具有较多DML操作的表。**位图索引适合于大多数对具有较少唯一值的列进行只读运算的数据仓库表。**如果表需要进行定期加载，就像典型的数据仓库应用中的表，很重要的一点是在加载前删除位图索引，然后加载数据，最后再重建位图索引。

位图索引也可以建立在分区表上，**但必须被建成局部索引**。对于Oracle数据库11gR2版本来说，**位图索引不能建为全局索引。位图索引也不能被建立为唯一索引**。下列代码清单中，在列`Country`和`Region`上建了两个新的位图索引。`SELECT`语句声明了`Country`和`Region`列上的谓语。执行计划示出了3个主要的运算:应用谓语`country='Spain'`从位图索引`sales_fact_part_bm1`中取出了位图，应用谓语`region='Western Europe'`从位图索引`sales_fact_part_bm2`中取出了位图，然后将这两个位图使用**`BITMAP AND`**运算来进行计算以得出最终的位图。这个合成的位图被转化为行编号并使用这些行编号来访问表数据行。

> 位图索引

{% code lineNumbers="true" %}
```sql
drop index sales_fact_part_bm1;
drop index sales_fact_part_bm2;

create bitmap index sales_fact_part_bm1 on sales_fact_part(country) Local;
create bitmap index sales_fact_part_bm2 on sales_fact_part(region) Local;

select * from sales_fact_part 
where country='Spain' and region='Western Europe';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>位图索引</p></figcaption></figure>

如果索引建立在需要通过**DML**活动进行大量修改的列上，位图索引可能会引起严重的锁定问题。更新一个具有位图索引的列必须同时更新位图，而位图通常覆盖一组数据行。因此，对某一行进行更新可能会锁定位图中的一组数据行。**位图索引主要在数据仓库应用中使用，在OLTP应用中的使用很有限。**

### 索引组织表

常规的数据表都是按照堆表的形式来组织的，因为表数据行能够存储在任何表数据块中。使用主键从常规的数据表中获取一行将会进行主键索引遍历，然后使用行编号来进行表数据块访问。在索引组织表(`imndex organized tables`，`IOTs`)中，表本身被组织为一个索引，所有列存储在索引树自身中，使用主键来访问数据行将只会包含索引访问。这种使用`IOT`进行访问的方法更好因为所有列都可以通过访问索引结构来获取，从而避免了表访问。这是一种高效的访问模式，因为实现了访问次数的最小化。

在常规的表中，每一行都有一个行编号。**一旦在表中建立了一行数据，它们就不再移动(可能会有行链接或行迁移，但行的头部不会移动)。**不同的是，`IOT`数据行存储在索引结构自身中因此，数据行可能由于DML运算而迁移到不同的叶子块中，从而引起索引叶子块结构的分裂与合并。简单来说，`IOT`中的数据行没有物理行编号，而位于堆表中的数据行都会有一个固定的行编号。

`IOT`适合于具有下面特点的表。&#x20;

* **数据行长度较短的表，**数据列较少并且很短的表适合于IOT。如果数据行长度更长，索引 结构就会过大，导致比堆表使用更多的资源。
* **大多使用主键列进行访问的表，**尽管可以在IOT上建立次级索引，如果主键列较长则次级索引也可能会耗占大量资源。稍后在本节中将会讲述次级索引。

下列代码清单中，通过声明关键字`organization index`建立了`IOT Sales_iot`。注意`SELECT`语句指定了主键中的很少的列。执行计划显示列值通过索引范围扫描来获取，从而避免了表访问。如果这是一张常规堆表的话，你将会看到首先是一个索引唯一扫描访问路径,接下来是基于行编号进行一次表访问。

> 索引组织表

{% code lineNumbers="true" %}
```sql
drop table sales_iot;

create table sales_iot(
    prod_id number not null,
    cust_id number not null,
    time_id date not null,
    channel_id number not null,
    promo_id number not null,
    quantity_sold number(10,2) not null,
    amount_sold number(10,2) not null,
    primary key(prod_id,cust_id,time_id,channel_id,promo_id)
) organization index;

insert into sales_iot select * from sales;

select quantity_sold,amount_sold from sales_iot
where prod_id=13 and cust_id=2 and channel_id=3 and promo_id=999;

PLAN_TABLE_OUTPUT                                                                     |
--------------------------------------------------------------------------------------+
Plan hash value: 2718767049                                                           |
--------------------------------------------------------------------------------------|
| Id  | Operation        | Name              | Rows  | Bytes | Cost (%CPU)| Time     ||
--------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT |                   |     1 |    78 |     2   (0)| 00:00:01 ||
|*  1 |  INDEX RANGE SCAN| SYS_IOT_TOP_77826 |     1 |    78 |     2   (0)| 00:00:01 ||
--------------------------------------------------------------------------------------|
```
{% endcode %}

也可以在IOT上建立**次级索引**。常规的索引存储(**列值，行编号**)对值。但是，在IOT中数据行没有物理行编号，而是有一个(**列值，逻辑行编号**)对存储在次级索引中。这个逻辑行编号实质上是一个行数据值高效存储的主键列。**通过次级索引的访问首先使用次级索引来获取逻辑行编号，然后使用逻辑行编号通过主键`IOT`结构来访问数据行片断。**

下列代码清单中，在`IOT Sales_iot`中建立了一个次级索引`Sales_iot_sec`。`SELECT`语句声明了次级索引列上的谓语。执行计划显示了一个全索引访问，其中逻辑行编号通过对次级索引`Sales_iot sec`进行索引范围扫描访问方法获取，然后使用这个逻辑行编号进行索引唯一扫描，从`IOT`的主键中获取数据行。同时，还要注意次级索引的大小接近主键索引大小的一半，如果主键列更长的话次级索引可能会耗占更多的资源。

> IOT中的次级索引

{% code lineNumbers="true" %}
```sql
drop index sales_iot_sec;

create index sales_iot_sec on sales_iot(channel_id,time_id,promo_id,cust_id);

select quantity_sold,amount_sold from sales_iot
where channel_id=3 and promo_id=999 and cust_id=12345 
	and time_id=TO_DATE('2000-01-30','YYYY-MM-DD');

PLAN_TABLE_OUTPUT                                                                      |
---------------------------------------------------------------------------------------+
Plan hash value: 3659842553                                                            |
---------------------------------------------------------------------------------------|
| Id  | Operation         | Name              | Rows  | Bytes | Cost (%CPU)| Time     ||
---------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT  |                   |     1 |    74 |     2   (0)| 00:00:01 ||
|*  1 |  INDEX UNIQUE SCAN| SYS_IOT_TOP_77826 |     1 |    74 |     2   (0)| 00:00:01 ||
|*  2 |   INDEX RANGE SCAN| SALES_IOT_SEC     |     1 |       |     2   (0)| 00:00:01 ||
---------------------------------------------------------------------------------------|
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
select segment_name,sum(bytes/1024/1024) sz from dba_segments
where SEGMENT_NAME in ('SALES_IOT','SALES_IOT_SEC')
group by SEGMENT_NAME
```
{% endcode %}

索引组织表是一种能够有效减少数据行较短且需要进行大量`DML`和`SELECT`活动的表中额外索引的特殊结构。但如果`IOT`的主键列较长，在其中加入次级索引可能会增大索引大小、重做区大小以及撤销区大小。

## 分区索引

**索引也可以像表分区结构那样来进行分区。**有多种方法可以对索引进行分区。在分区表上可以创建局部或全局索引。并且，有多种分区方案可选，例如**范围分区**、**散列分区**、**列表分区**以及**混合分区**方案。**自Oracle数据库10g版本以来，也可以在非分区表上建立分区索引。**

### 局部索引

局部分区索引使用**`LOCAL`**关键字来建立，其分区边界与表相同。简单来说，与每个表分区相联结的有一个索引分区。因为维护操作可以在独立分区级进行，表的可用性更好。对索引分区的维护操作仅需要锁定相应的表分区而不是整张表。如果局部索引包含分区键列并且如果`SQL`语句声明了分区键列上的谓语,执行计划就仅需要访问一个或很少的索引分区。这个概念通常被称为**分区消除**(`Partition Elimination`)。如果执行计划在最少的分区中进行搜索，性能就会得到提升。在下列代码清单中，使用`Year`列上的分区键建立了一张分区表`Sales_fact_part`。在列`Product`和`Year`上创建了分区索引`Sales_fact_part_n1`。首先，`SELECT`语句只声明了`Product`列上的谓语而没有声明任何分区键列上的谓语。在这个例子中，所有5个索引分区都必须使用谓语`product='Xtend Memory'`来进行访问。执行计划中的PStart和PStop列表明为了执行这个SQL语句，访问了所有分区。

接下来,代码清单中的`SELECT`语句声明了`Product`和`Year`列上的谓语。使用谓语`Year-1998,`优化器确定了仅需要访问第2个分区，消除了对其他所有分区的访问。因为正如执行计划中的`PStart`和`PStop`列所表明的，只有第2个分区中存储了`1998`年的数据。并且，执行计划中的关键字**`TABLE ACCESSBY LOCAL INDEX ROWID`**表明数据行使用局部索引进行访问。

> 局部索引

{% code lineNumbers="true" %}
```sql
drop table sales_fact_part;
create table sales_fact_part
partition by range(year) (
    partition p_1997 values less than (1998),
    partition p_1998 values less than (1999),
    partition p_1999 values less than (2000),
    partition p_2000 values less than (2001),
    partition p_max values less than (maxvalue)
) as select * from sales_fact;

create index sales_fact_part_n1 on sales_fact_part(product,year) local;

select * from (
    select * from sales_fact_part where product = 'Xtend Memory'
) where rownum < 21;

select * from (
    select * from sales_fact_part 
    where product = 'Xtend Memory' and year=1998
) where rownum < 21;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>索引全分区扫描</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>只扫描第二分区</p></figcaption></figure>

尽管表的可用性很重要，你仍然应该考虑另外一点:如果谓语没有声明分区键列，那么在局部索引中必须访问所有索引分区以识别候选的数据行。如果分区数非常多,达到几千的量级的话这有可能会导致性能问题的出现。即使这样，你也需要衡量创建局部索引而不是全局索引所带来的影响。 创建局部索引还可以改进并行度。

将在讨论散列分区方案时再讨论这个概念。

### 全局索引

全局索引通过关键字**GL0BAL**来创建。在全局索引中，索引的分区边界与表的分区边界不一定要匹配，并且表和索引的分区键也可以不一样。

下列代码清单中，在`Year`列上创建了一个全局索引`Sales_fact_part_n1`。尽管分区列是一样的，表和索引的分区边界是不同的。接下来的`SELECT`语句声明了谓语`year=1998`来访问表，执行计划显示访问了`索引分区1`和`表分区2`。在表和索引级上都进行了分区剪裁。 **对全局索引的任何维护都将需要获得对表的较高层级的锁,从而降低了应用的可用性。相反对于局部索引的维护可以只在分区级上完成，只会影响相应的表分区。**在这个例子中，重建索引`Sales_fact_part_nl`将会需要排他模式的**表级锁**，导致应用停机。

> 全局

{% code lineNumbers="true" %}
```sql
create index sales_fact_part_n1 on sales_fact_part(year)
global partition by range(year) (
    partition by p_1998 values less than (1999),
    partition by p_2000 values less than (2001),
    partition by p_max values less than (maxvalue)
)

select * from (
    select * from sales_part where product='Xtend Memory' and year=1998
) where rownum<21;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption><p>全局索引</p></figcaption></figure>

唯一索引可以建立为不包含分区列的全局索引。但如果建立唯一局部索引，则表的分区键必须包含在局部索引中。

到目前为止所讨论的分区方案被称为**范围分区**方案。在这种方案中，每个分区存储分区列值在一定范围内的数据行。例如，`partition p_2000 values less than(2001)`子句声明了分区的上边界，从而分区`P_2000`将会存储`Year`列值小于`2001`的数据行。这个分区的下边界由前一个分区的声明子句`partition p_1998 values less than(1999)`来确定。因此`partition p_2000`将会存储`Year`列的`1999`和`2000`之间的值。

### 散列分区与范围分区

在散列分区方案中，分区键列的值使用散列算法进行散列化来确定存储数据行的分区。这种类型的分区方案适合于分区列使用人造键值填充的情况，例如分区列由顺序生成的值填充。如果列值的分布是均匀的，那么每个分区将会存储几乎相等数目的数据行。

散列分区方案还有几点额外的优点。范围分区方案有一些管理开支，因为在将来要想放入新的数据行就需要定期加入新的分区。例如，如果分区键是`order_date`列，那么就必须增加新分区(或者具有最大值的分区进行分裂)来放入将来日期的数据。而在散列分区方案中，这个开支就避免了，因为数据行在使用散列算法进行分区的各个分区上是均匀分布的。如果列值是均匀分布的则所有分区将保存近似相等数量的数据行，也就没有理由定期增加更多新的分区了。

{% hint style="info" %}
由于散列算法的属性，最好是使用二元权力，也就是2、4、8等来进行分区计数。如果你对分区进行分裂，最好是将分区数乘以2以保持近似相等的分区大小。
{% endhint %}

**散列分区表**和**索引**在应对由**唯一索引**和**主键索引**所引起的与并发性相关的性能问题时是非常有效的。典型的主键列可能会使用生成的顺序序列值来填充。因为索引按照排序后的顺序存储列值，新数据行的列值将会进入到索引最右侧的叶子块。在该叶子块满了以后，接下来插入的数据行就会进入新的最右侧叶子块中，资源争夺点也就从一个叶子块转移到了另一个叶子块。随着表的插入并发性增加，会话将会大幅修改索引的最右侧块。基本上，索引的当前最右侧叶子块将会是最主要的资源争夺点。你将会看到会话等待事件例如缓冲区繁忙等待。在RAC(`Real Application Clusters`真实应用集群)中，由于全局缓存通信成本导致了这个问题的放大，gc缓冲器繁忙将会是排在第一位的等待事件。这种类型的最右侧索引的快速增长被称为`右侧增长索引`。

**与右侧增长索引相关的并发性问题可以通过将索引散列分区到多个分区中来消除。**例如，如果索引被散列化到32个分区中，那么插入操作将会被有效地分散到32个最右侧叶子块中，因为有32个索引树(每个索引分区一个索引树)。使用散列分区方案来对表进行分区，然后再在分区后的表上创建局部索引也将具有同样的效果。

代码清单中，建立了一张散列分区的表`Sales_fct_part`，主键`id`列通过`Sfseq`序列来填充。这个表中有32个分区，对于`Sales_fact_part_n1`索引有与之匹配的32个索引分区，因为索引被定义为局部索引。接下来的`SELECT`语句使用谓语`id=1000`来访问表。执行计划中的`Pstart`列和`Pstop`列显示发生了分区剪裁，仅访问了分区`25`。优化器通过对列值`1000`使用散列函数来定位到分区`25`。

> 散列分区方案

{% code lineNumbers="true" %}
```sql
drop sequence sfseq;

create sequence sfseq cache 200;

drop table sales_fact_part;

create table sales_fact_part
partition by hash(id)
partitions 32
as select sfseq.nextval id,f.* from sales_fact f;

create unique index sales_fact_part_n1 on sales_fact_part(id) local;

select * from sales_fact_part where id = 1000;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>散列分区方案</p></figcaption></figure>

如果分区键上的数据分布是均匀的，如由序列生成的情况，那么数据行在所有分区上均匀分布。你可以使用`dbms_rowid`包来衡量散列分区表上的数据分布。下列代码清单中,使用`dbms_rowid,rowid_object`调用来得到分区的`object_id`。由于每一个分区都有自己的`object_id`，你可以按照`obiect_id`对数据行进行聚合来衡量分区间的数据行分布。输出显示所有分区具有接近相同的数据行数。

> 散列分区分布

{% code lineNumbers="true" %}
```sql
select 
    dbms_rowid.rowid_object(rowid) obj_id,
    count(*)
from sales_fact_part
group by dbms_rowid.rowid_object(rowid);
```
{% endcode %}

数据行使用散列算法在各个分区上均匀分布。在少数情况下，你可能需要预先计算某个数据行将会被存储的分区。这个知识对于改进大量数据载入到散列分区表中的情况是很有用的。从`Oracle`数据库`11gR2`版本起,如果提供了分区键值,就可以使用`ora_hash`函数来获得分区`id`。例如，对于一个具有32个分区的数据表，`ora_hash(columnname,31,0)`将会返回分区`id`。`ora_hash`函数的第2个参数为分区计数减1。在x下列代码清单中，使用`ora_hash`和`dbms_rowid.rowid_object`来显示`object_id`和散列算法输出之间的映射关系。需要提醒一点的是: `Oracle`数据库的未来版本中，在依赖于这个策略之前你需要进行测试，因为散列分区表的内部实现可能会改变。

> 散列分区算法

{% code lineNumbers="true" %}
```sql
select 
    dbms_rowid.rowid_object(rowid) obj_id,
    ora_hash(id,31,0) part_id,
    count(*)
from sales_fact_part
group by dbms_rowid.rowid_object(rowid),ora_hash(id,31,0)
order by 1;
```
{% endcode %}

从本质上来说，通过对表进行分区并将右侧增长索引创建为局部索引可以增加并发性。如果表不能进行分区，那么也可以单独对索引使用散列分区方案进行分区来解决性能问题。

## 应用特点的解决方案

Oracle数据库还提供能够匹配应用特点的索引功能。例如，某些应用可能会大量使用函数调用，此类应用中的SQL语句可以调节为使用基于函数的索引。我将讨论几种在Oracle数据库中可用的特殊索引选项。

### 压缩索引

**压缩索引是常规B-树索引的变体。这种类型的索引更适合于引导列中具有重复值的列。**通过将引导列中的重复值在索引叶子块中仅保存一次来实现压缩。数据行区的指针指向这些前置行，避免在数据行区显式存储这些重复值。如果列具有很多重复值的话，与常规的索引相比压缩索引可能小很多。在处理压缩索引的时候CPU使用率可能会略有上升，这可以很安全地忽略。

声明压缩索引的简化语法是:

{% code lineNumbers="true" %}
```sql
create index <index name> on <schema.table_name>
(col1 [,col2 …… coln])
compress n storage-parameter-clause;
```
{% endcode %}

使用`compress N`语法创建压缩索引的时候，可以声明进行压缩的引导列数目。例如，在一个3列索引中对两个引导列进行压缩，可以声明子句`compress 2`。前两列中的重复值只在前置区保存一次。你只能对引导列进行压缩。例如，你不能压缩列1和列3。下列代码清单中，通过声明`compress 2`压缩子句对两个引导列进行压缩，在列`Product`、`Year`和`Week`上创建了压缩索引`Sales_fact_c1`。在这个例子中，`Product`列和`Year`列的重复值因为声明了`compress 2`在叶子块中只保存了一次。由于这两列值的重复度较高，通过压缩这两个引导列，索引尺寸从`6MB`(常规索引)降低到了`2MB`(压缩索引)。

> 压缩索引

{% code lineNumbers="true" %}
```sql
select * from (
    select product,year,week,sale from sales_fact
    order by product,year,week
) where rownum<21;

create index sales_fact_c1 on sales_fact(product,year,week);

select 'Compressed index size (MB) :' || trunc(bytes/1024/1024,2)
from user_segments where segment_name = 'SALES_FACT_C1';
--Compressed index size (MB) :6

create index sales_fact_c1 on sales_fact(product,year,week) compress 2;

select 'Compressed index size (MB) :' || trunc(bytes/1024/1024,2)
from user_segments where segment_name = 'SALES_FACT_C1';
--Compressed index size (MB) :2
```
{% endcode %}

下图中给出了一个压缩索引叶子块的高层次概览。这个压缩索引是在`Continent`和`Country`列上的两列索引。`Continent`列的重复值在索引叶子块前置区域仅存储一次，因为索引使用`compress 1`子句来创建。使用指针来从行区域指向前置数据行。例如，`Continent`列的值`ASIA`出现在3行`[(Asia, HongKong)、(Asia, India)及(Asia,Indonesia)]`中，但在前置区仅存储了一次。这3行重用了`Continent`列的值，避免了3次显式存储该值。这种列值的重用减少了索引的大小。

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption><p>压缩索引</p></figcaption></figure>

很明显，数据自身的特点对压缩比起着非常重要的作用。如果列值重复的次数越高，则索引压缩就能得到越多的益处。如果没有重复数据，则压缩索引可能比常规索引还要大。因此，压缩索引适合于引导列具有较少唯一值的索引。`dba_indexes/user_indexes`视图中的`Compression`和`Prefix_length`列显示了索引的压缩属性。

选择对多少列进行压缩取决于列值的分布。为了确定用于进行压缩的最佳列数，可以使用`analyze index/validate structure`语句。在下列代码清单中，使用`validate structure`子句来对未压缩的索引`SALES_FACT_C1`进行分析。这个分析语句对`Imdex_ststs`视图进行填充。`Index_stats.opt_cmpr_count`列给出了最优的压缩列数。对于这个索引来说为`2`。`Index_stats.Cmpr_pctsave`列显示了对`Opt_cmpr_count`这个列进行压缩所节约的索引大小。在这个例子中，将会节约`67%`的索引空间使用。因此，使用`compress 2`子句进行压缩的索引大小将为常规未压缩索引大小的`33%`。这个估算的值为`1.98 MB`，很接近实际的索引大小。

{% hint style="warning" %}
`analyze index validate structure`语句需要对表的共享级锁，可能会引起应用停机。
{% endhint %}

> 最右压缩列数

{% code lineNumbers="true" %}
```sql
analyze index sales_fact_c1 validate structure;

select opt_cmpr_count,opt_cmpr_pctsave from index_states
where name = 'SALES_FACT_C1';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>最右压缩列数</p></figcaption></figure>

不过压缩索引也有几个限制条件。例如，对于非唯一索引的情况，所有列都可以进行压缩:对于唯一索引，除了最后一列以外都可以进行压缩。

### 函数索引

如果一个谓语在索引列上应用了函数,则优化器不会选用该列上的索引。例如,对于谓语`to_char(id)='1000'`，不会选用`id`列上的索引，因为在索引列上应用了`to_char`函数。这个限制可以通过在表达式`to_char(id)`上创建基于函数的索引来克服。基于函数的索引预存函数的结果。谓语中所声明的表达式必须与基于函数的索引所声明的表达式相匹配。

基于函数的索引也可以建立在用户自定义函数上，但这个函数必须定义为确定性函数，也就是说对于这个函数的每一次执行必须返回一致的值。不遵守这一规则的用户自定义函数不能用来创建基于函数的索引。

在下列代码清单中，`SELECT`语句使用`to_char(id)='1000'`子句来访问`Sales_fact_part`表。如果没有基于函数的索引，优化器会选择全表扫描访问计划。通过表达式`to_char(id)`增加了基于函数的索引`fact_part_fbi1`之后，优化器就为该`SELECT`语句选用了基于索引的访问路径。

> 基于函数的索引

{% code lineNumbers="true" %}
```sql
drop index sales_fact_part_fbi1;

select * from sales_fact_part where to_char(id)='1000';

create index sales_fact_fbi1 on sales_fact_part(to_char(id));

select * from sales_fact_part where to_char(id)='1000';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption><p>全表扫描</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption><p>索引范围扫描</p></figcaption></figure>

注意最后所打印出来的访问谓语`"SYS_NC00009$"='1000'`。关于基于函数索引的一些实现上的细节列于下面的代码清单。**基于函数的索引加入了一个虚拟列，所声明的表达式值作为默认值，然后在这个虚拟列上建立索引。**这个虚拟列从`dba_tab_cols`视图中可见，并且`dba_tab_cols.data_default`列显示了用来填充虚拟列的表达式。进一步的`dba_ind_columns`视图显示对虚拟列进行了索引。

> 虚拟列于基于函数的索引

{% code lineNumbers="true" %}
```sql
select data_default,hidden_column,virtual_column from dba_tab_cols
where table_name='SALES_FACT_PART' and virtual_column='YES';

select index_name,column_name from dba_ind_columns
where index_name = 'SALES_FACT_PART_FBI1';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption><p>虚拟列于基于函数的索引</p></figcaption></figure>

在增加了基于函数的索引后收集表的统计信息是很重要的。如果不收集，新的虚拟列就没有统计信息，这有可能会导致性能异常。脚本`analyze_table_sfp.sql`被用来收集表的统计信息并设置`cascade=>true`。下列代码清单给出了`analyze_table_sfp.sql`脚本的内容。

> Analyze\_table\_sfp.sql脚本

{% code lineNumbers="true" %}
```plsql
begin
    dbms_stats.gather_table_stats(
        ownname=>user,
        tabname=>'SALES_FACT_PART',
        estimate_percent=>30,
        cascade=>true
    );
end;
```
{% endcode %}

基于函数的索引也可以显式使用虚拟列来实现。在这个虚拟列上也可以加上索引。这种方法额外的好处就是你还可以使用虚拟列作为分区键来应用分区方案。在下列代码清单中，使用`virtual`关键字在表中加入了一个新的虚拟列`id_char`。然后在`id_char`列上建立了全局分区索引。`SELECT`语句的执行计划显示表使用新建的索引来访问，并且谓语`to_char(id)='1000'`被重写为谓语`id_char='1000'`以使用虚拟列。

> 自定义虚拟列与基于函数的索引

{% code lineNumbers="true" %}
```sql
alter table sales_fact_part add
(id_char varchar2(40) generated always as (to_char(id)) virtual)

create index sales_fact_part_c1 on sales_fact_part(id_char)
global partition by hash(id_char)
partitions 32;

--更新统计信息
@analyze_table_sfq

select * from sales_fact_part where to_char(id) = '1000';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption><p>自定义虚拟列与基于函数的索引</p></figcaption></figure>

### 反转键索引

反转键索引也是的右侧增长索引相关性能问题的另一个选项。在反转键索引中，列值按照逐个字符的反向顺序存储。例如，列值`12345`在索引中存储为`54321`。因为列值是按照反向顺序存储的，连续的列值将会存储在不同的索引叶子块中,从而避免了右侧增长索引所带来的资源争夺问题。但是，在表数据块中，这些列值还是存储为12345的。

反转键索引有两个问题。

* 反转键索引的范围扫描不能使用范围运算符如`between`、`<`、`>`等。这是可以理解的，因为索引范围扫描的基本假设就是列值按照`逻辑键升序`或`降序来存储`。反转键索引由于列值按照反转顺序存储，没有按照逻辑键的顺序来维护违反了这个假设。因此索引范围扫描不适用于反转键索引。
* 反转键索引可能会人为地增加物理读取的次数，因为列值被存储在很多个叶子块中，而这些叶子块可能需要读取到缓冲区缓存中来修改块。但是，这个I/O成本的增加需要与右侧增长索引所引起的并发性问题相对照来衡量。

在下面代码清单中，使用关键字`reverse`建立了一个反转键索引`Sales_fact_part_n1`。首先，具有谓语`id=1000`的`SELECT`语句使用了反转键索引，因为等式谓语可以使用反转键索引。但接下来的具有谓语`id between 1000 and 1001`的`SELECT`语句使用了全表扫描访问路径，因为索引范围扫描访问路径不能使用反转键索引。

> 反转键索引

{% code lineNumbers="true" %}
```sql
drop index sales_fact_part_n1;

create unqiue index sales_fact_part_n1 on sales_fact_part(id) global reverse;

select * from sales_fact_part where id = 1000;

select * from sales_fact_part where id between 1000 and 1001;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption><p>等值索引唯一扫描</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>范围全表扫描</p></figcaption></figure>

尤其是在`RAC(Real Application Clusters)`真实应用集群中,右侧增长索引可能会引起不能容忍的性能问题。反转键索引被引入来解决性能问题。但有时候你可能应该考虑**散列分区索引**而不是**反转键索引**。

### 降序索引

**索引默认按照升序存储列值，但可以通过使用降序索引来切换为降序存储。**如果你的应用按照特定的顺序来获取数据，则在数据行被发送给应用之前需要进行排序。通过降序索引可以避免这个排序。如果应用按照某个特定的顺序上百万次地获取数据,则这类索引是非常有用的。例如。取自客户交易表的按照时间顺序逆序排列的客户数据。

下列代码清单中，使用`product desc`、`year desc`及`week desc`声明了3个降序排列的列，在其上添加索引`Sales_fact_c1`。`SELECT`语句声明了与索引顺序相匹配的排序顺序`product desc`、`year desc`、`week desc`来访问表。执行计划显示即使在`SELECT`语句中有`order by`子句，在语句执行的过程中并没有排序步骤。

自`Oracle`数据库`11gR2`版本开始，降序索引实现为基于函数的索引。

> 降序索引

{% code lineNumbers="true" %}
```sql
drop index sales_fact_c1;

create index sales_fact_c1 on sales_fact(product desc,year desc,week desc);

select year,week from sales_fact s 
where year in (1998,1999,2000) and week<5 and product='Xtend Memory'
order by product desc,year desc,week desc;

select index_name,index_type from dba_indexes
where index_name = 'SALES_FACT_C1';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>降序索引</p></figcaption></figure>

## 管理问题的解决方案

索引可以用来解决现实世界中的运算问题。例如，要想看一个新索引在生产环境应用中的效果，你可以使用不可见索引。你还可以使用不可见索引来安全地删除索引。

### 不可见索引

在某些场景下，你可能需要增加一个索引来对`SQL`语句的性能进行调优，但你不太确定索引所带来的负面影响。不可见索引在以较小的风险来衡量新索引所带来的影响方面非常有用。一个索引可以加入到数据库中并被标记为不可见，这样优化器就不会选用这个索引。可以在确定某个索引没有负面影响或对执行计划没有负面影响后将它标记为可见。

在数据库中加入索引以后，你可以在会话中将`optimizer_use_invisible_indexes`参数设置为`true`，这样不会影响应用性能。然后复查`SQL`语句的执行计划。在下列代码清单中，第1个`SELECT`语句在执行计划中使用索引`sales_fact_c1`。接下来的`SQL`语句将`sales_fact_c1`索引标记为不可见，从而同一个`SELECT`语句的第2个执行计划显示该索引被优化器忽略了。

> 不可见索引

{% code lineNumbers="true" %}
```sql
select * from (
    select * from sales_fact 
    where product = 'Xtend Memory' and year=1998 and week=1
) where rownum<21;

alter index sales_fact_c1 invisible;

select * from (
    select * from sales_fact 
    where product = 'Xtend Memory' and year=1998 and week=1
) where rownum<21;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption><p>索引范围扫描</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption><p>设置索引不可见后全表</p></figcaption></figure>

在下列代码清单中，执行计划显示在会话级将参数设置为`true`之后优化器选择了`Sales_fact_c1`索引。

不可见索引还有另一个应用场景。这种索引有助于在删除不使用的索引时用来降低风险。从生产数据库中删除不使用的索引并不是令人愉快的经历,可能之后却意外地认识到删除的索引在一个很重要的报表中用到了。即使在经过充分的分析之后，也有可能被删除的索引在某个商务过程中是必须的，而重建索引可能会导致应用停机。从`Oracle`数据库`11g`以来，你可以将索引标记为不可见，等过了几周以后，如果没有任何进程要用到这个索引，则可以较为安全地将其删掉。如果在被标记为不可见后发现索引是需要的,则可以很快地使用一个`SQL`语句来将索引还原为可见状态。

> Optimizer\_user\_invisible\_indexes

{% code lineNumbers="true" %}
```sql
alter session set optimizer_user_invisible_indexes = true;

select * from (
    select * from sales_fact 
    where product = 'Xtend Memory' and year=1998 and week=1
) where rownum<21;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption><p>Optimizer_user_invisible_indexes</p></figcaption></figure>

不可见索引在`DML`过程中的维护与任何其他索引类似。这个操作上的特性对于降低删除索引的风险是很有用的。

### 虚拟索引

你曾经有过增加了一个索引,但后来却意外地认识到由于数据分布或某种统计信息问题的原因这个索引不会被优化器选中的经历吗?虚拟索引对于查看索引的有效性是很有用的。虚拟索引不会分配存储空间，因此可以很快建立。虚拟索引与不可见索引的不同之处在于不可见索引是有与之相关的存储的，只是优化器不能选择它们。而虚拟索引没有与之关联的存储空间。由于这个原因，虚拟索引也被称为**无段索引**。

会话可修改的一个下划线参数`use_nosegment_indexes`控制了优化器是否可以考虑选择虚拟索引。这个参数的默认值是`false`，应用不会选择虚拟索引。你可以通过下面这种方法在不影响其他应用的基础上测试虚拟索引:创建索引，在你的会话中将这个参数设置为`true`，然后验证SQL语句的执行计划。下列代码清单中，使用`nosegment`子句创建了一个虚拟索引`Sales_virt`。在当前会话中将参数值修改为`tue`之后,检查了`SELECT`语句的执行计划。执行计划显示优化器将为`SQL`语句选择这个索引。在检查过执行计划之后，可以将这个索引删除掉并重建为常规索引。

> 虚拟索引

{% code lineNumbers="true" %}
```sql
create index slaes_virt on sales(cust_id,promo_id) nosegment;

alter session set "_use_nosegment_indexes"=true;

select * from sales where cust_id = '987' and promo_id = '999';

PLAN_TABLE_OUTPUT                                                                                                |
-----------------------------------------------------------------------------------------------------------------+
Plan hash value: 1558598240                                                                                      |
-----------------------------------------------------------------------------------------------------------------|
| Id  | Operation                          | Name       | Rows  | Bytes | Cost (%CPU)| Time     | Pstart| Pstop ||
-----------------------------------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT                   |            |    33 |   957 |     9   (0)| 00:00:01 |       |       ||
|   1 |  TABLE ACCESS BY GLOBAL INDEX ROWID| SALES      |    33 |   957 |     9   (0)| 00:00:01 | ROWID | ROWID ||
|*  2 |   INDEX RANGE SCAN                 | SLAES_VIRT |  9188 |       |     1   (0)| 00:00:01 |       |       ||
-----------------------------------------------------------------------------------------------------------------|
```
{% endcode %}

虚拟索引没有与之相关的存储，因此这些索引不需要进行维护。但你可以像常规索引那样收集这些索引的统计信息。虚拟索引可以用来改进谓语的基数估计而不增加与常规索引相关的存储成本。

### 位图联结索引

位图联结索引对于数据仓库应用中物化事实表和维度表之间的联结是很有用的。在数据仓库表中，一般来说，事实表比维度表要大得多，并且维度和事实表使用主键进行联结—在它们之间存在外键关系。这种联结的成本由于事实表很大而更高。如果能够预先存储联结结果则这些查询的性能就会得到提高。物化视图是预先计算联结结果的可选项之一，位图联结索引是另一个可选项。&#x20;

在下列代码清单中，给出了一个典型的数据仓库查询及其执行计划。在这个查询中，`Sales`表是一张事实表，其他表是维度表。`Sales`表与其他维度表通过维度表上的主键列相联结。执行计划显示在这个联结步骤中`Sales`表是引导表,与其他维度表进行联结来得出最终结果集。这就是执行计划中的4个联结运算。如果事实表很大的话这个执行计划的成本就会很高。

> 典型的数据仓库(DW)查询

{% code lineNumbers="true" %}
```sql
select sum(s.quantity_sold),sum(s.ammount_sold)
from sales s,products p,customers c,channels ch
where s.prod_id = p.prod_id and s.cust_id = c.cust_id 
    and s.channel_id = ch.channel_id and p.prodcut_name='Y box'
    and c.cust_first_name='Abigail' and ch.channel_desc='Direct_sales';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption><p>典型的数据仓库(DW)查询</p></figcaption></figure>

在下列代码清单中，建立了一个位图联结索引`Sales_bji1`来预先计算联结结果。注意索引创建语句与查询中类似的联结了`Salesman`表和维度表。在创建索引后再次执行了`SELECT`语句，该`SELECT`语句的执行计划显示首先访问位图联结索引，接下来不使用任何联结操作来访问`Sales`表。在内部，表中增加了3个新的虚拟列，并且在这3个虚拟列上创建了索引。**简单来说，位图联结索引通过虚拟列上的索引物化了结果集，从而避免了成本较高的联结操作。关于位图联结索引有一点局限性，所有维度都需要定义有经过验证的主键或唯一键约束，索引必须是局部的等。**下列代码清单中的前3个语句将约束的状态修改为`validated`以启用位图联结索引的创建。

> 位图联接索引

{% code lineNumbers="true" %}
```sql
alter table product modify primary key validate;
alter table customers primary key validate;
alter table channels modify primary key validate;

create bitmap index sales_bji1 on sales(p.prod_name,c.cust_first_name,ch.channel_desc)
from sales s,products p,customers c,channels ch
where s.prod_id = p.prod_id and s.cust_id = c.cust_id 
    and s.channel_id = ch.channel_id
LOCAL;

select sum(s.quantity_sold),sum(s.ammount_sold)
from sales s,products p,customers c,channels ch
where s.prod_id = p.prod_id and s.cust_id = c.cust_id 
    and s.channel_id = ch.channel_id and p.prodcut_name='Y box'
    and c.cust_first_name='Abigail' and ch.channel_desc='Direct_sales';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>位图联接索引</p></figcaption></figure>

依赖于一个好的数据模型,位图联结索引在数据仓库环境中是很有用的。但这种索引在`OLTP(Online Transaction Processing)在线事务处理系统`的应用中是没用的。

## 小结

对于**索引类型**和**索引列**的很好的选择对于维护应用性能是很关键的。经过各种不同类型索引知识的武装，你需要集中精力将索引类型与应用访问路径匹配起来。因为Oracle数据库提供了丰富的索引功能，最好是能够应用最优的索引类型来匹配你的应用访问模式。只有在必须的时候才加入索引也是非常重要的一点。不必要的索引不仅浪费存储空间，而且会浪费宝贵的CPU周期以及内存。更进一步地来说，过量的索引将会增加**撤销区**和**重做区**大小。

[^1]: system用户密码

[^2]: 连接服务名

[^3]: 示例数据库地址

[^4]: 示例数据库地址

[^5]: system的密码

[^6]: 对应用户sys

[^7]: 对应用户hr

[^8]: 对应用户oe

[^9]: 对应用户pm

[^10]: 对应用户ix

[^11]: 对应用户sh

[^12]: 对应用户bi

[^13]: 示例数据库地址

[^14]: 
