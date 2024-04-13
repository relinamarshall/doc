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

# 视图

视图在Oracle中应用相当普遍，所以也比较重要。视图在数据库中可以理解为一张虚拟的表，使用视图可以补充表结构在某些需求方面的不足,可以让开发人员更方便地查询复杂数据还可以缩短开发周期，节省公司成本。

## 什么是试图

### 认识试图

根据官方的文档可以这样理解视图:它是一个基于一个表或多个表的逻辑表，视图本身不包含任何数据。通俗来说，可以把视图看成是虚拟的表，只是一个查询语句的结果，它的数据最终是从表中获取的，这些表通常称为源表或基表。当基表的数据发生变化时，视图里的数据同样发生变化。通常视图的数据源有下面三种情况:&#x20;

* 单一表的子集。&#x20;
* 多表操作结果集。&#x20;
* 视图的子集。

### 试图的作用

* 使数据简化。在表中很多数据对业务来说是冗余的，这时开发者会使用比较复杂的SQL语句得到自己想要的。实际开发中不能要求每个人都能做到这一点，所以，通常情况下由一个人把该复杂语句做成视图，其他人员直接调用该视图即可。这样对视图使用人员就简化了数据，隐藏了数据的复杂性。
* 使数据更加独立。程序开发时，大多数是程序直接访问数据库的表，当这些表的结构随着业务的变化而不得不重新设计时会影响到程序(通常表一旦设计完成就很难再做修改)，所以可以使得程序直接访问视图。这样视图就可以把程序和数据库的表隔离开来，降低开发者的劳动成本。
* 增加安全性。视图可以查询表指定的列来展现给用户，而不必让使用者完全看见表的所有字段。这种情况很多是一个公司提供给其他合作伙伴查询数据的接口，而视图通常也会设成只读属性。

## 试图的语法

```plsql
CREATE [ OR REPLACE ] [ [ NO ] FORCE ] VIEW
    [ schema. ]view
    [(alias,...) inline_constraint(s))
        [out_of_line_constraint(s)]
AS subquery
[
    WITH{ READ ONLY | CHECK OPTION [ CONSTRAINT constraint ] }
]

【语法说明】
OR REPLACE:表示新建视图可以覆盖同名视图。
[NO]FORCE:即FORCE或NOFORCE，表示是否强制创建视图。
    例如，在基表不存在的情况下就创建视图是有错误的，这时可以用FORCE关键词强制创建视图，然后再创建基表。
    Oracle中NO FORCE是默认值。
[schema.]view:这是视图的所属方案名称和视图本身的名称。
[(alias,...) inline_constraint(s)]:视图字段的别名和内联约束。
[out_of_line_constraint(s):也是约束，是与inline_constraint(s)相反的声明方式。
WITH READ ONLY:设置视图只读，这样的视图具有更高的安全性。
WITH CHECK OPTION[CONSTRAINT constraint]:一旦使用该限制，当对视图增加或修改数据时必须满足子查询的条件。
    也就是说，是把子查询的条件作为一个约束，而constraint是这个约束的名称。
```

## 试图的创建

### 创建单表试图

{% code lineNumbers="true" %}
```plsql
CREATE OR REPLACE VIEW SIMPLE_PRODUCTIFO_VIEW
AS
    SELECT PRODUCTID,PRODUCTNAME,PRODUCTPRICE,CATEGORY ,ORIGIN
    FROM PRODUCTINFO
    WHERE ORIGIN = '中国'
    AND ROWNUM < 6;
```
{% endcode %}

{% hint style="info" %}
根据官方提供的资料，在当前用户下创建视图需要有CREATE VIEW系统权限，这里直接给当前用户赋子了DBA权限。

如果执行成功将会出现如下字样: 视图已创建
{% endhint %}

使用SELECT查询语句可以查看视图的效果，语法同查询表数据一样，只需要把FROM后面换成要查询的视图名称即可。

```plsql
SELECT
    PRODUCTID 产品ID,
    PRODUCTNAME 产品名称,
    PRODUCTPRICE 产品价格, 
    CATEGORY 产品类型编码,
    ORIGIN 产地
FROM SIMPLE_PRODUCTINFO_VIEW;
```

{% hint style="info" %}
可以利用下面的语句查看当前用户下的所有视图:

SELECT VIEW\_NAME FROM USER\_VIEWS;
{% endhint %}

### 创建多表试图

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE VIEW MULTI_PRODUCTIFO_VIEW
AS
    SELECT PT.PRODUCTID,PT.PRODUCTNAME,PT.PRODUCTPRICE,
        PT.CATEGORY ,PT.ORIGIN ,CG.CATEGOYNAME
    FROM PRODUCTINFO PT,CATEGROYINFO CG
    WHERE PT.CATEGORY = CG.CATEGORYID
    AND ORIGIN = '中国'
    AND ROWNUM < 6;
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE VIEW VI_PRODUCINFO_VIEW
AS 
    SELECT PRODUCTID,PRODUCTNAME
    FROM MULTI_PRODUCTINFO_VIEW;
```
{% endcode %}

```sql
CREATE OR REPLACE FORCE VIEW NOTABLE_VIEW AS
SELECT ID,NAME
FROM NOTABLE;
```

{% hint style="info" %}
执行提示:创建的试图有编译错误。

如果出现以上提示，表示该视图已经创建。
{% endhint %}

{% code lineNumbers="true" %}
```plsql
CREATE OR REPlACE VIEW CONST_VIEW
(
    ID,
    --使用inline方式对列PRODUCTNAME创建UNIQUE约束
    NAME CONSTRAINT NAME_UNQ UNIQUE RELY DISABLE NOVALIDATE,
    PRICE,
    --使用out_of_line方式对视图设置主键约束
    --RELY DISABLE NOVALIDATE表示约束对此前和此后的数据都不进行检查，并告知Oracle此视图现在符合这两种约束条件。
    CONSTRAINT VI_PRK PRIMARY KEY(ID) RELY DISABLE NOVALIDATE
)
AS
SELECT ID,NAME,PRICE
FROM PRODUCTINFO
WITH CHECK OPTION;
```
{% endcode %}

{% hint style="info" %}
视图约束比较特殊。它的约束是声明式的，并不能真正作用于视图本身。当创建约束后增加或更新违背约束的数据时，它会执行成功。这种声明实际上是告诉Oracle优化器它的数据是符合约束条件的，但Oracle本身并没有验证数据是否真的符合条件。
{% endhint %}

## 操作试图数据的限制

视图允许做DML操作，但需要注意的地方比较多。因为视图增加或更新数据实际上是在操作视图的源表。除此之外，视图本身可以设置更新限制条件。

### 试图READ ONLY设置

创建视图时为了避免用户修改数据，可以把视图设成只读属性，其操作比较简单

```sql
CREATE OR REPLACE VIEW VIEW_NAME AS
SELECT ID,NAME FROM TABLE_NAME
WITH READ ONLY;
```

{% hint style="info" %}
当插入或修改视图数据时，会提示“无法对只读视图执行DML操作"。
{% endhint %}

### 试图CHECK OPTION设置

在某些情况下允许修改视图的数据，修改数据的本质是修改视图源表的数据。假如某个视图查询出来的是年龄大于20的所有数据，如果为该视图增加一条年龄为10的记录，那么该记录将不会出现在视图中。显然这是不符合逻辑的。为了避免这种情况的发生，可以利用CHECK OPTION选项来设置视图的检查约束。

CHECK OPTION选项表示视图启动了和子查询条件一样的约束。也就是说，如果对视图修改或插入的数据和查询条件不一致，那么该操作会被中止。

* 增加数据\
  新增的数据与试图查询条件不一致，会提示错误信息
* 修改数据\
  修改的数据与试图查询条件不一致，也会提示错误信息
* 删除数据\
  删除的数据与试图查询条件不一致；脚本执行后并没有出现错误提示，但提示0行被操作。这不但说明了视图过滤出来的数据和源表的数据在逻辑上彻底分离，也说明了CHECK OPTION项对删除没有作用(如果有作用，删除语句将无法执行)。

### 试图创建语句对试图操作的影响

如果想要一个可以更新(这里的更新是指增加、删除、修改)的视图，源表应尽量是单表，否则限制比较多。下面的情况一旦出现在视图中，视图就不允许更新。&#x20;

* DISTINCT关键字。&#x20;
* 集合运算或分组函数，如INTERSECT、SUM、MAX、COUNT等函数。&#x20;
* 出现GROUP BY、ORDER BY、MODEL、START WITH等语句。&#x20;
* 出现伪列关键字，如ROWNUM。

除了以上情况外,还需要考虑基表的一些约束,这些约束对视图数据的更新都有一定影响。 如果需要创建可以更新的视图，可以使用INSTEAD OF触发器。

## 试图的修改

视图的修改比较特殊，不能像表一样修改，更准确地说它没有修改选项，可以覆盖原有视图，但这不会影响视图的使用。因为视图本身不包含数据，所以覆盖原有视图时就不存在数据丢失的问题。

当视图创建完成后，可以对其约束进行添加、删除、修改操作。

{% tabs %}
{% tab title="增加试图约束" %}
```sql
ALTER VIEW [schema.]view
ADD [CONSTRAINT constraint name]
{UNIQUE(columnt [,column ]...)
| PRIMARY KEY (column [, colum ]...)
| FOREIGN KEY (column [, colum ]...)
    references_clause
| CHECK (condition)
}
[constraint state]

【语法说明】
ALTER VIEW:表示修改视图的关键词。
ADD[CONSTRAINT constraint_name]:为视图增加一项约束，可以带约束名称。
UNIQUE:唯一约束。
PRIMARY KEY:主键约束
FOREIGN KEY:外键约束。
CHECK:检查约束。
constraint_state:约束声明。
```
{% endtab %}

{% tab title="删除试图约束" %}
```sql
ALTER VIEW VIEW_NAME
DROP CONSTRAINT;
```
{% endtab %}
{% endtabs %}

## 试图的删除

视图删除和表删除操作方式一样，可以使用SQL语句删除，也可以使用PL/SQLDeveloper工具删除。在本节中就将分别讲述使用这两种方式删除视图的方法。

```sql
DROP VIEW [schema.]view [CASCADE CONSTRAINTS]

【语法说明】
CASCADE CONSTRAINTS:删除视图时删除约束。
```

## 小结

在Oracle中经常使用视图，视图可以根据业务的需要从不同的角度展示数据，它给数据库管理者和用户都提供了一定的便捷性，也提高了数据的安全性。视图本身不包含任何数据，它只是一个查询，所有的数据都是从其他表或视图中获取的，但数据有着逻辑独立性。
