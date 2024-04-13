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

# 子查询因子化

子查询因子化(Subquery Factoring)。在Oracle 11gR2之前，Oracle官方文档很少提及它，仅提供了其用法的概要、少数几个限制条件以及一个简单的例子。如果我提到SELECT语句的WITH子句，你可能马上就会知道我指的是什么，因为这个术语更为大家所熟悉。

在本章中这两个术语都将会用到。随着Oracle11gR2(版本11.2)的发布，WITH子句得到了增强,具有了递归的能力。也就是说在一定的限制条件下因子化的子查询可以调用其自身。这样得到的值可能并不一定很明显。如果你曾经使用过`CONNECT BY`子旬来创建层次查询(Hierarchical Query)，你就会感谢递归子查询允许同样的功能以ANSI标准的形式来实现。

如果不了解术语子查询因子化，你可能听说过ANSI标准术语公共表表达式(`common table expression`，通常称为CTE)。公共表表达式是在1999 ANSI SQL标准中第一次进行声明的。由于某些原因，Oracle选择将这个名字模糊化。其他数据库供应商采用了公共表表达式的名称，那么也许Oracle选择子查询因子化就是为了与众不同吧。

## 标准用法

当首次被引人的时候，WITH子句最有用的特性之一就是消除复杂的SQL查询。当一个查询中包含大量的表和数据列的时候，想要搞清楚查询中的数据流向就变得很困难。通过使用子查询因子化，通过一个查询就可以将一些较复杂的部分移到主查询之外，从而使得查询更易于理解。下列代码清单中的查询使用PIV0T运算符生成了一个交叉数据分析报告。所采取的格式在定程度上增加了SQL语句的可读性，但在这方面还有很多需要做的。最里层的查询在Sales表的关键列上创建了一系列的聚合，而接下来的最外层查询只是提供了在PIVOT运算符中所出现列的列名，从而生成了每种产品不同渠道和季度的最终销售值。

> 没有进行子查询因子化的交叉数据分析查询

{% code lineNumbers="true" %}
```sql
select * from (
	select 
		product,
		channel,
		quarter,
		country,
		quantity_sold
	from (
		select 
			prod_name product,
			country_name country,
			channel_id channel,
			substr(calendar_quarter_desc,6,2) quarter,
			sum(amount_sold) amount_sold,
			sum(quantity_sold) quantity_sold
		from sh.sales join sh.times on times.time_id = sales.time_id
			join sh.customers on customers.cust_id = sales.cust_id
			join sh.countries on countries.country_id = customers.country_id
			join sh.products on products.prod_id = sales.prod_id
		group by
			prod_name,
			country_name,
			channel_id,
			substr(calendar_quarter_desc,6,2)
	)
) PIVOT (
	sum(quantity_sold)
	FOR (channel,quarter) IN (
		(5,'02') as CATALOG_Q2,
		(4,'01') as INTERNET_Q1,
		(4,'04') as INTERNET_Q4,
		(2,'02') as PARTNERS_Q4,
		(9,'03') as TELE_Q3,
	)
)
order by product,country
```
{% endcode %}

现在让我们使用WITH子句来将这个查询分解为易于理解的字节级大小的块。上面的这个SQL语句在下列代码清单中使用WITH子句建立3个因子化子查询来进行了重写，分别命名为`sales_countries`、`top_sales`和`sales_rpt`。注意`top_sales`和`sales_rpt`子查询都通过名称引用了其他子查询，就好像它们是一张表或视图那样。通过选用使每个子查询的内容易于理解的名字，SQL语句的可读性增强了。例如，子查询名称`sales_countries`指的是销售所发生的国家，`top_sales`收集销售数据，而`sales_rpt`子查询对这些数据进行聚合。`sales_rpt`子查询的结果被用来在主查询中回答:“每种产品每季度在各个国家的销售分类细账如何?"这样的问题。进行子查询因子化的SQL语句结构使得理解代码的含义变得更容易了。

除此以外，直接与PIV0T运算符相关的语句同样还在SOL语句的最下面的部分，这进一步增强了可读性。

> 进行子查询因子化的交叉表

{% code lineNumbers="true" %}
```sql
with sales_countries as (
	select 
		cu.cust_id,
		co.country_name
	from sh.countries co,sh.customers cu
	where cu.country_id = co.country_id
),
top_sales as (
	select 
		p.prod_name,
		sc.country_name,
		s.channel_id,
		t.calendar_quarter_desc,
		s.amount_sold,
		s.quantity_sold
	from sh.sales s join sh.times t on t.time_id = s.time_id
		join sh.customers c on c.cust_id = s.cust_id
		join sales_countries sc on sc.cust_id = c.cust_id
		join sh.products p on p.prod_id = s.prod_id
),
sales_rpt as (
	select
		prod_name product,
		country_name country,
		channel_id channel,
		substr(calendar_quarter_desc,6,2) quarter
		sum(amount_sold) amount_sold,
		sum(quantity_sold) quantity_sold
	from top_sales
	group by 
		prod_name,
		country_name,
		channel_id,
		substr(calendar_quarter_desc,6,2)
)

select * from (
	select prodcut,channel,quarter,country,quarter_sold
	from sales_rpt
) pivot (
	sum(quantity_sold)
	FOR (channel,quarter) IN (
		(5,'02') as CATALOG_Q2,
		(4,'01') as INTERNET_Q1,
		(4,'04') as INTERNET_Q4,
		(2,'02') as PARTNERS_Q4,
		(9,'03') as TELE_Q3,
	)
)
order by product,country
```
{% endcode %}

尽管这并不是一个非常复杂的SQL例子，但确实可以用来说明WITH子句是如何能够被用来增强SQL语句的可读性和可维护性的。通过使用这一技术，大而复杂的查询可以变得更易于理解。

## SQL优化

当一个SQL查询被设计或修改以利用子查询因子化时,在优化器为查询建立执行计划的时候可能会有一些不太隐晦的变化。下面这段话引自Oracle11gR2文档(_`Oracle Database SOL,Language Reference`_)中关于SELECT的部分中子查询因子化小节下的内容。

`WITH query_name`子句可以让你为子查询块分配一个名称。然后你就可以通过声明`query_name`在查询中多次引用这个子查询块。Oracle数据库通过将这个查询名称作为内嵌视图或临时表对待来优化查询。

注意Oracle可能将因子化的子查询作为临时表来处理。在一个表被引用多次的查询中，这可能是一个独特的性能上的优势，因为Oracle可以物化查询结果集，从而避免多次执行一些非常耗占资源的数据库运算。在这里需要注意的是只是“可能”的独特性能优势。需要牢记于心的一点是物化结果集需要创建一个临时表并将数据行插入其中。如果同一个结果集将会被引用很多次的话，这样做可能是很值得的，否则就有可能极大地降低性能。

使用提示词显示指定

{% code lineNumbers="true" %}
```sql
select /* inline */ * from table_name

select /* materialize */ * from table_name
```
{% endcode %}

1. **INLINE提示**: 这个提示告诉优化器将一个查询块嵌入到主查询中，而不是作为一个独立的子查询来执行。这可以减少查询中的子查询数量，从而减少了优化器需要处理的复杂性。然而，这种方式可能会增加主查询的复杂度，因此并不总是最佳选择。
2. **MATERIALIZE提示**: 这个提示告诉优化器将一个查询块的结果集存储在一个临时表中，以便在查询的其他部分中重复使用。这可以降低查询的成本，特别是当一个查询块被多次引用时。然而，这可能会增加存储和内存开销，因为需要额外的空间来存储结果集。

## 递归子查询

Oracle 11.2中新出现的是递归子查询因子化(`recursive subquery factoring`，本章中以下简称为**RSF**)。正如你可能会猜想的那样，ANSI标准中这个特性的名称是递归公共表表达式(`recursive common table expression`)。不管你怎么称呼它，Oracle在很久以前就已经以SELECT语句中的`CONNECT BY`子句的形式具有类似的特性。这一特性在Oracle 11gR2中得到了增强。

### CONNECT BY

下列代码清单中的传统的`CONNECT BY`查询看起。内嵌视图`emp`被用来与`EMPLOYEE`和`DEPARTMENT`表进行联结，然后将一个数据集提供给`SELECT …… CONNECT BY`语句。用一个`PRIOR`运算符来将当前的`EMPLOYEE_ID`与另一行中的`MANAGER_ID`列值进行匹配。反复地这么做就建立了一个递归查询。

下列代码清单在输出中包含了一些额外的列来帮助说明`PRIOR`运算符是如何运算的。让我们从**Lex De Haan**这一行开始来看一下输出。你可以看到**Lex**的`EMPLOYEE_ID`是`102`。`PRIOR`运算符将会查找所有`MANAGER_ID`是`102`的数据行并将它们放在**Lex De Haan**的层级下。唯一满足这个标准的一行是**Alexander Hunold**那一行，`EMPLOYEE_ID`为`103`。然后对**Alexander Hunold**重复这个过程:有没有哪些行的`MANAGER_ID`是`103`?找到有4行数据的`MANAGER_ID`为`103`:员分别为**Valli Pattaballa、Diana Lorentz、Bruce Ernst和David Austin**，因此这些都被包括在**Alexander Hunold**之下的输出中由于这4名员工的`EMPLOYEE_ID`值都没有任何`MANAGER_ID`与之相匹配,Oracle将会转到还没有进行处理的数据行的层级上去(在这个例子中，是**Alberto Errazuriz**)并继续进行处理，直到所有的数据行都处理完毕。&#x20;

`START WITH`子句是用来指引从`MANAGER_ID`为空的那一行开始。因为这是一个在最上层只有一个人的组织层次结构，从而使得查询从**Stephen King**开始。作为**CEO**，**King**先生没有上级，因此他那一行的`MANAGER_ID`列就设为空。 `LEVEL`伪列保存了递归的深度值,使得可以通过一个简单的方法来对输出进行缩进，从而可以直观地看出组织层次结构。

> 基本的CONNECT BY

{% code lineNumbers="true" %}
```sql
select 
	lpad(' ',level*2-1,' ') || emp.emp_last_name emp_last_name,
	emp.emp_first_name,
	emp.employee_id,
	emp.mgr_last_name,
	emp.mgr_fisrt_name,
	department_name
from (
	select /*+ inline gather_plan_statistics */
		e.last_name emp_last,
		e.first_name emp_first_name,
		e.employee_id,
		d.department_id,
		es.last_name mgr_last_name,
		es.first_name mgr_first_name
	from hr.employees e left join hr.departments d on d.department_id = e.department_id
		left join hr.employees es on es.employee_id = e.manager_id
) emp
connect by prior emp.employee_id = emp.manager_id
start with emp.manager_id is null
order by siblings by emp.emp_last_name;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption><p>基本的CONNECT BY</p></figcaption></figure>

### RSF(`recursive subquery factoring`)

对`EMPLOYEES`表的示例查询在下列代码清单中使用`RSF`进行了重写，其中主要的子查询是`emp_recurse`。这个例子中的定位点成员通过选取唯一的一行`MANAGER_ID`为空的数据行来选出层级中最上面的一行。这与上面代码清单中的`START WITH EMP.MANAGER_ID IS NULL`是等价的。递归成员通过将其与`emp`查询进行联结来引用定义性的查询`emp_recurse`。这个联结用来查找每个员工的经理所对应的行，与上面代码清单中的`CONNECT BY PRIOREMP.EMPLOYEE_ID = EMP.MANAGER_ID`等价。下列代码清单的结果与上面代码清单是一致的。

> 基本的递归子查询因子化

{% code lineNumbers="true" %}
```sql
with emp as (
	select /*+ inline gather_plan_statistics  */
		e.last_name,
		e.first_name,
		e.employee_id,
		e.manager_id,
		d.department_name
	from hr.employees e left join hr.department_id d on d.department_od = e.department_id
),
emp_recures(last_name,first_name,employee_id,manager_id,department_name,lvl) as (
	select 
		e.last_name,
		e.first_name,
		e.employee_id,
		e.manager_id,
		e.department_name,
		1 as lvl
	from emp e where e.manager_id is NULL
	union all
	select
		emp.last_name,
		emp.first_name,
		emp.employee_id,
		emp.manager_id,
		emp.department_name,
		emp.lvl + 1 as lvl
	from emp join emp_recurse empr on empr.employee_id = emp.manager_id
)
search depth first by last_name set order1
select 
	lpad(' ',lvl*2-1,' ') || er.last_name last_name,
	er.first_name,
	er.department_name
from emp_recurse er;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>基本的递归子查询因子化</p></figcaption></figure>

尽管新的`RSF`方法第一眼看上去显得有点冗长，它的工作原理的基础理解起来比`CONNECT BY`要简单,并且允许进行更复杂的查询。递归的`WITH`子句需要两个查询块:**定位点成员**和**递归成员**，这两个子查询块必须通过集合运算符`UNION ALL`结合到一起。定位点成员是`UNION ALL`之前的查询而递归成员是其后面的查询。递归子查询必须引用定义子查询—这样做了，就是进行了递归。

> RSF的限制条件

正如你可能会猜想的那样，RSF的使用比`CONNECT BY`要灵活得多。但是，它的使用也有一些限制。按照11gR2文件中对于`SELECT`语句的说法，下面的这些元素不能在RSF的递归成员中使用:&#x20;

* DISTINCT关键字或GROUP BY子句；
* model子句；
* 聚合函数。但是，在SELECT列表中可以使用分析函数；
* 引用query\_name的子查询；
* 引用query\_name作为右表的外联结。

> 与CONNECT BY的不同点

与CONNECTBY相比较，在使用RSF的时候有几个不同点；你可能会奇怪`LEVEL`伪列发生了什么，在这个查询中并没有这一列，而是被LVL列取代了。我将在稍后来说这个问题。同时还要注意`RSF`查询所返回的列必须在查询定义中声明。另一个新的特性就是`SEARCH DEPTH FIRST`。默认的搜索是`BREADTH FIRST`，这通常不是一个层级型查询所想要的输出。下列代码清单中给出了没有使用`SEARCH`子句或将其设置为`BREADTH FIRST`的输出。这样的搜索在返回任何子数据行之前返回每一层级上的兄弟数据行。指定`SEARCH DEPTH FIRST`将会按照层级的顺序返回数据行。SEARCH子句中的SET ORDER1部分将ORDER1伪列的值设置为数据行返回的顺序值，类似于你在使用ROWNUM时那样但你对这一列进行了命名。这在后面的例子中也会用到。

> 默认的BEREADTH FIRST搜索

{% code lineNumbers="true" %}
```sql
search bereadth first by last_name set order1
select 
	lpad(' ',lvl*2-1,' ') || er.last_name last_name,
	er.first_name,
	er.department_name
from emp_recurse er;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

注意在上列代码清单中所使用的`SEARCH`子句指定了按照`LAST_NAME`进行搜索。也可以是按照`FIRST_NAME`，或者按照一个列的列表，例如`LAST_NAME`,`FIRST_NAME`来进行搜索。这样做控制了每一个层级中各行的顺序。`SEARCH`子句以`SET ORDER1`结尾。这有效地将`ORDER1`伪列加入到了递归子查询所返回的列中。在接下来的一些例子中你会看到更多这样的应用。

## 复制CONNECT BY功能

伴随着Oracle数据库版本的改进，`CONNECT BY`子句的功能也在不断发展。`CONNECT BY`中有一些层级查询运算符、伪列和一个函数在RSF中是无法直接使用的。但是，它们所提供的功能可以在RSF中复制。这些功能可能不是完全模仿使用`CONNECT BY`时的情况，但同样也可能做你需要做的事情。从`RSF`得到你所需的功能有时候需要放下键盘来想一想你所要实现的结果，而不是去想你要如何来编代码。观点的变化给你从所写的SQL语句中很容易地得到想要的结果带来的帮助是很令人吃惊的。

下表中列出了`CONNECT BY`的运算符和伪列。我将根据需要逐个讲述一遍，给出其在`CONNECT BY`中用法的例子，然后在`RSF`中复制同样的功能。记住`RSF`是多功能的，因此实现方式是多种多样的。请不必拘束，多加练习并找出得到相同结果的其他方法。

> CONNECT BY函数、运算符和伪列

<table><thead><tr><th width="102">类型</th><th width="229">名称</th><th>用途</th></tr></thead><tbody><tr><td>函数</td><td>SYS_CONNECT_BY_PATH</td><td>返回当前数据行的所有祖先</td></tr><tr><td>运算符</td><td>CONNECT_BY_ROOT</td><td>返回根数据行的值</td></tr><tr><td>运算符</td><td>PRIOR</td><td>用来表明层级型查询，在递归子查询中不需要</td></tr><tr><td>伪列</td><td>CONNECT_BY_ISCYCLE</td><td>在层级中检测循环</td></tr><tr><td>参数</td><td>NOCYCLE</td><td>CONNECT BY的参数，与CONNECT_BY_ISCYCLE一起使用</td></tr><tr><td>伪列</td><td>CONNECT_BY_ISLEAF</td><td>标识叶子数据行</td></tr><tr><td>伪列</td><td>LEVEL</td><td>用来表明层级中的深度</td></tr></tbody></table>

还将讨论涉及`RSF`中`SEARCH`子句的内容，因为它对于一些问题的解决有指导性作用。

### LEVEL伪列

从LEVEL伪列开始。这在层级型查询中经常被用来进行输出缩进，使得层级看起来很直观。下列代码清单给出了一个如何生成LEVEL的简单例子。随着层级深度的增加，LEVEL值也增加了。同样地，当层次降低一级的时候，LEVEL的值也相应减小。

> LEVEL伪列

{% code lineNumbers="true" %}
```sql
select
	LPAD(' ',level*2-1,' ') ||e.last_name last_name,
	level
from hr.employees e
connect by prior e.employee_id = e.manager_id
start with e.manager_id is null
order siblings by e.last_name;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>LEVEL伪列</p></figcaption></figure>

这在RSF中也同样可以实现，尽管你可能需要稍微费点工夫。详见下列代码清单。这样也可以实现可能确实让人觉得有点意外。LVL的值只会增加，不会减小。回忆一下RSF的默认搜索方法是`BREADTH FIRST`。显然Oraclc是按照同级兄弟节点的顺序来进行处理的，最开始是层级的顶层(King)，接下来是其下一层级的子数据行，直到到达最后一行。这样的行为方式也可以让你来解决一些其他的问题。

> 创建一个LVL列

{% code lineNumbers="true" %}
```sql
with emp_recures(employee_id,manager_id,last_name,lvl) as (
	select 
		e.employee_id,
		null,
		e.last_name,
		1 as lvl
	from employee e where e.manager_id is NULL
	union all
	select
		e1.employee_id,
		e1.manager_id,
		e1.last_name,
		e2.lvl + 1 as lvl
	from employee e1 join emp_recurse e2 on e2.employee_id = e1.manager_id
)
search depth first by last_name set last_name_order
select 
	lpad(' ',lvl*2-1,' ') || r.last_name last_name,
	r.lvl
from emp_recurse r
order by last_name_order;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption><p>创建一个LVL列</p></figcaption></figure>

### SYS\_CONNECT\_BY\_PATH函数

这个函数是用来返回组成层级的直到当前行的值。这最好通过一个例子来进行解释，例如下列代码清单中的例子。这里的`SYS_CONNECT_BY_PATH`函数被用来建立一个冒号分隔的从根到节点的层级。

> SYS\_CONNECT\_BY\_PATH

{% code lineNumbers="true" %}
```sql
select
	LPAD(' ',level*2-1,' ') ||e.last_name last_name,
	SYS_CONNECT_BY_PATH(last_name,':') path
from hr.employees e
connect by prior e.employee_id = e.manager_id
start with e.manager_id is null
order siblings by e.last_name;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption><p>SYS_CONNECT_BY_PATH</p></figcaption></figure>

尽管`SYS_CONNECT_BY_PATH`函数不能在`RSF`查询中使用，你可以通过与重新产生`LEVEL`伪列几乎相同的方法来复制这个函数的功能。现在你不用使用计数器来计数,而是附加上一个字符串值。下列代码清单中示出了具体做法。

> 模拟SYS\_CONNECT\_BY\_PATH函数

{% code lineNumbers="true" %}
```sql
with emp_recures(employee_id,manager_id,last_name,lvl,path) as (
	select 
		e.employee_id,
		null,
		e.last_name,
		1 as lvl,
		':'||to_char(e.last_name) as path
	from employee e where e.manager_id is NULL
	union all
	select
		e1.employee_id,
		e1.manager_id,
		e1.last_name,
		e2.lvl + 1 as lvl,
		e2.path || ':' || e1.last_name as path
	from employee e1 join emp_recurse e2 on e2.employee_id = e1.manager_id
)
search depth first by last_name set last_name_order
select 
	lpad(' ',lvl*2-1,' ') || r.last_name last_name,
	r.path
from emp_recurse r
order by last_name_order;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption><p>模拟SYS_CONNECT_BY_PATH函数</p></figcaption></figure>

看一下这个SQL，你可能会注意到这里有一些事情是`SYS_CONNECT_BY_PATH`做不了的。例如，考虑一下如果你需要将层级显示为逗号分隔的列表。在这里只需要将冒号“:”替换为逗号“,”即可。而`SYS_CONNECT_BY_PATH`函数的问题在于输出中的第一个字符必须是冒号。使用`RSF`方法,你可以简单地去掉定位点成员的分隔符,并将递归成员的分隔符修改为逗号这在下列代码清单中给出，同时还给出了一个输出示例。如果你愿意，路径的第一个字符可以保持仍为冒号而将后面的值用逗号进行分隔。

{% code lineNumbers="true" %}
```sql
with emp_recures(employee_id,manager_id,last_name,lvl,path) as (
	select 
		e.employee_id,
		null,
		e.last_name,
		1 as lvl,
		e.last_name as path
	from employee e where e.manager_id is NULL
	union all
	select
		e1.employee_id,
		e1.manager_id,
		e1.last_name,
		e2.lvl + 1 as lvl,
		e2.path || ',' || e1.last_name as path
	from employee e1 join emp_recurse e2 on e2.employee_id = e1.manager_id
)
search depth first by last_name set last_name_order
select 
	lpad(' ',lvl*2-1,' ') || r.last_name last_name,
	r.path
from emp_recurse r
order by last_name_order;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

### CONNECT\_BY\_ROOR运算符

这个运算符对CONNECT BY语法进行了强化，使得它可以返回当前行的根节点。在HR.EMPLOYEES表的例子中，所有行都会将“King”作为根节点返回。但是，你可以稍微做一点修改，临时改变一下Neena Kochhar那一行，将她放到与公司总裁Steven King相同的层级中。这样就可以使用`CONNECT_BY_ROOT`运算符限制输出来显示Ms.Kochhar的层级。你可以在下列代码清单中看到结果

> CONNECT\_BY\_ROOT

{% code lineNumbers="true" %}
```sql
update hr.employees set manager_id = null where last_name = 'Kochhar';

select /*+ inline gather_plan_statistics  */
	level,
	LPAD(' ',level*2-1,' ') ||e.last_name last_name,
	CONNECT_BY_ROOT last_name as root,
	SYS_CONNECT_BY_PATH(last_name,':') path
from hr.employees e
where connect_by_root last_name = 'Kochhar'
connect by prior e.employee_id = e.manager_id
start with e.manager_id is null;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption><p>CONNECT_BY_ROOT</p></figcaption></figure>

> 复制CONNECT\_BY\_ROOT运算符功能

{% code lineNumbers="true" %}
```sql
update hr.employees set manager_id = null where last_name = 'Kochhar';

with emp_recures(employee_id,manager_id,last_name,lvl,path) as (
	select /*+ gather_plan_statistics  */
		e.employee_id,
		null as manager_id,
		e.last_name,
		1 as lvl,
		':'||to_char(e.last_name) as path
	from employee e where e.manager_id is NULL
	union all
	select
		e1.employee_id,
		e1.manager_id,
		e1.last_name,
		e2.lvl + 1 as lvl,
		e2.path || ':' || e1.last_name as path
	from employee e1 join emp_recurse e2 on e2.employee_id = e1.manager_id
)
search depth first by last_name set order1,
emp as (
	select 
		lvl,
		last_name,
		path,
		SUBSTR(path,2,instr(path,':',2)-2) root
	from emp_recurse
)
select 
	lvl,
	lpad(' ',lvl*2-1,' ') || last_name last_name,
	root,
	path
from emps
where root = 'Kochhar';
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption><p>复制CONNECT_BY_ROOT运算符功能</p></figcaption></figure>

这并不是CONNECT\_BY\_RO0T运算符功能的一个完美的复制。在这个例子中，它完成了所需要做的事情。但是系统内嵌的`CONNECT_BY_ROOT`运算符在指定层级和返回该层级上的根值方面具有更多的灵活性。上面给出的例子还需要进行更多的修改来实现这个功能。但是，你会发现例子中所做的工作已经可以满足绝大多数应用。

### CONNECT\_BY\_ISCYCLE伪列和NOCYCLE参数

`CONNECT_BY_ISCYCLE`伪列使得在层级中检测循环变得很容易。这通过下列代码清单中的SOL语句来说明。在这里通过更新`HR.EMPLOYEES`表中的`President`行信息,将`Smith`设置为`King`的经理来故意引入了一个错误。这将导致`CONNECT_BY`中出现错误。

> CONNECT BY 中的循环错误

{% code lineNumbers="true" %}
```sql
update hr.employees set manager_id = 171 where employee_id = 100;

select
	LPAD(' ',level*2-1,' ') ||e.last_name last_name,
	first_name,
	employee_id,
	level
from hr.employees e
start with e.manager_id = 100
connect by prior e.employee_id = e.manager_id;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption><p>CONNECT BY 中的循环错误</p></figcaption></figure>

在输出中，Smith是King的经理，你知道这是不正确的。但如果不是已经知道了问题是什么，你要如何来发现这个问题呢?这就是`NOCYCLE`参数和`CONNECT_BY_ISCYCLE`大展拳脚的地方。它们可以被用来检测层级中的循环。`NOCYCLE`参数可以阻止`ORA-1436`错误的发生，使得所有行都可以被输出。`CONNECT_BY_ISCYCLE`运算符使得你可以很容易地找到导致错误发生的行。

如下列代码清单中所示，`CONNECT_BY_ISCYCLE`的值为1,表示`Smith`的那一行数据导致了错误。接下来的查询查找`Smith`的数据，所有一切看上去都很正常。最后，你再次查询这个表，这一次使用`Smith`的员工ID来寻找他所管理的所有员工。错误就很明显了—公司总裁没有经理。因此解决问题的办法就是将这一行的`MANAGER_ID`设置回空值。

> 通过CONNECT\_BY\_ISCYCLE检测循环

{% code lineNumbers="true" %}
```sql
update hr.employees set manager_id = 171 where employee_id = 100;

select
	LPAD(' ',level*2-1,' ') ||e.last_name last_name,
	first_name,
	employee_id,
	level,
	connect_by_iscycle
from hr.employees e
start with e.manager_id = 100
connect by nocycle prior e.employee_id = e.manager_id;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```sql
select last_name,first_name,employee_id,manager_id
from hr.employees
where employee_id = 171 OR manager_id = 171;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

那么，在`RSF`中你要如何来实现这一点呢?这是很简单的，因为Oracle已经提供了内嵌的`CYCLE`子句来很方便地在递归查询中检测循环。它在某种程度上比`CONNECT_BY_ISCYCLE`更强大的一点就是它可以让你来确定哪些值被用来表明发生了循环，并且同时提供一个列名。下列代码清单使用与上面代码清单中同样的数据错误，但这一次你将使用一个递归因子化查询。

> 在递归查询中检测循环

{% code lineNumbers="true" %}
```sql
update hr.employees set manager_id = 171 where employee_id = 100;

with emp_recures(employee_id,manager_id,last_name,first_name,lvl) as (
	select /*+ gather_plan_statistics  */
		e.employee_id,
		null as manager_id,
		e.last_name,
		e.first_name,
		1 as lvl
	from employee e where e.manager_id = 100
	union all
	select
		e1.employee_id,
		e1.manager_id,
		e1.last_name,
		e1.first_name,
		e2.lvl + 1 as lvl
	from employee e1 join emp_recurse e2 on e2.employee_id = e1.manager_id
)
search depth first by last_name set order1
CYCLE employee_id set is_cycle TO '1' DEFAULT '0'
select 
	lpad(' ',lvl*(2-1),' ') || last_name last_name,
	first_name,
	employee_id,
	lvl,
	is_cycle
from emps
order by order1;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption><p>在递归查询中检测循环</p></figcaption></figure>

注意`CYCLE`子句是如何让你将`IS_CYCLE`列的值设置为0或1的。这里只允许单值字符。这一列的名称同样是用户自定义的，在本例中设置为`IS_CYCLE`。检查输出，可以看到`RSF`中的`CYCLE`子句在指明导致数据循环的行时做得更好。出现错误的数据行很清楚地标记为King那一行，因此你可以查询那一行并迅速确定错误所在。

### CONNECT\_BY\_ISLEAF伪列

最后，还有一列`CONNECT_BY_ISLEAF`伪列。这可以很方便地用来在层级数据中识别叶子节点。你可以看到下列代码清单的输出中`CONNECT_BY_ISLEAF`值为1的叶子节点都被标了出来。

> CONNECT\_BY\_ISLEAF伪列

{% code lineNumbers="true" %}
```sql
select
	LPAD(' ',level*(2-1),' ') ||e.last_name last_name,
	connect_by_isleaf
from hr.employees e
start with e.manager_id is null
connect by prior e.employee_id = e.manager_id
order siblings by e.last_name;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption><p>CONNECT_BY_ISLEAF伪列</p></figcaption></figure>

在`RSF`中要想复制这一点还是具有一些挑战性的。可能有很多方法可以实现这一点，但都有一定的局限性。这是那些需要稍微多想一点才能解决的问题之一，其中的“解决”指的是你能够得到想要的输出，但并不一定要完全复制`CONNECT_BY_ISLEAF`的功能。

在这个例子中，你需要在员工层级中标识出叶子节点。从定义上来说，叶子节点都不能是经理，所以实现这一点的方法之一就是确定哪些行是经理。所有不是经理的行就是叶子节点。

下列代码清单中采用这种方法来解决问题。这样做的成本就是对`HR.EMPLOYEES`表多进行了两次扫描并多了3次索引扫描，但如果必须使用RSF，这是得到所需结果的一种方法。LEAVES子查询被用来寻找叶子节点，然后将结果与`EMPLOYEES`表进行左外联结。`LEAVES.EMPLOYEEID`列的值(或缺少这个值)表明当前行是否是叶子。

> 在递归查询中找出叶子节点

{% code lineNumbers="true" %}
```sql
with leaves as (
	select employee_id 
	from hr.employees
	where employee_id not in (
		select manager_id
		from hr.employees
		where manager_id is not null
	)
),
emp(manager_id,employee_id,last_name,lvl,isleaf) as (
	select
		manager_id,
		e.employee_id,
		e.last_name,
		1 as lvl,
		o as isleaf
	from employee e where e.manager_id is null
	union all
	select
		e1.manager_id,
		nvl(e1.employee_id,null) employee_id,
		e1.last_name,
		e2.lvl + 1 as lvl,
		decode(l.employee_id,null,0,1)
	from employee e1 join leaves e2 on e2.employee_id = e1.manager_id
		left join leaves l on l.employee_id = e.employee_id
)
search depth first by last_name set order1
select 
	lpad(' ',lvl*(2-1),' ') || last_name last_name,
	isleaf
from emp;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption><p>在递归查询中找出叶子节点</p></figcaption></figure>

实现这一点的另一种方法见下列代码清单。在这里分析函数`LEAD()`使用`LVL`列的值来确定数据行是否为叶子节点。虽然这样确实避免了上面代码清单中所进行的两次索引扫描，但正确地确定一行数据是否为叶子节点还取决于输出的顺序，如第16行所示。`LEAD()`函数依赖于`SEARCH`子句中所设定的`LAST_NAME_ORDER`列的值。

> 使用LEAD()来寻找叶子节点

{% code lineNumbers="true" %}
```sql
with emp(manager_id,employee_id,last_name,lvl) as (
	select
		manager_id,
		e.employee_id,
		e.last_name,
		1 as lvl
	from employee e where e.manager_id is null
	union all
	select
		e1.manager_id,
		nvl(e1.employee_id,null) employee_id,
		e1.last_name,
		e2.lvl + 1 as lvl
	from employee e1 join leaves e2 on e2.employee_id = e1.manager_id
)
search depth first by last_name set last_name_order
select 
	lpad(' ',lvl*(2-1),' ') || last_name last_name,
	lvl,
	lead(lvl) over(order by last_name_order) leadlvlorder,
	case 
		when (lvl-lead(lvl) over(order by last_name_order))<0
		then 0
		else 1
	end isleaf
from emp;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption><p>使用LEAD()来寻找叶子节点</p></figcaption></figure>

如果`SEARCH`子句从`DEPTH FIRST`改为`BREADTH FIRST`会发生什么呢?初看上去是很优雅的解决方案，但它因为依赖于数据的顺序而显得有点脆弱，所以输出有可能就是不正确的。

## 小结

尽管在大多数实践中你都可以在递归因子化子查询中复制`CONNECT BY`的功能，但问题是，你应该这样做吗?在很多情况下，使用`CONNECT BY`语法更简单，尽管这个语法需要稍花点工夫去熟悉。在`RSF`中做同样的事情在多数情况下需要更多的`SQL`代码。并且，`CONNECT BY`可以产生比`RSI`更好的执行计划，尤其是对于相对简单的查询。但要记住，不管怎么样，`RSF`是一个新特性，在Oracle的后续版本中可能会不断得到改进。

此外，也可能有很多的理由让你不使用`CONNECT BY`。可能你需要在应用中保持ANSI兼容性或者写出可以在其他支持递归公共表表达式的数据库上运行的层级型查询,能够简化在不同的数据库上运行的应用代码。在这样的情况下，RSF是很有用的。

不管为什么需要层级型查询，只要有一点点创造性，你就可以使用递归因子化子查询来写出恰当的针对层级数据的查询，并且它们能够做现在`CONNECT BY`所做的任何事情。
