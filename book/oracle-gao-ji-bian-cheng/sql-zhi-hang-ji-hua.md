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

# SQL执行计划

你已经见过了很多的执行计划,但在这一章中我将更详细地阐述如何正确产生以及解读执行计划。我已经建立起你需要理解的在执行计划中所用的最常见运算的知识的基础，你需要将这些知识应用到实践中。

到本章结束的时候，我需要你能够很自信地分解(即使是最复杂的)执行计划，并理解你所写的SOL语句是如何来执行的。随着可以得到解释计划输出的开发工具的普遍使用，生成解释计划就变得相当简单。而不那么简单的就是要得到执行计划。你也许会想知道解释计划和执行计划之间的区别是什么。在本章中你将会看到，区别可能很显著。 将简述解释计划输出与实际执行计划信息之间的区别。你将学会如何将预期的执行计划与实际的计划进行比较以及如何解释它们之间的不同之处。

## 解释计划

语句`EXPLAIN PLAN`用来显示优化器为SQL语句所选择的执行计划。我需要明的第一件事情就是当你得到了解释计划输出的时候,你其实是得到了当SQL语句执行的时候应该采用的**预期执行计划**。你并没有得到实际的执行计划以及与其相关的数据源执行统计信息。你所得到的只是估计值，而不是实际值。在本章中，将通过将估计的信息称为解释计划输出而把实际信息称为执行计划输出来区分实际的和预期的执行计划。

### 使用解释计划

当使用EXPLAIN PLAN来为一个查询生成预期的执行计划时，输出将包括以下几种。

* SQL语句中所引用到的每一张表。
* 访问每张表所用的方法。
* 每一对需要联结的数据源所用的联结方法。
* 按次序列出的所有需要完成的运算。
* 计划中各步骤的谓语信息列表。
* 对于每个运算，估计出该步骤所要操作的数据行数和字节数。
* 对于每个运算，计算出成本值。
* 如果适用，所访问的分区信息。
* 如果适用，并行执行的相关信息。

使用了`EXPLAIN PLAN`命令和`SQL*PIus AUTOTRACE`命令来生成解释计划输出。使用`AUTOTRACE`可以自动生成计划，使得你所要做的事情就是打开`AUTOTRACE`(使用`TRACEONLY EXPLAIN`选项)并执行一个查询。这样计划就生成了，并且所有的输出都在一步中显示。当使用这种方法生成计划的时候，`EXPLAIN PLAN`和`TRACEONLY EXPLAIN`选项都不实际执行查询，它只产生预期的执行计划。你所使用的开发工具也应该有相应的生成解释计划的选项。

你在解释计划输出中所看到的信息是由`EXPLAIN PLAN`命令生成并默认存储在表`PLAN_TABLE`中的。`AUTOTRACE`命令从所提供的`dbms_xplan`包中调用`display`的数来自动生成输出。当使用`EXPLAIN PLAN`命令的时候你必须手工执行查询(稍后我将更详细地讨论dbms\_xplan)。下列代码清单给出了Oracle11R2中`PLAN_TABLE`表的结构，供参考。

> PLAN\_TABLE表

<figure><img src="../.gitbook/assets/image (7) (1).png" alt=""><figcaption><p>PLAN_TABLE</p></figcaption></figure>

我并不会逐个来看所列出来的每一列，但我想给出这个表的结构描述，如果你愿意的话可以从中进行进一步的学习。你可以在Oracle文档中找到更多相关信息。

`dbms_xplan.display`函数的一个非常好的特性是它可以基于每一个特定的SQL语句所生成的执行计划而自动显示适当的列。例如,如果计划中使用了分区运算,在输出中就会包含`PARTITION_START`、`PARTITION_STOP`以及`PARTITION_ID`这些列。`dbms_xplan.display`函数的自动确定所要显示列的能力相比于自己手工查询`PLAN_TABLE`的老方法是一个超级特性。

&#x20;在上面例子中查询计划所显示的列是:`ID、OPERATION、OPTIONS、OBJECT _NAME、CARDINALITY、BYTES、COST、TIME`(这个字段被包括进来了但为了节省空间隐藏起来了)`ACCESS_PREDICATES`以及`FILTE_RPREDICATES`。这些是最典型的显示列。下表给出了这些常用列的简单定义。

| 列                  | 描述                          |
| ------------------ | --------------------------- |
| ID                 | 为每一个步骤分配的唯一编                |
| OPERATION          | 这一步骤所进行的内部运算                |
| OPTIONS            | 运算列的附加说明(附于OPERATION)       |
| OBJECT\_NAME       | 表或索引的名称                     |
| CARDINALITY        | 预期的运算所要访问的行数                |
| BYTES              | 预期的运算的字节数                   |
| COST               | 由优化器确定的运算所需要的成本值            |
| TIME               | 预计进行运算所需要的以秒为计量单位的时间        |
| ACCESS\_PREDICATES | 用来在访问结构(一般为索引)中确定数据行所在位置的条件 |
| FILTER\_PREDICTES  | 用来在数据行被访问后进行筛选的条件           |

在使用`dbms_xplan.display`函数的时候没有在计划输出中显示出来的`PLAN_TABLE`中的一列就是`PARENT_ID`列。在输出中想通过直观的可视化提示来表示计划中的父子关系，而不是显示列值。我觉得将`PARENT_ID`列的值包含在计划输出中也将有助于让计划清晰明了，但如果想要这样，你必须自己重新写对PLAN\_TABLE表的查询来产生输出，从而把这一列包含进去(如果你想这样做的话)。我建立了一个简单的查询来显示每一步的PARENT\_ID，并确保在计划非常复杂使用直观的缩进较难对齐的时候保持计划的可读性。我仍然使用缩进但将它限制在每行一个空格。下列代码清单示出了使用`PARENT_ID`执行查询。

> 显示PARENT\_ID

{% code lineNumbers="true" %}
```sql
SELECT ID,PARENT_ID,
    lpad(' ',level)||operation||' '||options||' '||
    object_name as operation
from plan_table
start with id = 0
connect by prior id = parent_id;
```
{% endcode %}

PARENTID对你是很有帮助的，因为如果你将计划中所包含的父子关系铭记于心的话将最容易读懂计划中的运算。计划中的每一步都将会有0\~2个子步骤。如果你把计划分解为按父-子关系分组的小块，你将更容易读懂和理解计划。

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

在示例计划中，每个运算有0个、1个或2个子运算。例如，全表扫描运算没有任何子运算。看一下ID=8的那一行。另一个运算没有子运算的例子就是第6行。如果你从上到下浏览一遍PARENTID列，你会发现其中没有步骤6和步骤8。这就意味着这两个运算的完成不依赖于其他任何运算步骤。但是，这两个步骤都是其他步骤的子运算，并会将它们所访问的数据传递给它们的父步骤。当一个运算没有子运算的时候，所展示出来的估计行数`(PLAN_TABLE`表中的`CARDINALITY`列)表示该运算一次迭代所获取的行数。这在一个运算向其选代父运算提供行的时候有点容易引起混乱。例如，第9步是一个估计只有1行的索引唯一扫描运算，但这个估计并没有表明该步骤所要访问的总数据行数。总数取决于父运算。我稍后将会更详细地讨论这方面的内容。 步骤6和步骤8的父步骤5和步骤7都是有一个子运算的例子。

总的来说，只有一个子运算的运算可以分为以下3类。&#x20;

* **加工运算** 从子运算接收一个数据行集并经过加工以后传递给其父运算。
* **传递运算** 只是起传递的作用而不对来自子运算的数据做任何修改或加工。它们基本上是用来确定某个运算的特性。VIEW运算就是传递运算的一个很好的例子。
* **迭代运算** 表示子运算要多次执行。你通常会在这类运算的名字上看到`ITERATOR、INLIST或ALL`等字眼。&#x20;

步骤5和步骤7都是加工运算。它们从子运算(步骤6和步骤8)中得到数据集然后做一些加工在步骤5中，索引全扫描中所返回的行编号被用来获取`DEPARTMENT`表的数据块。在步骤7中，对LOCATIONS表进行全扫描所返回的行按照联结列进行了排序。

最后，一个具有两个子运算的运算或者是迭代的或者是连续的。当父运算是迭代的时候，就需要访问子数据源集以便数据源A和B中的每一行都被访问。对于父运算紧接着子运算来进行的类型，第一个子数据源被访问接下来第二个子数据源也被访问。联结类型如嵌套循环和笛卡儿合并联结是迭代的，`FILTER运算`也是迭代的。其他有两个子运算的运算都是对它们的子数据集连续进行加工的。

复习这些内容的原因就是强调要学会采用“各个击破”的方法来阅读和理解计划输出的重要性。一个计划看上去越大越复杂，通常其关键问题所在就越难找到。如果你学会了在计划输出中寻找父子关系并将精力集中在计划的更小的小块上，你会发现处理你所见到的一切会更简单。

### 理解解释计划

解释计划输出最令人沮丧的地方就是它与语句实际执行时所使用的计划可能是不一致的。使用解释计划的时候有以下3点可能导致计划输出与实际执行计划不一致的地方，请谨记。

* 解释计划是基于你使用它的时候的环境来生成的。&#x20;
* 解释计划不考虑绑定变量的数据类型(所有的绑定变量都是`VARCHAR2`的)。&#x20;
* 解释计划不“窥视”绑定变量的值。

基于这些原因，解释计划很有可能就会生成与语句实际执行不一致的计划。下列代码清单证明了绑定变量数据类型的第二点。

> 解释计划于绑定变量数据类型

{% code lineNumbers="true" %}
```plsql
create table regions
(
    region_id varchar2(10) primary key,
    region_name varchar2(25)
);

variable regid number
exec :regid := 1;

select * from regions where region_id = :regid;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

你注意到解释计划输出中是如何表明将使用主键索引而实际执行计划使用全表扫描的了吗?其原因在谓语信息那一部分很清楚地表明了。在解释计划输出中，谓语是`“REGION_ID”=:REGID`，而在实际执行计划中所示出的谓语是`TO_NUMBER("REGION_ID")=:REGID`。这说明了解释计划不考虑绑定变量的数据类型并假设所有的绑定变量都是字符串类型的方式。对于解释计划来说，数据类型被认为都是一样的(都是字符串)。然而，当语句真正执行的时候所准备的执行计划却要考虑数据类型,Oracle隐式地将字符串数据类型的REGION ID列转换为数值类型来匹配绑定变量的数据类型(数值型)。这是可以预见的行为，因为当进行比较的两种数据类型不匹配时Oracle总是尝试将字符串类型转换为与之匹配的非字符串类型。在这个例子中通过这样做，`TO_NUMBER函数`使得不允许使用索引。这是需要牢记于心的另一个预期的行为:谓语必须严格匹配索引定义，否则将不会使用索引。

如果你在开发环境中测试这个语句并使用解释计划输出来确定使用了索引,你就错了。从解释计划输出中，你可能会看到如你所期望的，计划使用了索引，但是当该语句实际执行的时候，性能可能就不是特别让人满意，因为实际使用的是全表扫描。

使用解释计划输出作为测试的唯一来源导致的另一个问题是你可能永远也得不到语句是如何来使用资源的真实情况。预期永远只能是预期。要真正确定SQL语句的行为并准确判断语句是否已经提供了最优的性能，你需要去查看实际执行统计信息。稍后我将详细讨论如何抓取并理解实际执行统计信息。

### 阅读计划

在更深入地讨论抓取实际执行计划数据之前，我想要确认你能够轻松阅读计划。我已经讨论过了PARENT\_ID列对于你将很长很复杂的计划分解为较短的更易管理的分块更易于你理解的重要性。将一个计划分解为小块将有助于你来阅读它，但你需要知道如何从头到尾阅读整个计划。

有3种途径有助于你阅读和理解所有计划:(1)学会识别和分割父子组;(2)掌握计划中运算执行的顺序;以及(3)学会以叙述的形式来阅读计划。我已经学会了做这3件事，因此当我看一个计划的时候，就能够很自如地浏览并很快找出潜在问题所在的区域。最开始的时候可能会比较慢，让你有点沮丧，但假以时日并勤加练习，就将形成一种第二本能。

让我们首先从执行的顺序开始。计划是按照运算的顺序ID号来显示的。但是，每个运算执行的顺序并不是严格按照自上而下来进行的。通过运算的缩进这一视觉线索，你可以很快地浏览整个计划并寻找缩进最多的运算。缩进最多的运算实际上是执行过程中首先进行的运算。如果在同一层次上有多个运算，则按照自上而下的顺序来依次执行。

> 解释计划例子

<figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption><p>解释计划例子</p></figcaption></figure>

乍一看，你可以看到第6行和第8行缩进是最深的。第6行将首先执行并将索引全扫描所得到的行编号传递给它的父步骤(第5行)。接下来将执行第8行并将其行数据源传递给它的父步(第7行)。这些步骤将从缩进最深的到缩进最少的一步接一步来执行并将结果传递给其父步骤，直到所有步骤都执行完毕。为了更清楚地看出执行的步骤，下列代码清单来读取PLAN\_TABLE表并将输出按照执行的顺序来列出。

> 按照执行顺序显示的计划运算

{% code lineNumbers="true" %}
```plsql
select id,parent_id,operation
from (
    SELECT level lvl,ID,PARENT_ID,
        lpad(' ',level)||operation||' '||options||' '||
        object_name as operation
    from plan_table
    start with id = 0
    connect by prior id = parent_id;
)
order by lvl desc,id;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

解释计划输出中最有用的部分之一就是被称为谓语信息的部分。在这个部分中，将会示出`ACCESS_PREDICATES`和`FILTER_PREDICATES列。`这两列与计划运算列表中的一行(用ID列来指示)相关。你会发现计划中每一个有相关的访问或选谓语的运算，在其ID的旁边都有一个星号`(*)`。当你看到星号的时候，你就知道要在谓语信息部分寻找ID号来确定哪个谓语`(WHERE子句中的条件)`是与该运算相关的。通过使用这些信息你就可以确认用来进行索引访问的列是正确(或不正确)的，并且可以确定在哪里进行了条件的过滤。

较晚地进行过滤是常见的性能抑制剂。例如，如果你想把100块石头中的一堆从前院搬到后院，但你只想要重量约在2.3\~4.5千克之间的石头。你会先把所有100块石头都搬到后院然后再把那些你需要的移走，还是只搬那些满足你重量要求的石头呢?一般来说，你会只搬那些你需要的石头，对吗?

使用筛选型谓语信息可以帮助你验证不需要的行在计划中已经尽可能早地从你的结果集中过滤掉了。就好像你把所有石头一起搬到后院没什么意义一样，把最终不会包含在结果集中的很多行放在计划运算的行集中也没什么意义。你可以使用过滤信息来验证每个条件都尽可能早地在计划中得到了应用。如果一个过滤条件进行得太晚，你可以调整SQL语句或者采取其他步骤(例如确认统计信息是最新的)来确保你的计划没有比实际所需运行得更困难。

最后，学会把计划当做一段文字描述来进行阅读会非常有帮助。对于很多人来说，将一系列的计划运算转化为一段文字描述能够比其他方法更有助于理解计划执行。让我们来将例子中的计划转化为文字描述并看看这样是否使得计划的阅读和理解变得更简单了。下面这一段就是对于计划例子的文字描述样例。

为了生成这个SELECT语句的结果集，`DEPARTMENTS`表中的数据行将会通过对`DEPARTMENTS.LOCATION_ID`列进行索引全扫描来访问。通过对`LOCATIONS`表使用进行全扫描，将会取出按`LOCATION_ID`进行排序的数据行。然后将这两个数据行进行合并生成联结后的包含`DEPARTMENTS`和`LOCATIONS`表中相匹配数据行的数据集。这个数据行集,我将其称为`DEPT_LOC`,将会再与`COUNTRIES`表进行联结,并会选代取出`DEPT_LOC`数据集中的每一行来在`COUNTRIES`表中寻找与`COUNTRY_ID`相匹配的行。这样得到的数据集，我将其称为`DEPT_LOC_CTRY`。现在其中包含`DEPARTMENTS`、`LOCATIONS`和`COUNTRIES`表中的数据并将会散列化到内存中并与`REGIONS`表通过`REGION_ID`列来进行联结匹配。这个结果集`DEPT_LOC_CTRY_REG`，又将会被散列化到内存中并通过`DEPARTMENT_ID`列来与`EMPLOYEES`表进行匹配以生成最终的数据行的结果集。

要想生成这段描述性文字,我只要按照执行的顺序来简单看一下每一个步骤并写下这些步骤以及各步骤之间是如何联系(联结)的文字描述。我依次处理每一对父子运算集合直到所有的步骤都干完。你会发现写下计划的文字描述有助于你更清晰地领会整个计划。对于更复杂的计划，你会发现找出整个计划中的几个核心部分并写出这些部分的文字描述将会有助于你更好地理解运算的流程。其中的关键就是使用文字描述来更好地理解计划。如果你发现这样做起来较困难，那么就保持计划原有的样子不变吧。但是，花点时间学会将计划转变为文字描述形式是项很有用的技能，因为它可以帮助你以一种甚至不需要任何人看实际的计划输出的方式描述你的查询是如何来执行的。

## 执行计划

当一条SQL语句执行的时候将会生成该语句的实际执行计划。在语句被硬解析之后，所选的执行计划就会被存到库高速缓存中以便以后重用。可以通过查询`V$SOL_PLAN`来查看计划运算。

`V$SQL_PLAN`的定义与`PLAN_TABLE`的基本相同，其不同点是`V$SOL_PLAN`包括了一些含有如何在库高速缓存中定位并找出当前所执行语句的列。这些附加的列是:`ADDRESS、HASH_VALUE、SQL_ID、PLAN_HASH_VALUE、CHILD_ADDRESS以及CHILD_NUMBER`。你可以使用这些值中的一个或多个来找到库高速缓存中的任何SQL语句。

### 查看最近生成的SQL语句

下列代码清单给出了一个对`V$SQL`查询最近SC0TT用户执行的SQL语句的例子,并给出了每一列的标识信息值。

> 获取最近执行的SQL语句的V$SQL查询

{% code lineNumbers="true" %}
```sql
SELECT SQL_ID,CHILD_NUMBER,HASH_VALUE,ADDRESS,EXECUTIONS,SQL_TEXT
FROM V$SQL
WHERE PARSING_USER_ID = (
    SELECT USER_ID FROM ALL_USERS
    WHERE USERNAME = 'SCOTT'
)
AND COMMAND_TYPE IN (2,3,5,7,189)
AND UPPER(SQL_TEXT) NOT LIKE UPPER ('%RECENTSQL%')
```
{% endcode %}

在以SC0TT用户连接上以后，执行查询。然后，当针对`V$S0L`执行查询的时候，你可以看到它们现在被载入到了库高速缓存中,并且每一个都有与之相联结的标识。`SOL_ID`和`CHILD_NUMBER`列包含了你最常用的获取语句执行计划和执行统计信息的标识信息。

### 查看相关执行计划

有好几种方法可以用来查看任何之前已经执行过的SQL语句保存在库高速缓存中的执行计划。最简单的方法就是使用`dbms_xplan.display_cursor`函数。下列代码清单示出了如何使用`dbms_xplan.display_cursor`来显示最近执行的SQL语句的执行计划。

> 使用dbms\_xplan.display\_cursor函数

{% code lineNumbers="true" %}
```sql
select /*+ gather_plan_statistics */ empno,ename 
from scott.emp
where ename = 'KING';

set serveroutput of
select * from table(dbms_xplan.display_cursor(null,null,'ALLSTATS LAST'));
```
{% endcode %}

首先，注意在查询中使用的`gather_plan statistics`提示。为了为计划抓取行数据源执行统计信息，你必须告诉Oracle在语句执行的时候来收集这些信息。行数据源的执行统计信息包括行数、一致性读取次数、物理读取次数、物理写入次数，以及每一个运算在一行数据上的运行时间。可以使用这个提示来一句一句地收集这些信息，或者你可以将`STATISTICS_LEVEL`实例参数的值设置为`ALL`。抓取这些统计信息确实增加了语句执行的成本，因此你并不一定要总是把这个功能“打开”。提示允许你在需要的时候使用它—仅用在你所选择的那条语句上。这个提示的使用会收集相关信息并在Starts、A-Rows、A-Time以及Bufers列中显示出来。

### 收集执行计划统计信息

当无法获取计划统计信息的时候所给出的计划运算与解释计划的输出本质上是一样的。要想准确地知道计划的效果如何，你需要计划的行数据源执行统计信息。这些值可以告诉你计划中的每一个运算实际上发生了什么。该数据是从`V$SQL_PLAN_STATISTICS`视图中取出的。这个视图将计划的每一个运算行与一行统计数据联系起来。一个名为`V$SOL_PLAN_STATISTICS_ALL`的复合视图包括了`V$SQL_PLAN`中的所有列加上`V$SQL_PLAN_STATISTICS`中的列以及一些包含内存使用信息的附加列。下列代码清单描述了`V$SOL_PLAN_STATISTICS_ALL`视图中的列。

> V$SQL\_PLAN\_STATISTICS\_ALL视图描述

<figure><img src="../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

包含与涉及`dbms_xplan.display_cursor`函数输出相关的统计信息的列均以前缀`LAST`开头。当你使用`ALLSTATS LAST`格式选项的时候，计划就会为其中的每一行显示这些列的值。因此，对于每一个运算，你将能够准确地知道将会返回多少行(`LAST_OUTPUT_ROWS`在`A-Rows`列中给出)，发生了多少次一致性读取(`LAST_CR_BUFFER_GETS`在`Buffers`列中给出)，发生了多少次物理读取(`LAST_DISK_READS`在`Reads`列中给出),以及每一步骤执行的次数(`LAST_STARTS`在`Starts`列中给出)。根据所执行的运算不同，还将显示其他一些列，但上面列出的这些是最常见的。

`dbms_xplan.display_cursor`的调用签名如下。

<figure><img src="../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

在下列代码清单的例子中，所使用的3个参数是`SOLID=>null、CURSOR_CHILD_NO=>null`和`FORMAT=>ALLSTATS LAST`。`SOL_ID`和`CURSOR_CHILD_NO`参数使用空值表明了需要取出上一个执行语句的执行计划。

{% hint style="info" %}
你可能已经注意到了，我在调用`dbms_xplan.display_cursor`函数之前执行了SQL\*Plus命令`SET SERVEROUTPUT OFF`。当你执行一个语句并打开`SERVEROUTPUT`，都会隐式地调用`dbms_output`。如果你没有将`SERVEROUTPUT`关闭，那么最后执行的一条语句将会是这个`dbms_output`的调用。将前两个参数设置为空值将不能给你你所执行的SQL语句，而是会试图给你`dbms_output`调用所使用的计划。简单地将这个设置关掉将会停止隐式调用，并确保你能够得到最近执行的SOL语句的计划。
{% endhint %}

### 取回标识SQL语句的计划

如果想取出之前执行过的一句语句，你可以从`V$SQL`中取出`SOL_ID`和`CHILD_NUMBER`。为了简化寻找正确语句标识的过程，尤其是在我进行测试的时候，我在所执行的每一句语句上都加了一个唯一的注释来进行标识。从而，无论什么时候想从库高速缓存中取出计划，我所需要做的事情就是查询`V$SQL`来定位包含我所使用注释的语句文本。下列代码清单给出了一个这样的例子，并且给出了后来我用来找到想要的语句所用的查询。

{% code lineNumbers="true" %}
```sql
select /*+ gather_plan_statistics*/ /* KM-EMPTEST1 */
   empno,ename from emp
where job = 'MANAGER';

select sql_id,child_number,sql_text from v$sql 
where sql_text like '%KM-EMPTEST1%';

select * from table(dbms_xplan.display_cursor('59mn6y61zkzta',0,'ALLSTATS LAST'));

PLAN_TABLE_OUTPUT                                                                   |
------------------------------------------------------------------------------------+
SQL_ID  59mn6y61zkzta, child number 0                                               |
-------------------------------------                                               |
select /*+ gather_plan_statistics*/ /* KM-EMPTEST1 */ empno,ename from              |
emp where job = 'MANAGER'                                                           |
Plan hash value: 3956160932                                                         |
------------------------------------------------------------------------------------|
| Id  | Operation         | Name | Starts | E-Rows | A-Rows |   A-Time   | Buffers ||
------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT  |      |      1 |        |      3 |00:00:00.01 |       7 ||
|*  1 |  TABLE ACCESS FULL| EMP  |      1 |      3 |      3 |00:00:00.01 |       7 ||
------------------------------------------------------------------------------------|
Predicate Information (identified by operation id):                                 |
---------------------------------------------------                                 |
   1 - filter("JOB"='MANAGER')                                                      |
```
{% endcode %}

你会注意到当我查询`V$S0L`的时候，显示了两条语句。一条是我为了寻找`V$SOL`中的条目所执行的SELECT查询语句，另一条是我实际执行的查询。虽然这些步骤可以得到我想要的语句，但我发现如果把整个步骤放到一个脚本中来自动执行会更简单。在该脚本中，我通过将我对V$SOL所执行的用来寻找最近执行语句的那一条查询过滤掉来直接找出我想要的查询语句,并使用我的标识性注释来确保找到的是最近执行的那一条语句。下列代码清单给出了我实际使用的脚本。

{% code lineNumbers="true" %}
```sql
select /*+ gather_plan_statistics*/ /* KM-EMPTEST1 */
   empno,ename from emp
where job = 'MANAGER';

SELECT XPLAN.*
FROM (
   SELECT 
      MAX(SQL_ID) KEEP (DENSE_RANK LAST ORDER BY LAST_ACTIVE_TIME) SQL_ID,
      MAX(CHILD_NUMBER) KEEP (DENSE_RANK LAST ORDER BY LAST_ACTIVE_TIME) CHILD_NUMBER
   FROM V$SQL
   WHERE UPPER(SQL_TEXT) LIKE '%'||'KM-EMPTEST1'||'%'
      AND UPPER(SQL_TEXT) NOT LIKE '%FROM V$SQL WHERE UPPER(SQL_TEXT) LIKE %' 
) SQLINFO,
TABLE(DBMS_XPLAN.DISPLAY_CURSOR(SQLINFO.SQL_ID,SQLINFO.CHILD_NUMBER,'ALLSTATS LAST')) XPLAN;
```
{% endcode %}

这个脚本将会返回与你输入的模式相匹配的最后一条执行的SQL语句的执行计划。正如我所提过的，如果你使用了注释来进行标注，要查找这个语句就会更简单，但它将查找所有字符串来找出与你所输入的文本相匹配的文本。然而，如果有多个语句都有相匹配的文本，这个脚本只会显示出与之相匹配的最近一条执行过的语句。如果你需要另外一条语句，你就必须发起一个针对`V$S0L`的查询，然后将正确的`SQL_ID`和`CHILD_NUMBER`提供给`dbms_xplan.display_cursor`调用。

### 深入理解DBMS\_XPLAN的细节

OracIe提供`DBMS_XPLAN`包并且可以用来简化执行计划输出的获取和显示。为了更全面地使用该包中的所有步骤和功能，你需要对一些固定的视图具有优先权。对`SELECT_CATALOG_ROLE`的单一授权就可以确保你具有访问所有你需要的对象的权限。但最低限度来说，为了恰当地执行`display`和`display_cursor`的数，你至少应该对`V$SOL`、`V$SOL_PLAN`、`V$SESSION`以及`V$SOL_PLAN_STATISTICS_ALL`具有优先选择权。在本节中,我将会多讲述关于这个包的使用的细节, 特别是关于`display`和`display_cursor`函数的格式选项。

自从在Oracle 9版本中第一次出现以来，`dbms_xplan`包得到了不断发展。最初的时候，只含有`display`函数。在`Oracle11g Release2`中，这个包包含了21个函数，尽管在文档中只有6个。这些函数不仅可以用来展示解释计划输出，而且可以用来输出存储在自动工作负载信息库(`Automatic Workload Repository，AWR`)、SQL调试集、缓存SQL游标以及SQL计划基线中的语句计划。5个主要的用来在上述领域显示计划的表函数分别如下 。

* DISPLAY&#x20;
* DISPLAY\_CURSOR
* DISPLAY\_AWR&#x20;
* DISPLAY\_SOLSET
* DISPLAY\_SOL\_PLAN\_BASELINE&#x20;

这5个表的数都返回`DBMS_XPLAN_TYPE_TABLE`类型，由一个300字节的字符串组成。这个类型包含了每一个表函数用来动态显示计划表中的列所需的不同格式。这些函数是表函数这样一个事实意味着当在SELECT语句中使用这些函数的时候，你必须用TABLE函数来将返回类型转换为正确的类型。一个表函数就是一个存储起来的，行为与通常对表的查询类似的PLSQL函数。其好处在于你可以在函数中写代码来在数据返回给最终结果集之前对其进行数据转换。在对`PLAN_TABLE`或`V$SQL_PLAN`进行查询的时候，表函数的使用可以实现只输出某个给定的SQL语句相关的列所需的所有动态格式，而不必努力创建多个查询来处理不同的需求。

这些表函数中的每一个都可以接受`F0RMAT`参数作为输入。`FORMAT`参数控制着哪些信息被包括在显示输出中。下面列出的是文档中这些参数的值。

* BASIC只显示运算名称和选项。
* TYPICAL显示相关信息以及在适当的情况下可能会显示选项如分区和并发使用。这是默认值。
* SERIAL与TYPICAL相同但总是排除并发信息。
* ALL在输出中显示最多的信息。

除了基本的格式参数值以外，还有一些附加的细化选项可以用来定制基值的默认行为。你可以通过逗号或空格分隔来声明多个关键字，并使用加号标记(+)表示包含或使用减号标记(-)表示排除某个特定的显示元素。所有这些选项都仅显示相关的信息。下面是一些可选关键字。

* ADVANCED显示与ALL相同的信息再加上大纲部分和窥视的绑定值部分。
* ALIAS显示查询块名称/对象别名部分。
* ALL显示查询块名称/对象别名部分，谓语部分，以及列投影部分。
* ALLSTATS\*与IOSTATS LAST等价。
* BYTES显示估计的字节数。
* COST显示优化器所计算出的成本信,息。&#x20;
* I0STATS\*显示游标执行的IO统计信息。
* LAST\*显示最后执行的游标执行计划统计信息(默认为ALL并且是可累积的)。
* MEMSTATS\*为内存密集运算如散列联结、排序，或一些类型的位图运算显示内存管理统 信息。&#x20;
* NOTE显示注释部分。
* 0UTLINE显示大纲部分(将会重新生成计划的一系列提示) 。
* PARALLEL显示并行执行信息。
* PARTITION显示分区裁剪信息。&#x20;
* PEEKED\_BINDS显示绑定变量值。
* PREDICATE显示谓语部分。
* PROJECTION显示列投影部分(每一行中的哪些列被传递给其父列以及这些列的大小) 。
* REMOTE显示分布式查询信息。

后面有一个星号的关键字不能在`DISPLAY`函数中使用，因为它们需要使用只有在一个语句已经执行了的情况下才会在`V$SQL_PLAN_STATISTICS_ALL`中存在的信息。下列代码清单给出了一些使用上述各种选项的例子。

> 显示使用FORMAT参数的选项

{% code lineNumbers="true" %}
```sql
explain plan for 
select * from emp e,dept d
where e.deptno = d.deptno
    and e.ename = 'JONES';
    
select * from table(dbms_xplan.display(format=>'ALL'));

PLAN_TABLE_OUTPUT                                                                       |
----------------------------------------------------------------------------------------+
Plan hash value: 3625962092                                                             |
----------------------------------------------------------------------------------------|
| Id  | Operation                    | Name    | Rows  | Bytes | Cost (%CPU)| Time     ||
----------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT             |         |     1 |    58 |     4   (0)| 00:00:01 ||
|   1 |  NESTED LOOPS                |         |       |       |            |          ||
|   2 |   NESTED LOOPS               |         |     1 |    58 |     4   (0)| 00:00:01 ||
|*  3 |    TABLE ACCESS FULL         | EMP     |     1 |    38 |     3   (0)| 00:00:01 ||
|*  4 |    INDEX UNIQUE SCAN         | PK_DEPT |     1 |       |     0   (0)| 00:00:01 ||
|   5 |   TABLE ACCESS BY INDEX ROWID| DEPT    |     1 |    20 |     1   (0)| 00:00:01 ||
----------------------------------------------------------------------------------------|
Query Block Name / Object Alias (identified by operation id):                           |
-------------------------------------------------------------                           |
   1 - SEL$1                                                                            |
   3 - SEL$1 / E@SEL$1                                                                  |
   4 - SEL$1 / D@SEL$1                                                                  |
   5 - SEL$1 / D@SEL$1                                                                  |
Predicate Information (identified by operation id):                                     |
---------------------------------------------------                                     |
   3 - filter("E"."ENAME"='JONES')                                                      |
   4 - access("E"."DEPTNO"="D"."DEPTNO")                                                |
Column Projection Information (identified by operation id):                             |
-----------------------------------------------------------                             |
   1 - (#keys=0) "E"."EMPNO"[NUMBER,22], "E"."ENAME"[VARCHAR2,10],                      |
       "E"."JOB"[VARCHAR2,9], "E"."MGR"[NUMBER,22], "E"."HIREDATE"[DATE,7],             |
       "E"."SAL"[NUMBER,22], "E"."COMM"[NUMBER,22], "E"."DEPTNO"[NUMBER,22],            |
       "D"."DEPTNO"[NUMBER,22], "D"."DNAME"[VARCHAR2,14], "D"."LOC"[VARCHAR2,13]        |
   2 - (#keys=0) "E"."EMPNO"[NUMBER,22], "E"."ENAME"[VARCHAR2,10],                      |
       "E"."JOB"[VARCHAR2,9], "E"."MGR"[NUMBER,22], "E"."HIREDATE"[DATE,7],             |
       "E"."SAL"[NUMBER,22], "E"."COMM"[NUMBER,22], "E"."DEPTNO"[NUMBER,22],            |
       "D".ROWID[ROWID,10], "D"."DEPTNO"[NUMBER,22]                                     |
   3 - "E"."EMPNO"[NUMBER,22], "E"."ENAME"[VARCHAR2,10], "E"."JOB"[VARCHAR2,9],         |
       "E"."MGR"[NUMBER,22], "E"."HIREDATE"[DATE,7], "E"."SAL"[NUMBER,22],              |
       "E"."COMM"[NUMBER,22], "E"."DEPTNO"[NUMBER,22]                                   |
   4 - "D".ROWID[ROWID,10], "D"."DEPTNO"[NUMBER,22]                                     |
   5 - "D"."DNAME"[VARCHAR2,14], "D"."LOC"[VARCHAR2,13]                                 |
```
{% endcode %}

<pre class="language-sql" data-line-numbers><code class="lang-sql">select /*+ gather_plan_statistics */ /*KM-EMPTEST2*/ * from emp e,dept d
where e.deptno = d.deptno
    and e.ename = 'JONES';
<strong>
</strong><strong>SELECT XPLAN.*
</strong>FROM (
   SELECT 
      MAX(SQL_ID) KEEP (DENSE_RANK LAST ORDER BY LAST_ACTIVE_TIME) SQL_ID,
      MAX(CHILD_NUMBER) KEEP (DENSE_RANK LAST ORDER BY LAST_ACTIVE_TIME) CHILD_NUMBER
   FROM V$SQL
   WHERE UPPER(SQL_TEXT) LIKE '%'||'KM-EMPTEST2'||'%'
   AND UPPER(SQL_TEXT) NOT LIKE '%FROM V$SQL WHERE UPPER(SQL_TEXT) LIKE %' 
) SQLINFO,
TABLE(DBMS_XPLAN.DISPLAY_CURSOR(SQLINFO.SQL_ID,SQLINFO.CHILD_NUMBER,'ALLSTATS LAST -COST -BYTES')) XPLAN;

PLAN_TABLE_OUTPUT                                                                                 |
--------------------------------------------------------------------------------------------------+
SQL_ID  7x1cds3a0uvhz, child number 0                                                             |
-------------------------------------                                                             |
select /*+ gather_plan_statistics */ /*KM-EMPTEST2*/ * from emp e,dept                            |
d where e.deptno = d.deptno     and e.ename = 'JONES'                                             |
Plan hash value: 3625962092                                                                       |
--------------------------------------------------------------------------------------------------|
| Id  | Operation                    | Name    | Starts | E-Rows | A-Rows |   A-Time   | Buffers ||
--------------------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT             |         |      1 |        |      1 |00:00:00.01 |       9 ||
|   1 |  NESTED LOOPS                |         |      1 |        |      1 |00:00:00.01 |       9 ||
|   2 |   NESTED LOOPS               |         |      1 |      1 |      1 |00:00:00.01 |       8 ||
|*  3 |    TABLE ACCESS FULL         | EMP     |      1 |      1 |      1 |00:00:00.01 |       7 ||
|*  4 |    INDEX UNIQUE SCAN         | PK_DEPT |      1 |      1 |      1 |00:00:00.01 |       1 ||
|   5 |   TABLE ACCESS BY INDEX ROWID| DEPT    |      1 |      1 |      1 |00:00:00.01 |       1 ||
--------------------------------------------------------------------------------------------------|
Predicate Information (identified by operation id):                                               |
---------------------------------------------------                                               |
   3 - filter("E"."ENAME"='JONES')                                                                |
   4 - access("E"."DEPTNO"="D"."DEPTNO")                                                          |
</code></pre>

<pre class="language-sql" data-line-numbers><code class="lang-sql">select /*+ parallel(d,4) parallel(e,4)*/ /*+ gather_plan_statistics */  /*KM-EMPTEST3*/ 
<strong>    d.dname,avg(e.sal),max(e.sal)
</strong>from dept d,emp e
where e.deptno = d.deptno
group by d.dname
order by max(e.sal),avg(e.sal) desc;
<strong>
</strong><strong>SELECT XPLAN.*
</strong>FROM (
    SELECT 
        MAX(SQL_ID) KEEP (DENSE_RANK LAST ORDER BY LAST_ACTIVE_TIME) SQL_ID,
        MAX(CHILD_NUMBER) KEEP (DENSE_RANK LAST ORDER BY LAST_ACTIVE_TIME) CHILD_NUMBER
    FROM V$SQL
    WHERE UPPER(SQL_TEXT) LIKE '%'||'KM-EMPTEST3'||'%'
    AND UPPER(SQL_TEXT) NOT LIKE '%FROM V$SQL WHERE UPPER(SQL_TEXT) LIKE %' 
) SQLINFO,
TABLE(DBMS_XPLAN.DISPLAY_CURSOR(SQLINFO.SQL_ID,SQLINFO.CHILD_NUMBER,'TYPICAL LAST -COST -BYTES')) XPLAN;

PLAN_TABLE_OUTPUT                                                                                 |
--------------------------------------------------------------------------------------------------+
SQL_ID  0tm3hrrjqscnz, child number 0                                                             |
-------------------------------------                                                             |
select /*+ parallel(d,4) parallel(e,4)*/ /*+ gather_plan_statistics */                            |
/*KM-EMPTEST3*/  d.dname,avg(e.sal),max(e.sal) from dept d,emp e                                  |
where e.deptno = d.deptno group by d.dname order by                                               |
max(e.sal),avg(e.sal) desc                                                                        |
Plan hash value: 3078011448                                                                       |
--------------------------------------------------------------------------------------------------|
| Id  | Operation                     | Name     | Rows  | Time     |    TQ  |IN-OUT| PQ Distrib ||
--------------------------------------------------------------------------------------------------|
|   0 | SELECT STATEMENT              |          |       |          |        |      |            ||
|   1 |  PX COORDINATOR               |          |       |          |        |      |            ||
|   2 |   PX SEND QC (ORDER)          | :TQ10004 |     4 | 00:00:01 |  Q1,04 | P->S | QC (ORDER) ||
|   3 |    SORT ORDER BY              |          |     4 | 00:00:01 |  Q1,04 | PCWP |            ||
|   4 |     PX RECEIVE                |          |     4 | 00:00:01 |  Q1,04 | PCWP |            ||
|   5 |      PX SEND RANGE            | :TQ10003 |     4 | 00:00:01 |  Q1,03 | P->P | RANGE      ||
|   6 |       HASH GROUP BY           |          |     4 | 00:00:01 |  Q1,03 | PCWP |            ||
|   7 |        PX RECEIVE             |          |    14 | 00:00:01 |  Q1,03 | PCWP |            ||
|   8 |         PX SEND HASH          | :TQ10002 |    14 | 00:00:01 |  Q1,02 | P->P | HASH       ||
|*  9 |          HASH JOIN BUFFERED   |          |    14 | 00:00:01 |  Q1,02 | PCWP |            ||
|  10 |           PX RECEIVE          |          |     4 | 00:00:01 |  Q1,02 | PCWP |            ||
|  11 |            PX SEND HASH       | :TQ10000 |     4 | 00:00:01 |  Q1,00 | P->P | HASH       ||
|  12 |             PX BLOCK ITERATOR |          |     4 | 00:00:01 |  Q1,00 | PCWC |            ||
|* 13 |              TABLE ACCESS FULL| DEPT     |     4 | 00:00:01 |  Q1,00 | PCWP |            ||
|  14 |           PX RECEIVE          |          |    14 | 00:00:01 |  Q1,02 | PCWP |            ||
|  15 |            PX SEND HASH       | :TQ10001 |    14 | 00:00:01 |  Q1,01 | P->P | HASH       ||
|  16 |             PX BLOCK ITERATOR |          |    14 | 00:00:01 |  Q1,01 | PCWC |            ||
|* 17 |              TABLE ACCESS FULL| EMP      |    14 | 00:00:01 |  Q1,01 | PCWP |            ||
--------------------------------------------------------------------------------------------------|
Predicate Information (identified by operation id):                                               |
---------------------------------------------------                                               |
   9 - access("E"."DEPTNO"="D"."DEPTNO")                                                          |
  13 - access(:Z>=:Z AND :Z&#x3C;=:Z)                                                                  |
  17 - access(:Z>=:Z AND :Z&#x3C;=:Z)                                                                  |
</code></pre>

### 使用计划信息来解决问题

既然你已经知道了如何来访问各种信息，那么要如何来使用它们呢?计划信息，尤其是计划统计信息，能够帮助你确定计划是如何来执行的。你可以利用这些信息来确认是否还存在任何易出故障的问题点，从而你可以调整SQL语句的写法，增加或修改索引，或者甚至是使用这些数据来支持需要进行统计信息更新或调整实例参数设置。例如，如果缺少了一个索引或某个索引是次优的，你可以从计划中看出来。

{% code lineNumbers="true" %}
```sql
select 
    COLUMN_NAME,NUM_DISTINCT ,DENSITY  
from ALL_TAB_COLS
where table_name = 'T1';

EXEC DBMS_STATS.GATHER_TABLE_STATS(USER, 'T1',
    estimate_percent => 100,
    cascade => TRUE,
    method_opt => 'FOR ALL COLUMNS SIZE AUTO');
```
{% endcode %}

你需要使用一个`method_opt`参数来收集直方图信息。这样，你就会重新收集一次统计信息，这一次使用`method_opt=>'FOR ALL COLUMNS SIZE AUTO'`。这个设置使得Oracle可以正确地收集objecttype列上的直方图信息。现在当你执行这个查询的时候，就得到了正确的估计数从而选择了全表扫描计划。在这个例子中，全表扫描运算是最好的选择，因为查询返回表中所有行的近80%，全表扫描访问的数据块数比索引扫描计划要少。

## 小结

在每一条SQL语句的计划输出中都包含着丰富的信息。在本章中，回顾了如何通过使用解释计划得出唯一的估计的计划信息，或者在语句执行后通过使用DBMSXPLAN命令从库高速缓存中提取出实际的计划信息。很多时候，你可能只能使用解释计划输出，尤其是如果查询的执行需要很长时间，不容易或者不可能等待查询执行完成再获取实际执行数据的时候。然而，为了最大可能地得到信息来决定索引、查询语法的改变，或者需要更新统计信息或参数设置，使用实际执行计划统计信息是必经之路。&#x20;

讲述了你可以使用计划信息来帮助诊断并解决SQL语句性能问题的一些方法。通过仔细查看计划输出，你可以识别次优索引或索引的缺失，并且可以确定统计信息是否过期，是否需要进行更新。使用你所得到的关于访问和联结数据的各种计划运算，以及理解如何阅读和有效地使用计划信息的知识，你不仅掌握了出现问题的时候快速和有效的解决方法，并且能够验证任何SQL语句的特征和性能印迹，从而你可以从一开始就写出高效的SQL语句。
