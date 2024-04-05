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

# 面向集合思维

要想成为写SQL语句的高级专家，最困难的一个转变就是从面向过程的思维方式转变到**面向声明**(或**面向集合**)的思维方式。通常，如果你使用各种编程语言已经很长时间了，要培养面向集合的思维方式是最困难的。如果你就是这样的，你可能非常熟悉IF-THEN-ELSE、WHILE-DOLOOP-ENDLOOP以及BEGIN-END等结构。这些结构支持通过过程化、一步一步、自上而下的方式来运行逻辑及数据。SOL语言并不打算按照过程化的观点来实现，而是以面向集合的方法实现。转变到面向集合的思维方式所花的时间越长，你真正成为能够写出功能正确并且高度优化的SOL语句的专家的时间也就越长。

在本章中，你将浏览到从过程化思维方法转变为非过程方法所需涉及的常见领域。你将开始去理解相对于顺序步骤来说如何使用数据元素集合。你还将看到一系列特殊集合运算(UNION、INTERSECT及MINUS)以及空值是如何影响面向集合的思维方式的。

## 面向集合的思维方式思考

首先需要做的是停止那些一次处理一行数据的过程化步骤思维。如果你一次只想处理一行，实现你的想法将会使用短语如“for each row dox”或者“while value is y dox”。试着把思路转移到使用类似于“for all”的短语上来。有关于此的一个简单的例子就是加数字。当你按过程化来考虑的时候，你就会想把一行的数值与另一行的数值加起来直到把所有行加到一起。对所有行求和的思维与此不同。这是个非常简单的例子，但类似的思维方式的转变同样适用于更复杂的不是那么明显的情况下。

例如，如果我让你生成一个所有在公司里每个工作岗位上干了同样年数的员工列表，你会怎么做?如果你按照过程化的思维方式来进行，你可能需要去查看每个工作岗位，计算出在这个岗位上的工作年限，然后与在其他各个岗位的工作年限比较。如果年数不匹配，那么你就不会把这个员工放到结果列表中。这种方法将会通过如下的一个自联结的查询来进行:

> 过程化与基于集合的方法对比

{% code lineNumbers="true" %}
```sql
select distinct emp_id
from job_history j1
where not exists (
    select null from job_history j2
    where j2.emp_id = j1.emp_id
        and round(months_between(j2.start_date,j2,end_date)/12,0) <>
            round(months_between(j1.start_date,j1,end_date)/12,0)
)
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption><p>过程化执行计划</p></figcaption></figure>

相反,如果你使用面向集合的观点来看待这个问题,你就会写出对表只进行一次访问的查询，按照员工进行分组,然后筛选出那些在某个岗位上工作的最短年数与某个岗位上工作的最长年数相一致的员工。

{% code lineNumbers="true" %}
```sql
select emp_id 
from job_history
group by emp_id 
having min(round(months_between(j2.start_date,j2,end_date)/12,0)) =
       max(round(months_between(j1.start_date,j1,end_date)/12,0))
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption><p>集合思维执行计划</p></figcaption></figure>

关键是要开始以完成后的结果的形式(而不是以处理步骤的形式)来思考。要找集合的特征而不是单独的步骤或行为。在基于集合的思维方式中,所有事物都以应用于集合的筛选条件或约束所定义的状态存在。你不再按照过程步骤来思考而是要按照集合的状态来思考。下图给出了处理步骤图与嵌套集合图之间的一个比较用来说明我的观点。

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption><p>处理步骤流程图与嵌套集合图</p></figcaption></figure>

处理流程图表明结果集(A)是通过一系列以其他步骤为基础的处理步骤来产生的最终答案。B通过遍历C和D得出，然后A通过遍历B和E得出。但是，嵌套集合图中将A看做是不同集合的组合的结果。

### 面向过程vs.面向集合例子

在这个例子中，任务是要计算出一个顾客在各个订单之间的平均天数。代码清单给出了通过面向过程的思维方式的实现方法。为了让例子的输出较短，我将只计算一个顾客的，但是由此可以很容易地转变为计算所有顾客的。

> 面向过程的思维方式

{% code lineNumbers="true" %}
```sql
--第一步
select customer_id,order_date
from orders
where customer_id = 102;
--第二步
select 
    trunc(order_date)-trunc(prev_order_date) days_betwwen
from (
    select customer_id,order_date
        lag(order_date,1,order_date)
        over (partition by customer_id order by order date)
        as prev_order_date
    from orders
    where customer_id = 102
);
--第三步
select 
    avg(trunc(order_date)-trunc(prev_order_date)) avg_days_betwwen
from (
    select customer_id,order_date
        lag(order_date,1,order_date)
        over (partition by customer_id order by order date)
        as prev_order_date
    from orders
    where customer_id = 102
);
```
{% endcode %}

这看上去相当优雅，不是吗?在这个例子中，我依次执行了一系列查询来展示我是如何思考的，并按照逐步进行的过程方法来书写查询语句。所做的事情就是按照order\_date的顺序读取102号顾客的每一行订单信息，然后使用LAG函数，回过头再看前一行的订单数据以获得该行的order\_date。当得到这两个order\_date(当前订单行的日期以及前一行订单的日期)以后利用这两个日期相减得出中间相差的天数就非常简单了。最后，我使用求平均值聚合函数来得到最终的答案。 你可能会指出这个查询是按照非常过程化的方式建立起来的。理解这种方式最好的办法就是依次来看几个不同的查询以展示如何建立最终结果集的。在这个过程中我可以看到相关详细信息。当以基于集合的思维方式进行思考的时候，你会发现并不需要去关心每一个单独的元素。下列代码清单给出了一个按照基于集合的思维方式来写的查询例子。

> 基于集合的思维方式

{% code lineNumbers="true" %}
```sql
select 
    (max(trunc(order_date))-min(trunc(order_date)))/count(*) as avg_day_between
from orders
where customer_id = 102;
```
{% endcode %}

这样怎么样?根本不需要任何花哨的技巧来解决这个问题。用来计算订单之间的平均天数所要做的事情就是计算出第一笔和最后一笔订单之间的天数以及总的订单数。不需要一步一步地来考虑问题，也不会写一个一行一行读取数据然后计算出结果的程序。所需要的就是把我考虑问题的思维方式转变到将集合数据作为一个整体来考虑。 我并不是完全无视过程化方法。可能有的时候你不得不采用这样的方法来完成工作。然而，我想鼓励你进行思维方式的转变:首先寻找基于集合的方式，只有在需要的时候才采用更大程度的过程化方法。通过这样做，你可能会发现自己可以得到更简单、直接的，通常性能也更好的解决方案。

## 集合运算

Oracle支持4种集合运算符:**`UNION、UNION ALL、MINUS以及INTERSECT`**。集合运算符将两个或更多SELECT语句的结果合并形成一个结果集。其与联结的区别就在于联结是用来将不同表中的列组合起来形成一行。**集合运算比较所输入查询的所有行并返回一个不包含重复值的行集。关于这点的例外就是使用UNION ALL，返回两个集合中的所有行，包含重复。UNION返回来自所有输入查询的不包含重复值的结果集。MINUS返回在第一个输入查询中存在但接下来的查询中不存在的非重复数据行。INTERSECT返回在所有输入查询中都存在的非重复行。**

所有进行集合运算的查询都必须符合下面的条件。&#x20;

* 所有的输入查询必须返回相同数目的列。&#x20;
* 每一列的数据类型必须与对应的其他输入查询一致(按照查询列清单中的顺序)。数据类型也可以不是直接匹配的，但只有在所有输入查询的数据类型都必须可以隐式转换为第一个输入查询的数据类型的情况下才是这样。
* ORDER BY子句不能在某个单独的查询中应用，只能用在整个查询的最后，用来对整个集合运算的结果集进行排序。&#x20;
* 列名源自第一个输入查询。

每个输入查询都先被单独处理然后进行集合运算。最后，如果指定了0RDER BY运算的话，将会应用于整个结果集。当使用UNION和INTERSECT的时候，运算因子是可以互换的(即查询的顺序并不要紧)。但是，在使用MINUS的时候，顺序就很重要，因为这个集合运算使用第一个输入查询结果作为基础来与其他查询的结果进行比较。除了UNION ALL以外的所有集合运算都需要对结果进行排序/取唯一值操作，这就意味着需要额外的支出来处理查询。如果你知道不会出现重复或者你并不关心是否会出现重复，请一定使用UNION ALL。

### UNION和UNION ALL

当需要将两个或更多个单独的查询的结果要合并成一个最终结果集的时候使用UNION和UNION ALL。下图使用维恩图来直观地说明这两种集合运算的结果集是如何生成的。

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption><p>UNION和UNION ALL结果集</p></figcaption></figure>

集合运算`UNION`将会返回两个查询的结果但会去掉重复的行，而`UNION ALL`运算则会返回所有行，其中包括重复行。正如前面提到的，当你需要去除重复的时候，使用`UNION`。但当你不关心重复行的存在与否或者预计不会出现重复的时候，选用`UNION ALL`。使用`UNION ALL`耗占的资源比使用UNION要少，因为`UNION ALL`不需要做去除重复的工作。这些工作可能会相当耗费资源以及延长响应时间。在Oracle10版本之前，使用排序运算来去除重复行。从Oracle10版本开始，可以使用`HASH UNIQUE`运算来去除重复行。`HASH UNIQUE`不进行排序而是比较散列值。我提及这一点是为了确定你认识到了即使结果集看上去是排过序的，也不能保证确实是这样，除非你显式地使用了ORDER BY子句。下列代码清单给出了使用UNION和UNION ALL的例子。

> UNION和UNION ALL的例子

{% code lineNumbers="true" %}
```sql
--UNION
select color from table1
union
select color from table2;

--UNION ALL
select color from table1
union all
select color from table2
```
{% endcode %}

这些例子展示了两个查询的UNION运算。记住可以有合并在一起的多个查询。

### MINUS

当第一个输入查询的结果作为基础数据集减去另一个输入查询结果作为最终结果集的时候使用MINUS。MINUS通常用来替代NOT EXISTS(反联结)询。所解决的问题可以描述为“我需要返回在数据行源A中存在但是在B中不存在的数据行集。”下图使用维恩图来直观地展现了MINUS运算的结果集是如何得到的。

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>MINUS结果集</p></figcaption></figure>

> MINUS的例子

{% code lineNumbers="true" %}
```plsql
select color from table1
minus
select color from table2
```
{% endcode %}

### INTERSECT

INTERSECT用来返回在所有输入查询中都存在的唯一行集。INTERSECT通常用来代替EXISTS(半联结)。所解决的问题可以描述为“我需要返回源A和B中都存在的数据行集”。下图通过一个维恩图来直观地展示了INTERSECT运算的结果集是如何生成的。

> INTERSECT的例子

{% code lineNumbers="true" %}
```plsql
select color from table1
intersect
select color from table2
```
{% endcode %}

## 集合与空值

你经常会听到一个术语--空值，但实际上，空值并不是一个值。空值最多就是一个标记我一直把空值的意思理解为“我不知道"。SQL语言对空值的处理不是那么直观--至少我是这样认为的。使用空值得到的结果就现实功能来说通常并不是我所想要的。

> 术语“空值”是错误的吗
>
> 严格来说，空值并不是一个值，而更像是缺了一个值。然而，术语“空值”被广泛使用。 如果久用SQL的话，你肯定会遇到一些人武断地说使用术语“空值”是多么的错误。&#x20;
>
> 但使用术语“空值”真的错了吗?
>
> 如果你在听讲座上，有反对意见请随便提。术语“空值”广泛使用于ANSI和ISO的SQL标准中。“空值”是SQL语言官方用语的一部分，因此在讨论这种语言的时候使用它是完全可以的。
>
> 但是要记住，SOL语言与关系理论之间是有区别的。一个真正挑剔的人可能会在说到SOL语言的时候支持使用“空值”,但是说到SOL语言不是严格依赖的关系理论时就会反对这个术语

### 空值与非直观结果

下列代码清单给出了一个简单的查询例子，我预期会出现某个结果集，但最终结果集却和我预想的不太一样。我想的是如果我查询哪些行不存在某个特定的值，并且未找到对应关系，甚至这一行中的这一列为空值，Oracle应该会将这一行包含在所返回的结果集中。

{% code lineNumbers="true" %}
```sql
select * from scott.emp;

select * from scott.emp where deptno in (10,20,30);

select * from scott.emp where deptno not in (10,20,30);

select * from scott.emp 
where deptno not in (10,20,30) or deptno is null;
```
{% endcode %}

这个代码清单说明了空值是多么令人沮丧:除非显式声明，它们不会被包含在结果集中。在该例子中，表中deptno为10、20或30。由于表中总共有14行，我希望通过一个查询来查找deptno不为10、20或30的行以显示剩下的那一行。但从查询的结果你可以看到我想错了。如果显式地把deptno为空值这个条件也包括进去，才得到了想要的所有雇员的列表。本以为在我的脑海中想把空值等同于一个空的字符串。但是，不管在我脑子中想将空值当作什么，空值就是空值。空值不能用来进行比较。空值不能与任何东西进行加、减、乘、除运算。如果这样做了，返回值还将是空值。下列代码清单展现了空值的这一特点以及它们是如何被包含在比较以及表达式中的。

> 比较和表达式中的空值

{% code lineNumbers="true" %}
```sql
-- 1 rows selected.
select * from scott.emp where deptno is null;

-- no rows selected.
select * from scott.emp where deptno = null;
```
{% endcode %}

因此，当在上面的代码清单中查询返回deptno为空值的行时，我必须提醒自己当对空值进行比较的时候，所得到的答案将会是“我不知道”。这就好像当你问我在你的冰箱里是否有橙汁时我也会回答“我不知道”一样。你的冰箱中可能有橙汁也可能没有，但我并不知道。因此我除了实事求是以外别无他法。

关系模型是基于两个值(真，假)的逻辑关系，但是SQL语言允许3个值(真、假以及未知的逻辑。问题就来自于此。混入了第3个值以后，SQL将会返回3值逻辑认为“正确”的结果，但这个结果就你的预想来说可能就不正确。在上面的例子中，没有数据行被选中的结果是正确的，因为有一行的deptno为空值。如果列的值是10、20或30之外的情况，你无法知道到底是什么。为了如实地回答，答案就必须是未知。就好像我并不知道你的冰箱里是否有汁一样!因此在你写SQL语句的时候必须要牢记空值的特性。如果你没有很谨慎地来对待空值，你的SQL语句就很可能返回错误的值。至少就你期望的结果来说是错误的。

### 集合运算中的空值

集合运算将空值作为一个可以用等式进行比较的值来对待。这相对于之前的讨论来说是很有趣并且出乎意料的。下列代码清单示出了在集合运算中是如何来对待空值的。

<pre class="language-plsql" data-line-numbers><code class="lang-plsql"><strong>--结果
</strong><strong>--NULL
</strong>select null from dual
union
select null from dual;
--NULL,NULL
select null from dual
union all
select null from dual;
--NULL
select null from dual
intersect
select null from dual;
--
select null from dual
minus
select null from dual;
--1,NULL
select 1 from dual
union
select null from dual;
--1,NULL
select 1 from dual
union all
select null from dual;
--
select 1 from dual
intersect
select null from dual;
--1
select 1 from dual
minus
select null from dual;
</code></pre>

在第一个例子中，当具有空值的两行进行联合时，你只会得到一行。这就表明了这两行是相等的，因此在进行联合的时候重复行就被去掉了。你可能也注意到了，这对于其他的集合运算也是一样的。因此请记住集合运算将所有空值作为相等的值来对待。

### 空值与GROUP BY和ORDER BY

如同在集合运算中一样，`GROUP BY和ORDER BY`子句也将空值作为可以用等式进行比较的值来对待。你将会注意到在分组和排序中，空值总是会像其他已知值那样被放在一起。下列代码清单给出了空值在`GROUP BY`和`ORDER BY`中是如何进行处理的一个例子。

> 空值与GROUP BY和ORDER BY

{% code lineNumbers="true" %}
```plsql
select comm,count(*)
from scott.emp
group by comm;

select comm,count(*) ctr
from scott.emp
group by comm
order by comm;

select comm,count(*) ctr
from scott.emp
group by comm
order by comm
nulls first;

select ename,sal,comm
from scott.emp
order by comm
```
{% endcode %}

前两个例子示出了`GROUP BY`子句中空值的行为。由于第一个查询返回的结果看上去好像是按comm列降序排列的，所以我需要利用第二个查询来说明我曾经在本书中指出过的一点:唯一确定排序的方法是使用`0RDER BY`子句。仅仅因为第一个查询中的结果看上去像排过序的可并不一定意味着它就是排过序的。当我在第二个查询中加上`ORDER BY`子句后，空值的分组移到了最下面。在最后一个`ORDER BY`的例子中，注意空值出现在最后。这并不是因为空值被当做“高位值”来对待，而是因为默认的排序规则就是将空值放在最后。如果你想把空值放在最前面，只需要像第3个例子中那样在`ORDER BY`子句后面加上`NULLS FIRST`。

### 空值与聚合函数

一些运算，像集合运算、分组和排序运算中对空值的不同处理方法也同样适用于聚合函数。当在聚合函数如`SUM、COUNT、AVG、MIN以及MAX`等所包含的列中出现空值的时候，它们将会被从聚合中去掉。如果去掉后集合变成空的，聚合则会返回空值。**此规则的一个例外就是COUNT聚合函数的使用。**对于空值的处理将取决于COUNT函数是使用一个列名还是一个常量(例如\*或1)来公式化。下列代码清单说明了聚合函数是如何来处理空值的。

> 空值与聚合函数

{% code lineNumbers="true" %}
```sql
select count(*) row_ct,count(comm) comm_ct,
    avg(comm) avg_comm,min(comm) min_comm,
    max(comm) max_comm,sum(comm) sum_comm
from scott.emp;

ROW_CT|COMM_CT|AVG_COMM|MIN_COMM|MAX_COMM|SUM_COMM|
------+-------+--------+--------+--------+--------+
    14|      4|     550|       0|    1400|    2200|
```
{% endcode %}

注意COUNT(\*)和COUNT(comm)结果的区别。使用号得到的结果是14，也就是所有的行数。而使用comm得到的答案则是4，仅仅是comm值为非空的行数。你也可以很容易地验证出空值在计算AVG、MIN、MAX以及SUM值之前就被去掉了，因为所有这些函数都得到了一个结果。如果空值没有被去掉，这些函数都将会返回空值。

## 小结

要想写出简单易懂并且性能会比按过程化方法来写更好的SQL语句，你需要掌握的核心技能就是以集合的方式来思考。当你按过程化来思考的时候，你就会尝试强制让非过程化的SQL语言按照不必要的方式来实现其功能。

在本章中,复习了面向过程的和面向集合的两种思考方法并讨论了如何将你的思维方式从面向过程转变到面向集合上来。当你进一步学习本书的时候,将基于集合的思维方式谨记于心。如果你发现自己还是按照过程化的方法一行一行地来考虑问题,停下来检视一下。你练习得越多这种转变就会变得越简单。











































