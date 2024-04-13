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

# 查询语法

## 语法结构

{% code lineNumbers="true" %}
```sql
SELECT
	[DISTINCT|ALL]
	select_list
FROM table_list
[where_clause]
[group_by_clause]
[HAVING condition]
[order_by_clause]

【语法说明】
SELECT:查询动作关键字，也是必需关键字。
[DISTINCTIALL]:描述列表字段中的数据是否去除重复记录。
select_list:需要查询的字段列表，也可以说是占位符。可以是一个字段，也可以是多个字段。
FROM:必需关键字，表示数据的来源。
[where_clause]:查询的WHERE条件部分。
[group_by_clause ]:GROUP BY子句部分。
[HAVING condition ]:HAVING子句部分。
[order_by_clause]:排序。
```
{% endcode %}

## 字段具体语法

{% code lineNumbers="true" %}
```sql
{
* | [schema.]{table | view}.* | expr[ [AS] c_alias ]
}

【语法说明】
schema:模式名称。
table|view:表或视图。
expr:表达式。
[c_alias]:别名。
```
{% endcode %}

{% hint style="info" %}
SELECT语句中允许利用表达式或函数对符合条件的数据进行处理。
{% endhint %}

## ORDER BY基本语法

{% code lineNumbers="true" %}
```sql
ORDER BY 
{expr | position |c_alias}
[ASC | DESC]
[NULLS FIRST | NULLS LAST]
[
    , {expr | position | c_alias}
    [ASC | DESC]
    [NULLS FIRST | NULLS LAST]
]...

【语法说明】
ORDER BY:排序关键字。
expr:表达式。
position:表中列的位置。
c_alias:别名。
[ASC|DESC]:升序或降序。
NULLS FIRST|NULLS LAST:对空字段的处理方式。
可以根据多个字段排序。
```
{% endcode %}

{% hint style="info" %}
NULL值在排序过程中是个比较特殊的值类型，默认情况下排序时把它看成最大值。当排序的记录中出现NULL值时，默认情况下，升序排列时它在最后，降序排列时它在首位。其实NULL值在排序时，具体在前还是在后开发人员是可以指定的。NULLS FIRST(升序排在首位)，NULLS LAST(降序排在末位)。
{% endhint %}

排序时允许使用查询列表中字段的位置来作为排序字段，这么做一是为了方便，二是为了防止使用UNION时出现错误。

{% hint style="info" %}
利用字段在查询列表中的位置作为排序字段时，表示位置的数字不能超出查询列表中字段的个数。
{% endhint %}

## WHERE检索条件

WHERE条件子句中可以使用的操作符主要有关系操作符、比较操作符和逻辑操作符

1. 关系操作符包括:\
   <、<=、>、>=、=、!=、<>。
2.  比较操作符包括:

    IS NULL:如果操作数为NULL返回TRUE LIKE:模糊比较字符串值。 BETWEEN...AND.:验证值是否在范围之内。 IN:验证操作数在设定的一系列值中。
3. 逻辑操作符包括: \
   AND:两个条件都必须得到满足。 OR:只要满足两个条件中其中的一个。 NOT:与某个逻辑值取反。

## GROUP BY和HAVING子句

{% code lineNumbers="true" %}
```sql
GROUP BY
{ expr | {ROLLUP | CUBE} {{expr [, expr]...}}
}

【语法说明】
expr:通常表示数据库列名。
ROLLUP|CUBE:GROUP BY子句的扩展，可以返回小计和总计记录。
GROUP BY语句和分组函数一起使用，它可以根据某一列进行分组，也可以根据某几列进行分组。
```
{% endcode %}

`HAVING`子句通常和`GROUP BY`子句一起使用，限制搜索条件。它和`WHERE`子句不一样，`HAVING`子句与组有关，而不与单个的值有关。在`GROUP BY`子句中，它会作用于`GROUP BY`创建的组。

HAVING与WHERE的区别，`HAVING`对`GROUP BY`子句负责而`WHERE`对`FROM`负责。

## 子查询

子查询就是嵌套查询，它是嵌套在另外一个语句中的SELECT语句。WHERE后面的条件不是一个确切的值或表达式，而是另外一个查询语句的查询结果。子查询不仅仅出现在SELECT语句中，也会出现在DELETE和UPDATE语句中，它本质上是WHERE后的一个条件表达式。

子查询允许返回单行数据，也允许返回多行数据。如果返回的是单行数据(不管是普通查询还是分组查询)，那么这是逻辑上最简单的子查询嵌套查询语句。它和在WHERE条件中使用单一或多个条件限制的操作方法一致。

如果子查询返回的值为多行值，那么需要用到IN关键字，此时IN的用法和前面介绍的方式一致。除此之外，也可以使用量化比较关键字SOME、ANY、ALL，这些需要配合<、<=、=、>、>=使用。它们所表示的含义如下:

* ANY:表示满足子查询结果的任何一个。和<、<=搭配，表示小于等于列表中的最大值，而和>、>=配合时表示大于等于列表中的最小值。
* SOME:可以认为和ANY含义相同。
* ALL:表示满足子查询结果的所有结果。和<、<=搭配，表示小于等于列表中的最小值:而和>、>=配合时表示大于等于列表中的最大值。

{% code lineNumbers="true" %}
```sql
[例如]
select
    id,name
from table_name1
where id = [any|some|all] (select id from table_name2)
```
{% endcode %}

## 连接查询

最简单的连接查询是利用逗号完成的，它利用逗号把FROM后的表名隔开，这就构成了最简单的连接查询。笛卡尔积(行 \* 列)

> 内连接

内连接也称为简单连接，它会把两个或多个表进行连接，只能查询出匹配的记录，不匹配的记录将无法查询出来。

> 自连接

所谓自连接，就是把自身表的一个引用作为另一个表来处理，这样就能获取一些特殊的数据。

> 外连接

外连接分为左外连接、右外连接、全外连接。它们所表示的含义如下:

* 左外连接(`LEFT JOIN`):又称为左向外连接。使用左外连接的查询，返回的结果不仅仅是符合连接条件的行记录，还包含了左边表中的全部记录。也就是说，如果左表的某行记录在右表中没有匹配项，则在返回结果中右表的所有选择列表列均为空。
* 右外连接(`RIGHT JOIN`):又称为右向外连接。它与左外连接相反，将右边的表中所有的数据与左表进行匹配，返回的结果除了匹配成功的记录，还包含了右表中未匹配成功的记录，并在其左表对应的列补空值。
* 全外连接(`FULL JOIN`):返回所有匹配成功的记录，并返回左表未匹配成功的记录，也返回右表未匹配成功的记录。

> (+)的使用

在Oracle中使用外连接，有一种比较特殊的表示方法，利用`(+)`表示外连接。虽然这种方式可以实现外连接，但Oracle还是建议开发人员使用OUTER JOIN关键字。

(+)的使用方法读者可以简单记一下，该操作符总是放在非主表的一方，并且需要使用WHERE子句，不能存在OUTER JOIN关键字。

{% code lineNumbers="true" %}
```sql
左外连接
select * from product p left join detail d on p.id = d.id;
(+)左外连接
select * from product p ,detail d on p.id = d.id(+);

右外连接
select * from product p right join detail d on p.id = d.id;
(+)右外连接
select * from product p ,detail d on p.id(+) = d.id;

使用(+)需要注意的地方有:
该操作符只能用在WHERE子句中，并且不能与OUTER JOIN一起使用。
该操作符不能用于全外连接。
如果外连接有多个条件，那么每一个条件都需要使用该操作符。
```
{% endcode %}
