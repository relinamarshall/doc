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

# SQL基础

## SQL的种类

从SQL语言的种类来看，由数据库表的创建到给数据库中的对象进行权限管理全部都可以使用SQL语言，下面就按照SQL语言的使用顺序说明每种SQL语言的作用。

### 数据定义语言(DDL)

数据定义语言(Data Definition Language，DDL)正如它字面上的意思，是定义数据库中数据要如何存储的。DDL语言包括对数据库中对象的创建、修改、删除的操作，这些对象主要有数据库、数据表、视图、索引等。

### 数据操作语言(DML)

数据操纵语言(DataManipulationLanguage，DML)也像它字面上的意思，是对数据库表进行操作的。这些操作主要包括对数据库表中的数据进行增加、删除、修改的操作，并且在操作时一次可以把表中数据按条件进行多条或全部的处理，为数据库的使用提供方便。

### 数据查询语言(DQL)

数据查询语言(DataQueryLanguage，DQL)是对数据库表中的数据进行查询的，查询时既可以查询一个表也可以进行多表的查询，并且可以按不同的条件来检索数据，给数据库的查询统计工作带来了更多的便利。

### 数据控制语言(DCL)

数据控制语言(DataControlLanguage，DCL)是对数据库中的对象权限进行权限设置和取消等操作，但是只有数据库的系统管理员才有权力去执行对数据库对象权限的操作。使用DCL可以为数据库中不同的用户设置不同的权限，这样也能够提高数据库的安全性。

## 常用数据类型

在Oracle llg中提供的数据类型有23种，下面介绍常用的数据类型，并把数据类型分为字符型、数字型、日期类型和其他数据类型4类进行讲解。

### 字符型

字符型在Oracle 11g中有varchar2、char、nchar、nvarchar2和long五种，它们在数据库中是以ASCI码的格式存储的。下面用一个表格来讲解每种数据类型的作用



<table><thead><tr><th width="140">数据类型</th><th width="160">取值范围(字节)</th><th>说明</th></tr></thead><tbody><tr><td>varchar2</td><td>0-4000</td><td>可变长度的字符串</td></tr><tr><td>nvarchar2</td><td>0-1000</td><td>用来存储Unicode字符集的变长字符型数据</td></tr><tr><td>char</td><td>0-2000</td><td>用于描述定长的字符型</td></tr><tr><td>nchar</td><td>0-1000</td><td>用来存储Unicode字符集的订长字符型数据</td></tr><tr><td>long</td><td>0-2GB</td><td>用来存储变长的字符串</td></tr></tbody></table>

{% hint style="info" %}
在Oracle 1lg中long类型很少使用，最常使用的字符数据类型就是varchar2。
{% endhint %}

### 数字型

数字型在Oracle 11g中常用的有number和float类型两种，可以用它们来表示整数和小数。&#x20;

<table><thead><tr><th width="146">数据类型</th><th width="248">取值范围(字节)</th><th>说明</th></tr></thead><tbody><tr><td>number(p,s)</td><td>p最大精度是38位(十进制)</td><td>P代表的是精度，s代表的是保留的小数位数:可以用来存储定长的整数和小数</td></tr><tr><td>float</td><td>用来存储126位数据(二进制)</td><td>存储的精度是按二进制计算的，精度范围为二进制的1~126，在转化为十进制时需要乘以 0.30103</td></tr></tbody></table>

### 日期类型

日期类型在Oracle 11g中常用的有date和timestamp两种类型，可以用它们来存放日期和时间。

<table><thead><tr><th width="148">数据类型</th><th>说明</th></tr></thead><tbody><tr><td>date</td><td>用来存储日期和时间，范围在公元前4712年1月1日到公元9999年12月31日</td></tr><tr><td>timestamp</td><td>用来存储日期和时间，与date类型的区别就是在显示日期和时间时更精确，date类型的时间精确到秒，而timestamp的数据类型可以精确到小数秒。此外，使用timestamp存放日期和时间还能够显示当前是上午还是下午</td></tr></tbody></table>

### 其他类型

除了上面讲过的字符型、数字型、日期类型之外，在Oracle11g中还有存放大数据的数据类型以及存放二进制文件的数据类型。

<table><thead><tr><th width="127">数据类型</th><th width="198">取值范围(字节)</th><th>说明</th></tr></thead><tbody><tr><td>blob</td><td>最多可以存放4GB</td><td>存储二进制数据</td></tr><tr><td>clob</td><td>最多可以存放4GB</td><td>存储字符串数据</td></tr><tr><td>bfile</td><td>大小与操作系统有关</td><td>用来把非结构化的二进制数据存储在数据库以外的操作系统文件中</td></tr></tbody></table>

## 数据定义语言(DDL)

DDL主要包括数据库对象的创建(create)、删除(drop)和修改(alter)的操作。

### 创建表CREATE

在DDL语言中第一次使用数据库要用到的就是创建表，创建表使用create table语句完成。具体语法如下:

{% code lineNumbers="true" %}
```sql
CREATE TABLE table_name
(
    column_name datatype [null|not null],
    column_name datatype [null|not null],
    ...
    [constraint]
);

【语法说明】
table_name:在数据库中创建的数据表的名称，在一个数据库中数据表名是不能重复的。
column_name:表中的列名，列名在一个表中也是不能重复的。
datatype:该列存放数据的数据类型。
[null|not nul]:允许该列为空或者不允许该列为空，在创建表时默认为不允许该列为空。
[constraint]:为表中的列设置约束，约束主要包括主键约束、外键约束、检查约束等。
```
{% endcode %}

> 创建商品信息表

| 字段名          | 含义   | 数据类型           |
| ------------ | ---- | -------------- |
| PRODUCTID    | 商品编号 | varchar2(10)   |
| PRODUCTNAME  | 商品名称 | varchar2(20)   |
| PRODUCTPRICE | 商品价格 | number(8,2)    |
| QUANTITY     | 商品数量 | number(10)     |
| CATEGORY     | 商品类型 | varchar2(10)   |
| DESPERATION  | 商品描述 | varchar2(1000) |
| ORIGIN       | 产地   | varchar2(10)   |

{% code lineNumbers="true" %}
```sql
create table productinfo
(
    ProductId varchar2(10),
    ProductName varchar2(20),
    ProductPrice number(8,2),
    Quantity number(10),
    Category varchar2(10),
    Desperation varchar2(1000),
    Origin varchar2(10)
);
```
{% endcode %}

### 修改表ALTER

如果要对已经创建好的表进行修改，那么就需要使用altertable语句来修改。修改表的基本语法如下:

{% code lineNumbers="true" %}
```sql
ALTER TABLE table_name
ADD column_name | MODIFY column_name | Drop COLUMN column_name;

【语法说明】
ADD:用于向表中添加列。
MODIFY:用来修改表中已经存在的列的信息。
DROP COLUMN:删除表中的列，在删除表中的列时经常要加上CASCADECONSTRAINTS，是要把与该列有关的约束也一并删除掉。
```
{% endcode %}

> 修改productinfo商品信息表，向该表中增加一列向表中添加列使用的是ADD子句，向表中增加一列备注remark信息，字段类型是varchar2。

```sql
alter table PRODUCTINFO 
add remark varchar2(200);
```

> 修改productinfo商品信息表，修改列的字段类型修改字段类型需要使用的是MODIFY子句，修改productinfo中刚添加的remark列的字段类。

```sql
alter table PRODUCTINFO 
modify remark number(2,2);
```

> 修改productinfo商品信息表，删除表中的字段删除表中的字段要使用DROP子句，下面就删除productinfo表中的remark字段。

```sql
alter table PRODUCTINFO 
drop column remark;
```

> 修改productinfo商品信息表的多个字段 修改productinfo表中的ProductName字段，把字段的长度修改成25，并添加一个字段remark。

```sql
alter table PRODUCTINFO 
add remark varchar2(200)
modify productname varchar2(25);
```

### 删除表DROP

在使用数据库中的表时经常需要删除一些不需要的表，删除表需要使用DROP TABLE语句来完成。具体语句如下:

```sql
DROP TABLE table_name;
```

删除表的语句是非常简单的，只需要指定要删除的表名，即可删除该表。下面就利用上面删除表的语句完成删除的操作。如果要删除上面创建的productinfo表，只需要下面的语句即可完成:

```sql
 DROP TABLE productinfo;
```

## 约束的使用

约束是保证数据库表中数据的完整性和一致性的手段，Oracle 11g中的5个约束，即主键约束、外键约束、唯一约束、检查约束、非空约束。

### 主键约束

主键约束在每一个数据表中只有一个，但是一个主键约束可以由数据表中多个列组成。

> 使用主键约束创建表

在创建表时就创建主键约束，只需要使用primary key(字段名)即可完成。

{% code lineNumbers="true" %}
```sql
create table categoruinfo
(
    Id varchar2(10),
    Name varchar2(30),
    primary key(Id)
);
```
{% endcode %}

> 修改表添加主键约束

在创建表时如果没有创建主键约束，可以在修改表时为表添加主键约束。添加主键约束的语法如下:

{% code lineNumbers="true" %}
```sql
ALTER TABLE table_name
ADD CONSTRAINTS constraint_name PRIMARY KEY (column_name);

【语法说明】
constraint_name:约束的名称。
column_name:主键约束指定数据表中的列名。
```
{% endcode %}

> &#x20;移除主键约束

如果需要移除表中现有的主键约束，可以使用如下所示的语句完成:

{% code lineNumbers="true" %}
```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint name;

【语法说明】
constraint_name:要移除的约束名称，这个名称可以是在表中任意约束的名称。
```
{% endcode %}

### 外键约束

外键约束可以保证使用外键约束的数据库列与所引用的主键约束的数据列一致，外键约束在一个数据库中可以有多个。

> 使用主键约束创建表

外键约束时建立在两张表中的约束，需要在创建表的语句后面加上如下语句：

{% code lineNumbers="true" %}
```sql
CONSTRAINT constrain_name FOREIGN KEY (column_name)
REFERENCE table_name (column_name)
ON DELETE CASCADE;

【语法说明】
constraint name:创建的外键约束名字。
FOREIGNKEY(column_name):指定外键约束的列名。
REFERENCE:要引用的表名(列名)。
ON DELETE CASCADE:设置级联删除，当主键的字段被删除时，外键所对应的字段也被同时删除。
```
{% endcode %}

> 在修改数据库表时添加外键约束

在已经存在的数据库表中也是可以添加外键约束的。

{% code lineNumbers="true" %}
```sql
ALTER TABLE table_name
ADD CONSTRAINT constraint_name FOREIGN KEY(column_name)
REFERENCE table_name(column_name)
ON DELETE CASCADE;
```
{% endcode %}

> 移除外键约束

移除外键约束与移除主键约束的语法一致

```sql
ALTER TABLE table_name
DROP CONSTRAINT contraint_name;
```

### CHECK约束

CHECK约束是检查约束，能够规定每一个列能够输入的值，以保证数据的正确性。

> 创建表时添加CHECK约束

创建CHECK约束可以设置在“性别”列中只能输入男或者女，在“年龄”列中只能输入18\~30岁的年龄。创建CHECK约束的语句是在创建表的语句后面加上如下语句完成的:

```sql
CONSTRAINT constraint_name CHECK(condition);
```

其中，condition是检查约束的条件，检查约束的条件要建立在具体的字段中。例如，给字段Age设置为18\~30岁，就可以写成age>=18 and age<=30。

> 在修改数据表时添加CHECK约束

在修改数据表时添加检查约束的方法也比较简单，在ALTERTABLE语句的后面添加如下语句即可:

```sql
ALTER TABLE table_name
ADD CONSTRAINT costraint_name (condition);
```

> 移除CHECK约束

```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

### UNIQUE约束

UNIQUE约束称为唯一约束，可以设置在表中输入的字段值都是唯一的，这个约束和之前学习的主键约束非常相似。不同的就是唯一约束在一个表中可以有多个，而主键约束在一个表中只能有一个。下面就详细讲述UNIQUE约束的使用。

> 在创建表时添加UNIQUE约束

在创建表时可以为表中的字段直接添加UNIQUE约束，具体的创建方法是在创建表的语句后面加上下面的语句:

```sql
CONSTRAINT constraint_name UNIQUE(column_name);
```

> 在修改表时添加UNIQUE约束

```sql
ALTER TABLE table_name
ADD CONSTRAINT constraint_name UNIQUE(column_name);
```

> 移除UNIQUE约束

```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

### NOT NULL约束

NOTNULL约束就是非空约束,经常会在创建表时添加非空约束以确保字段必须要输入值。

> 创建NOT NULL约束

```sql
CREATE TABLE table_name(
    column_name varchar2(10) NOT NULL
);
```

> 修改表时设置NOT NULL约束

```sql
ALTER TABLE table_name
MODIFY column_name NOT NULL;
```

> 对于非空约束不需要删除，如果要取消某个列非空的约束，直接使用MODIFY语句把该列的非空约束写成NULL即可。

## 数据操纵语言(DML\DQL)

DML也就是用来操纵数据库中数据所使用的语言，对数据库中的数据操纵无非就是对数据进行增加、删除、修改、查询的操作。对于数据的查询也称为数据查询语言(DQL)。

### 添加数据INSERT

在创建好数据表之后，添加数据是首先要做的工作。在给表中添加数据时要与表中字段类型相匹配，也就是说，如果表中的字段是日期类型，那么在向该字段中添加数据时也要添加日期类型的数据。向表中添加数据的一般语法如下:

{% code lineNumbers="true" %}
```sql
INSERT INTO table_name(column_name1,column_name2,..)
VALUES(data1,data2...);

【语法说明】
column_namel:指定表中要添加数据的列名，可以是1个到多个。
data1:要填入指定列的数据值，这里要求添加值的数目要与列名的数量一致。
```
{% endcode %}

> 通过其他数据表向表中添加数据

如果在数据库中需要新创建一个数据表，但是这个表中的数据又与其他表中的数据有些相似，那么就可以直接把其他表中的数据添加到新创建的数据表中，这样就能减少添加数据的工作量。具体语法如下:

{% code lineNumbers="true" %}
```sql
INSERT INTO table_name1(column_name1,column_name2)
select column_name1,column_name2... FROM table_name2;

【语法说明】
table_name1:目标表的名称，也就是要插入数据的表名。
table_name2:数据的来源表。
```
{% endcode %}

{% hint style="info" %}
在使用来源表向目标表中插入数据时，一定要确保两个表的列的个数和列的数据类型都一致，否则会出现错误。
{% endhint %}

> 建表就直接通过源数据表加数据

{% code lineNumbers="true" %}
```sql
CREATE TABLE table_name AS 
SELECT column_name1,column_name2,... FROM source_table;

【语法说明】
table_name:要新创建的目标表的名称。
source_table:创建目标表时数据的来源表。这里可以指定查询表的字段，也可以用“*”
```
{% endcode %}

### 修改数据UPDATE

修改数据也是经常要使用的,在已经存在数据的表中修改数据使用UPDATE语句即可完成具体语法如下:

{% code lineNumbers="true" %}
```sql
UPDATE table_name 
SET column_name1=data1,column_name2=data2,...
[WHERE condition];

【语法说明】
column_namel:要修改数据列的字段名，可以是一个或多个。
data1:要赋给字段的新值，这个值的数据类型要与数据表中字段的数据类型一致
WHERE:条件，这里如果省略了WHERE语句，那么就意味着要修改表中该字段的所有值，如果加上WHERE语句，那么就可以有选择地修改数据表中的某个字段。
```
{% endcode %}

### 删除数据DELETE

经常要删除数据表中一些没有用的数据，删除数据要使用DELETE关键字来完成。使用它可以根据条件删除指定的数据，也可以删除表中的全部数据。一般的语法如下:

```sql
DELETE FROM table_name [WHERE condition];
```

其中，\[WHERE condition]子句是可以省略的，如果省略了\[WHERE condition]子句，就意味着删除数据表中全部的数据，如果加上了\[WHERE condition]子句就可以根据条件删除表中的数据。这里，删除数据都是指删除数据表中一条记录并不是删除表中某个字段。

### 查询数据SELETE

数据查询语言也称为DQL，在本小节中主要介绍SELECT语句的基本用法。SELECT的一般语法如下:

{% code lineNumbers="true" %}
```sql
SELECT column_name1, column_name2,...
FROM table_name WHERE[condition];

【语法说明】
column_namel:代表的是数据表中的字段名，可以查询数据表中的一个或多个字段,同时可以使用“*”号代替数据表中所有的字段。
WHERE[condition]:代表的是查询的条件，如果不指定查询条件则查询数据表中所有的记录，如果指定查询条件，那么就可以根据查询条件来查询记录了。
```
{% endcode %}

{% hint style="info" %}
如果在实际应用中只需要表中某些列的值，最好是指定出列名来查询，不要使用"\*"号来查询全部的记录，因为查询全部记录会影响查询的效率。
{% endhint %}

### 其他数据操作语句

> TRUNCATE语句

TRUNCATE语句和DELETE语句一样都是用来完成删除数据表中数据的，但是二者是有区别的。使用TRUNCATE语句删除表中的记录都是要把表中的记录全部删除，但是TRUNCATE语句删除表中数据的速度要比使用DELETE语句删除表中的数据更快一点。

```sql
TRUNCATE TABLE table_name;
```

> MERGE语句

MERGE语句与UPDATE语句的功能类似，都是修改数据表中数据的，但是MERGE语句与UPDATE语句也是有区别的。使用MERGE语句可以对数据表同时进行增加和修改的操作。具体语法如下:

{% code lineNumbers="true" %}
```sql
MERGE [INTo] table_name1
USING table_name2
0N (condition)
WHEN MATCHDED THEN merge_udate_clause
WHEN NOT MATCHED THEN merge_insert_clause;

【语法说明】
table_name1:要修改或添加的表,
table_name2:参照的更新的表。
condition:table_name1和table_name2之间的关系，或其他的一些条件。
merge_update_clause:如果和参照表table_name2中的条件匹配，就执行更新操作的SOL语句。
merge_insert_clause:如果条件不匹配，就执行增加操作的SOL语句。
```
{% endcode %}

{% hint style="info" %}
这里merge\_update\_clause和merge\_insert\_clause都是可以省略的，但是在操作时只能省一个，如果两个语句都省略，那么MERGE语句就失去意义了
{% endhint %}

## 数据控制语言(DCL)

数据控制离不开数据库的使用者，数据控制语言主要就是对数据库使用者赋予和撤销访问数据库的权限的设置，主要包括授予权限要使用的语句`GRANT`和收回权限的语句`REVOKE`。

详细请查看[安全管理](an-quan-guan-li.md)章节

