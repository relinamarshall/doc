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

# 数据库基础篇

## **熟悉数据库**

### **什么是数据库**

#### 数据库中的相关术语

在Oracle 11g数据库中每个数据库里面都包含很多对象，主要包括表、视图、存储过程触发器以及约束。

> 表

表，即在数据库中存放数据用的数据表。每一个数据库中都可以包含多张数据表，但是每一个数据表的名字都是不能重复的。表的一行代表一条记录，每一列都有一个列名，列名是唯一的，行与列的交叉点称为字段。

> 视图

视图是数据库中的虚拟表。在视图中存放的是从数据库表中查询出来的记录，使用视图主要是为了方便信息查询，同时也能够缩短查询数据的时间。

> 存储过程

存储过程是由SQL语句和控制流语句组成的语句块。存储过程存储在数据库内，可由应用程序通过存储过程的名称调用执行。存储过程在开发软件时，可以把大量的数据操作放在服务器端的存储过程中，而只返回需要的数据，这样就减少了数据的传输量，速度也可以大大地提高。

> 触发器

触发器是特殊的存储过程，也是由SQL语句和程序控制语句组成的。但是，触发器在数据库中是不需调用而自动执行的。例如，在触发器中可以定义在修改某张表记录后执行触发器中的内容。

> 约束

约束是在数据库中保证数据库里表中数据完整性的手段。在Oracle11g中使用的约束有主键约束、外键约束、唯一约束、检查约束、非空约束5个，其中主键约束和唯一约束都被认为是唯一约束，而外键约束被认为是参照约束。

> 主键(PrimaryKey)约束

主键约束在每个数据表中只能有一个，但是一个主键约束可以由多个列组成，通常把由多个列组成的主键又叫做复合主键或组合主键。主键约束可以保证主键列的数据没有重复值且值不为空，也可以说是唯一地标识表中的一条记录。

> 外键(ForeignKey)约束

外键约束之所以被认为是参照约束，是因为它主要用作把一个表中的数据和另一个表中的数据进行关联，表和表之间的关联是为了保证数据库中数据的完整性，使用外键保证数据的完整性，也叫参照完整性

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>外键</p></figcaption></figure>

在商品信息表中“商品编号”是主键，而在商品类型信息表中“类型编号”是主键，当把商品信息表中的“商品编号”与商品类型信息表中的“类型编号”设置为外键约束后，在商品信息表中的类型信息就可以用商品类型信息表中的类型编号代替。设置完外键约束后，商品信息表中类型字段值必须是在商品类型信息表中存在的，同时当在商品类型信息表中删除一个类型时，如果商品信息表已经使用过该类型，那么商品类型信息表中的数据就无法被删除。

> 唯一(unique)约束

唯一约束和主键约束一样都是设置表中的列不能重复的约束，区别就是一个表中只能有一个主键约束，而却可以有多个唯一约束。通常情况下设置唯一约束的目的就是使非主键列没有重复值。唯一约束与主键约束的另一个区别是如果数据表中的某一列中有空值，那么就不能把这个列设置为主键列，而可以设置成唯一约束。例如，在商品信息表中把商品编号设置成了主键，但是还要保证商品的名称不重名时，就可以把商品名称设置为唯一约束。

> 检查(check)约束

检查约束是用来指定表中列的值的取值范围的。例如，在员工信息表中员工年龄的列，如果要使员工年龄列的值为18\~50，就可以使用检查约束进行设置，当输入的值不在有效范围内时，就会出现错误。这样就保证了数据库中数据的有效性。

> 非空(Not Null)约束

非空约束是用来约束表中的列不允许为空的。例如，在员工信息表中员工身份证号码列，要求员工必须输入时，可以使用非空约束来保证该列不能为空。

#### 数据库设计的完整性

使用数据库约束就是保证数据库完整性的方法。数据库设计的完整性实际上就是为了保证数据的正确性。为了保证数据的正确性，在Oraclel1g中涉及的完整性主要有3个，即实体完整性、区域完整性、参照完整性。

> 实体完整性

实体完整性要求表中的主键字段都不能为空或者重复的值。例如，在学校里每个学生的学号是唯一的，银行卡的卡号也是唯一的，每个人的身份证号码都是唯一的等。

> 区域完整性

区域完整性是保证输入到数据库中的数据是在有效范围内的，可以使用3.1.4小节中讲的检查约束来设置。例如，输入邮箱的字段要求要有@，输入身份证号码要有15位或18位，输入年龄只能是数字，输入姓名不能有字母等。

> 参照完整性

参照完整性可以保证数据库中相关联的表里面数据的正确性，使用用3.1.4小节讲的外键约束就可以保证参照完整性。确保数据表的参照完整性，就可以避免误删和错加数据。例如，学生选课，如果学生已经选修了某门课程，但是管理员错误地把学生选的课程删除了，那么就会造成学生选修了课程但是无法上课，使用参照完整性设计数据表就会避免类似问题的发生。

### 范式(关系型数据库设计准则)

关系型数据库是目前流行和使用广泛的数据库，关系型数据库的设计标准就是数据库的范范式分别有第一范式、第二范式、第三范式。

#### 第一范式--关系型数据库设计的第一步

目前，只要是使用关系型数据库来设计数据库，都能够满足数据库设计的第一范式。第一范式(1NF)就是数据库表中的字段都是单一属性的，不可再分。这个单一属性可以是数据库中任何一种基本数据类型，如整型、字符型、日期型等。只要是关系型数据库都会满足第一范式。例如，一个产品信息表(product)，描述产品信息的字段有产品编号、产品名称、产品数量、产品价格、产品描述，如表3.4所示，那么这个产品信息表就满足第一范式的要求:每一个字段都是不可再分的单一属性。

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>产品信息表</p></figcaption></figure>

#### 第二范式--关系型数据库设计的第二步

第二范式是在第一范式的基础上进一步对关系型数据库进行规范，官方给出第二范式的定义是要求在数据库表中不存在非关键字段对任一候选关键字段的部分的数依赖。意思就是说在第二范式中组合主键(AB)里面的A或者B与其他字段不能存在组合重复。为解决这个问题，通常的做法是不用组合主键，添加一个编号列，作为单一主键即可满足第二范式。如果不想添加编号列，就满足组合主键(AB)里面的A或者B与其他字段不能存在组合重复。例如，设计一个购物信息表，字段包括客户编号、产品名称、产品数量、产品类型、产品价格、客户类型。如果用客户编号和产品名称作为组合主键，那么在组合主键中产品名称和产品类型存在一定关系，是由产品名称决定产品的类型，所以不符合第二范式的要求，如果不按照第二范式的要求设计表，就会出现以下4个问题:

1. 数据冗余 同一个产品由n个顾客购买，“产品类型”就重复"-1次;同一个顾客购买了多件产品，那么就会多次记录顾客的个人信息。
2. 更新异常 若调整了某个产品的类型，数据表中所有行的“产品类型”值都要更新，否则会出现同一个产品不同类型的情况。
3. 插入异常 假设新进了一个产品，暂时还没有人购买。这样，由于没有人购买，产品的名称和类型也无法记录到数据库中。
4. 删除异常 假设一批顾客把已经购买完的商品退货，这些产品信息就从数据表中删除了。但是，与此同时，产品名称和产品类型等信息也被删除了。这样就导致了删除异常。为了消除数据冗余、更新异常、插入异常和删除异常，可以把现有的一个表拆分成3张表:第1张表是产品类型表，表中有产品类型、产品名称。第2张表是客户信息表，表中有客户编号、客户类型。第3张表是产品信息表，表中有产品名称、产品类型、产品价格、产品数量。

#### 第三范式--关系型数据库设计的第三步

第三范式是在第二范式的基础上对数据库设计进行规范，第三范式的要求是数据表中不存在非关键字段对任一候选关键字段的传递函数依赖。所谓传递函数依赖，指的是如果存在A决定B、B决定C的决定关系，则C传递函数依赖于A。因此，满足第三范式的数据库表应该不存在依赖关系，假定员工信息表为employee(员工编号，姓名，年龄，所在部门，部门电话)，使用员工编号作为员工信息的主键，那么就存在决定关系:员工编号就决定了姓名、年龄、所在部门、部门电话这些字段。从上面的关系可以看出，在表中有一个主键，数据表的设计符合第二范式的要求。但是它不符合第三范式的要求，因为存在决定关系:员工编号就决定了所在部门，所在部门又决定了所在部门的电话，那么就存在了传递函数依赖关系，即员工编号决定部门电话，那么也会出现不满足第二范式时的数据冗余和更新、插入、删除异常的情况。为了满足第三范式的要求，必须把员工信息表拆分成如下两个数据表:

员工表:员工编号、姓名、年龄、所在部门

部门表:部门名称、部门电话。

> 除了上面的三种范式以外，还有一种范式经常使用，即鲍依斯-科得范式(BCNF)。它建立在第三范式的基础上，如果数据库表中不存在任何字段对任一候选关键字段的传递函数依赖，那么就符合BCNF范式。

## SQL基础

### SQL的种类

从SQL语言的种类来看，由数据库表的创建到给数据库中的对象进行权限管理全部都可以使用SQL语言，下面就按照SQL语言的使用顺序说明每种SQL语言的作用。

> 数据定义语言(DDL)

数据定义语言(Data Definition Language，DDL)正如它字面上的意思，是定义数据库中数据要如何存储的。DDL语言包括对数据库中对象的创建、修改、删除的操作，这些对象主要有数据库、数据表、视图、索引等。

> 数据操作语言(DML)

数据操纵语言(DataManipulationLanguage，DML)也像它字面上的意思，是对数据库表进行操作的。这些操作主要包括对数据库表中的数据进行增加、删除、修改的操作，并且在操作时一次可以把表中数据按条件进行多条或全部的处理，为数据库的使用提供方便。

> 数据查询语言(DQL)

数据查询语言(DataQueryLanguage，DQL)是对数据库表中的数据进行查询的，查询时既可以查询一个表也可以进行多表的查询，并且可以按不同的条件来检索数据，给数据库的查询统计工作带来了更多的便利。

> 数据控制语言(DCL)

数据控制语言(DataControlLanguage，DCL)是对数据库中的对象权限进行权限设置和取消等操作，但是只有数据库的系统管理员才有权力去执行对数据库对象权限的操作。使用DCL可以为数据库中不同的用户设置不同的权限，这样也能够提高数据库的安全性。

### 常用数据类型

在Oracle llg中提供的数据类型有23种，下面介绍常用的数据类型，并把数据类型分为字符型、数字型、日期类型和其他数据类型4类进行讲解。

> 字符型

字符型在Oracle 11g中有varchar2、char、nchar、nvarchar2和long五种，它们在数据库中是以ASCI码的格式存储的。下面用一个表格来讲解每种数据类型的作用



| 数据类型      | 取值范围(字节) | 说明                     |
| --------- | -------- | ---------------------- |
| varchar2  | 0-4000   | 可变长度的字符串               |
| nvarchar2 | 0-1000   | 用来存储Unicode字符集的变长字符型数据 |
| char      | 0-2000   | 用于描述定长的字符型             |
| nchar     | 0-1000   | 用来存储Unicode字符集的订长字符型数据 |
| long      | 0-2GB    | 用来存储变长的字符串             |

{% hint style="info" %}
在Oracle 1lg中long类型很少使用，最常使用的字符数据类型就是varchar2。
{% endhint %}

> 数字型

数字型在Oracle 11g中常用的有number和float类型两种，可以用它们来表示整数和小数。&#x20;

| 数据类型        | 取值范围(字节)        | 说明                                                 |
| ----------- | --------------- | -------------------------------------------------- |
| number(p,s) | p最大精度是38位(十进制)  | P代表的是精度，s代表的是保留的小数位数:可以用来存储定长的整数和小数                |
| float       | 用来存储126位数据(二进制) | 存储的精度是按二进制计算的，精度范围为二进制的1\~126，在转化为十进制时需要乘以 0.30103 |

> 日期类型

日期类型在Oracle 11g中常用的有date和timestamp两种类型，可以用它们来存放日期和时间。

<table><thead><tr><th width="175">数据类型</th><th>说明</th></tr></thead><tbody><tr><td>date</td><td>用来存储日期和时间，范围在公元前4712年1月1日到公元9999年12月31日</td></tr><tr><td>timestamp</td><td>用来存储日期和时间，与date类型的区别就是在显示日期和时间时更精确，date类型的时间精确到秒，而timestamp的数据类型可以精确到小数秒。此外，使用timestamp存放日期和时间还能够显示当前是上午还是下午</td></tr></tbody></table>

> 其他类型

除了上面讲过的字符型、数字型、日期类型之外，在Oracle11g中还有存放大数据的数据类型以及存放二进制文件的数据类型。

| 数据类型  | 取值范围(字节)  | 说明                            |
| ----- | --------- | ----------------------------- |
| blob  | 最多可以存放4GB | 存储二进制数据                       |
| clob  | 最多可以存放4GB | 存储字符串数据                       |
| bfile | 大小与操作系统有关 | 用来把非结构化的二进制数据存储在数据库以外的操作系统文件中 |

### 数据定义语言(DDL)

DDL主要包括数据库对象的创建(create)、删除(drop)和修改(alter)的操作。

#### 使用Create语句创建表

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

#### 使用Alter语句修改表

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

#### 使用Drop语句删除表

在使用数据库中的表时经常需要删除一些不需要的表，删除表需要使用DROP TABLE语句来完成。具体语句如下:

```sql
DROP TABLE table_name;
```

删除表的语句是非常简单的，只需要指定要删除的表名，即可删除该表。下面就利用上面删除表的语句完成删除的操作。如果要删除上面创建的productinfo表，只需要下面的语句即可完成:

```sql
 DROP TABLE productinfo;
```

### 约束的使用

约束是保证数据库表中数据的完整性和一致性的手段，Oracle 11g中的5个约束，即主键约束、外键约束、唯一约束、检查约束、非空约束。

#### 主键约束

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

#### 外键约束

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

#### CHECK约束

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

#### UNIQUE约束

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

#### NOT NULL约束

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

### 数据操纵语言(DML\DQL)

DML也就是用来操纵数据库中数据所使用的语言，对数据库中的数据操纵无非就是对数据进行增加、删除、修改、查询的操作。对于数据的查询也称为数据查询语言(DQL)。

#### 添加数据就用INSERT

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

#### 修改数据就用UPDATE

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

#### 删除数据就用DELETE

经常要删除数据表中一些没有用的数据，删除数据要使用DELETE关键字来完成。使用它可以根据条件删除指定的数据，也可以删除表中的全部数据。一般的语法如下:

```sql
DELETE FROM table_name [WHERE condition];
```

其中，\[WHERE condition]子句是可以省略的，如果省略了\[WHERE condition]子句，就意味着删除数据表中全部的数据，如果加上了\[WHERE condition]子句就可以根据条件删除表中的数据。这里，删除数据都是指删除数据表中一条记录并不是删除表中某个字段。

#### 查询数据就用SELETE

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

#### 其他数据操作语句

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

### 数据控制语言(DCL)

数据控制离不开数据库的使用者，数据控制语言主要就是对数据库使用者赋予和撤销访问数据库的权限的设置，主要包括授予权限要使用的语句GRANT和收回权限的语句REVOKE。

详细请查看数据库管理篇-与数据库安全性有关的对象章节

## SELECT检索数据

### SELECT语法结构

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

### select\_list具体语法

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

### order\_by\_clasuse基本语法

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

### WHERE检索条件

WHERE条件子句中可以使用的操作符主要有关系操作符、比较操作符和逻辑操作符

1. 关系操作符包括:\
   <、<=、>、>=、=、!=、<>。
2.  比较操作符包括:

    IS NULL:如果操作数为NULL返回TRUE LIKE:模糊比较字符串值。 BETWEEN...AND.:验证值是否在范围之内。 IN:验证操作数在设定的一系列值中。
3. 逻辑操作符包括: \
   AND:两个条件都必须得到满足。 OR:只要满足两个条件中其中的一个。 NOT:与某个逻辑值取反。

### GROUP BY和HAVING子句

{% code lineNumbers="true" %}
```sql
GROUP BY
{ expr | {ROLLUP | CUBE} {{expr [, expr]...}}
}

【语法说明】
expr:通常表示数据库列名。
ROLLUP|CUBE:GROUPBY子句的扩展，可以返回小计和总计记录。
GROUPBY语句和分组函数一起使用，它可以根据某一列进行分组，也可以根据某几列进行分组。
```
{% endcode %}

HAVING子句通常和GROUPBY子句一起使用，限制搜索条件。它和WHERE子句不一样，HAVING子句与组有关，而不与单个的值有关。在GROUPBY子句中，它会作用于GROUPBY创建的组。

HAVING与WHERE的区别，HAVING对GROUPBY子句负责而WHERE对FROM负责。

### 子查询

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

### 连接查询

最简单的连接查询是利用逗号完成的，它利用逗号把FROM后的表名隔开，这就构成了最简单的连接查询。笛卡尔积(行 \* 列)

> 内连接

内连接也称为简单连接，它会把两个或多个表进行连接，只能查询出匹配的记录，不匹配的记录将无法查询出来。

> 自连接

所谓自连接，就是把自身表的一个引用作为另一个表来处理，这样就能获取一些特殊的数据。

> 外连接

外连接分为左外连接、右外连接、全外连接。它们所表示的含义如下:

* 左外连接(LEFT JOIN):又称为左向外连接。使用左外连接的查询，返回的结果不仅仅是符合连接条件的行记录，还包含了左边表中的全部记录。也就是说，如果左表的某行记录在右表中没有匹配项，则在返回结果中右表的所有选择列表列均为空。
* 右外连接(RIGHT JOIN):又称为右向外连接。它与左外连接相反，将右边的表中所有的数据与左表进行匹配，返回的结果除了匹配成功的记录，还包含了右表中未匹配成功的记录，并在其左表对应的列补空值。
* 全外连接(FULL JOIN):返回所有匹配成功的记录，并返回左表未匹配成功的记录，也返回右表未匹配成功的记录。

> (+)的使用

在Oracle中使用外连接，有一种比较特殊的表示方法，利用“(+)”表示外连接。虽然这种方式可以实现外连接，但Oracle还是建议开发人员使用OUTER JOIN关键字。

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
该操作符只能用在WHERE子句中，并且不能与OUTERJOIN一起使用。
该操作符不能用于全外连接。
如果外连接有多个条件，那么每一个条件都需要使用该操作符。
```
{% endcode %}

## Oracle内置函数

Oracle有多种内置函数，本章将重点介绍其中的两种，它们分别是单行函数和集合函数。这两种类型的函数使用频率比较高。

单行函数是指当查询表或视图时每行都能返回一个结果，可用于SELECT、WHERE ORDER BY等子句中。而集合函数是作用在多行记录上返回一个结果，可用于带GROUPBY或HAVING子句的查询中。单行函数数量比较多，这里介绍其中常用的几种类型，它们分别是数值型函数、字符型函数、日期型函数、转换函数等。

介绍函数之前先简单介绍一下Oracle的DUAL表。该表是Oracle中真实存在的一个表，任何用户都可以读取，多数情况下可以用在没有目标的SELECT查询语句中。它本身只包含了一个DUMMY字段。DUAL表对Oracle很重要，用户不要试图删除该表，一旦删除，Oracle将无法启动。下面的函数讲解中会以DUAL表作为测试语句的目标表。

### 数值型函数

#### 绝对值、取余、判断数值正负函数

> ABS(n)函数

用于返回绝对值。该函数输入一个参数，参数类型为数值型，假如参数为可以隐式转换成数值类型，那么也可以。示例脚本如下:

```sql
SELECT ABS(100),ABS(-100),ABS('100') FROM DUAL

ABS(100)|ABS(-100)|ABS('100')|
--------+---------+----------+
     100|      100|       100|
```

> MOD(n2,n1)函数

该函数表示返回n2除以n1的余数。参数为任意数值或可以隐式转成数值的类型。如果n1为0，那么该函数将返回n2。示例脚本如下:

```sql
SELECT MOD(5,2),MOD(8/3,5),MOD('10',5),MOD(-10,6),MOD(1,0) FROM DUAL

MOD(5,2)|MOD(8/3,5)                              |MOD('10',5)|MOD(-10,6)|MOD(1,0)|
--------+----------------------------------------+-----------+----------+--------+
       1|2.66666666666666666666666666666666666667|          0|        -4|       1|
```

> SIGN(n)函数

返回参数n的符号。正数返回1，0返回0，负数返回-1。但如果n为BINARY\_FLOAT或BINARY DOUBLE类型时，n>=0或者n=NaN函数会返回1。示例脚本如下:

```sql
SELECT SIGN('9'),SIGN(-9),SIGN(0.00),SIGN(-2*'9') FROM DUAL;

SIGN('9')|SIGN(-9)|SIGN(0.00)|SIGN(-2*'9')|
---------+--------+----------+------------+
        1|      -1|         0|          -1|
```

#### 三角函数

> COS(n)函数。用于返回参数n的余弦，n为弧度表示的角度。示例脚本如下

```sql
SELECT COS(3.1415926),COS('3.1415926') FROM DUAL;

COS(3.1415926)                           |COS('3.1415926')                         |
-----------------------------------------+-----------------------------------------+
-0.99999999999999856406703032941211807559|-0.99999999999999856406703032941211807559|

与此类函数类似的还有如下几个。
ACOS(n):返回n的反余弦值。
COSH(n):返回n的双曲余弦值。
SIN(n):返回n的正弦值。
SINH(n):返回n的双曲正弦值。
ASIN(n):返回n的反正弦值。
TAN(n):返回n的正切值。
TANH(n):返回n的双曲正切值。
ATAN(n):返回n的反正切值。
```

#### 返回以指定数值为准整数的函数

> CEIL(n)函数

其返回结果是大于等于输入参数的最小整数。该输入参数要求是十进制数值类型，或可以隐式地转换成数值的类型，可以是非整数。示例脚本如下:

```sql
SELECT CEIL(10),CEIL('10.5'),CEIL(-10.2) FROM DUAL

CEIL(10)|CEIL('10.5')|CEIL(-10.2)|
--------+------------+-----------+
      10|          11|        -10|
```

> FLOOR(n)函数

其返回结果是小于或等于参数的最大整数。该函数输入参数要求是十进制数值类型，或可以隐式地转换成数值的类型。可以是非整数。同CEI函数相反。示例脚本如下:

```sql
SELECT FLOOR(10),FLOOR('10.5'),FLOOR(-10.2) FROM DUAL;

FLOOR(10)|FLOOR('10.5')|FLOOR(-10.2)|
---------+-------------+------------+
       10|           10|         -11|
```

#### 指数、对数函数

> SQRT(n)函数

该函数返回n的平方根。n为数字类型的时候不能为负数，将返回一个实数，当n为BINARY\_FLOAT或BINARY\_DOUBLE类型时，n<0将返回Nan。示例脚本如下:

```sql
SELECT SQRT(100),SQRT('53.9') FROM DUAL

SQRT(100)|SQRT('53.9')                            |
---------+----------------------------------------+
       10|7.34166193719106082894017459575956318933|
```

> POWER(n2.n1)函数

利用该函数可以得到n2的n1次幂的结果。这两个参数为任意数值，但如果n2为负数，那么n1必须为整数。示例脚本如下:

```sql
SELECT POWER(5,2),POWER('5',2),POWER(5.5,2.5),POWER(-5,2),5*5 FROM DUAL;

POWER(5,2)|POWER('5',2)|POWER(5.5,2.5)                           |POWER(-5,2)|5*5|
----------+------------+-----------------------------------------+-----------+---+
        25|          25|70.94253836732937201280515546736005249344|         25| 25|
```

与其相近的函数有:EXP(n)函数，表示返回e的n次幂，e为数学常量e=2.71828183...。

> LOG(n1,n2)函数

该函数可以返回以n1为底n2的对数，n1是除1和0以外的任意正数n2为正数。示例脚本如下:

```sql
SELECT LOG(10,100),LOG(10.5,'100'),POWER(10,2) FROM DUAL;

LOG(10,100)|LOG(10.5,'100')                         |POWER(10,2)|
-----------+----------------------------------------+-----------+
          2|1.95850074204804826547297135207244247989|        100|
```

与其相近的函数有: LN(n)函数，表示返回n的自然对数。n要求大于0。

#### 四舍五入截取函数

> ROUND(for number)函数

该函数的具体原型是ROUND(n,integer)。它将数值n四舍五入成第二个参数指定的形式的十进制数。参数integer要求是整数，如果不是整数，那么它将被自动截取为整数部分。当integer为正整数时，表示n被四舍五入为integer位小数。如果该参数为负数，则n被四舍五入至小数点向左integer位。示例脚本如下:

```sql
SELECT ROUND(10023456,4),ROUND(100.23456,2.56),ROUND(155.23456,-2)FROM DUAL;

ROUND(10023456,4)|ROUND(100.23456,2.56)|ROUND(155.23456,-2)|
-----------------+---------------------+-------------------+
         10023456|               100.23|                200|
```

> TRUNC(for number)函数

该函数的具体原型是TRUNC(n,integer)。它把数值n根据integer的值进行截取，截取时和integer的正负有关。参数integer要求是整数，如果不是整数那么它将被自动截取为整数部分。当integer为正整数时，表示n将截取到integer位小数;如果 integer为负数，则截取到小数点左第integer位，被截取部分用0代替。示例脚本如下:

```sql
SELECT TRUNC(100.23456,4),TRUNC(100.23456,2.56),TRUNC(155.23456,-2),TRUNC(155.23456)FROM DUAL;

TRUNC(100.23456,4)|TRUNC(100.23456,2.56)|TRUNC(155.23456,-2)|TRUNC(155.23456)|
------------------+---------------------+-------------------+----------------+
          100.2345|               100.23|                100|             155|
```

### 字符型函数

#### ASCI码与字符转换函数

> CHR(n\[USING NCHAR\_CS])函数

根据相应的字符集，把给定的ASCI 码转换为字符。USINGNCHAR\_CS指明字符集。以下示例用默认字符集，示例脚本如下:

```sql
SELECT CHR(65)||CHR(66)||CHR(67)ABC,CHR(54678) FROM DUAL;

ABC|CHR(54678)|
---+----------+
ABC|Ֆ         |
```

> ASCII(char)函数

返回参数首字母的ASCII码值。与CHR数相反。参数char的类型可以是CHAR、VARCHAR2、NCHAR或NVARCHAR2。该返回值总是以用户使用的字符集为基础的，如果用户的数据库字符集是7位的ASCI值，那就得到一ASCI码值。示例脚本如下:

```sql
SELECT ASCII('明'),ASCII('Adb'),ASCII('ABC') FROM DUAL;

ASCII('明')|ASCII('ADB')|ASCII('ABC')|
----------+------------+------------+
  15112334|          65|          65|
```

#### 获取字符串长度函数

> LENGTH函数

该函数可以得到指定字符串的长度，返回类型是数字。同样的，LENGTH函数也具有扩展形式，具体结构是{\[LENGTH]I\[LENGTHB]I\[LENGTHC]I\[LENGTH2]I\[LENGTH4]}(char)，其中char是参数。具体的示例脚本如下，这里仅以LENGTH操作为例。

* LENGTH:以字符为单位。
* LENGTHB:以字节为单位。
* LENGTHC:以unicode字符为单位。
* LENGTH2:以UCS2代码点为单位。
* LENGTH4:以UCS4代码点为单位。
* char:字符串参数。

```sql
select LENGTH('ABCDE我FGHI') from dual;

LENGTH('ABCDE我FGHI')|
--------------------+
                  10|
```

#### 字符串截取函数

> SUBSTR函数

该函数提供截取字符串的功能，而且该函数有很多的扩展形式，其具体语句结构是(\[SUBSTR]I\[SUBSTRB]I\[SUBSTRC]I\[SUBSTR2]I \[SUBSTR4]}(char,position\[,substring\_length])。各参数表示含义如下:

* SUBSTR:以字符为单位。
* SUBSTRB:以字节为单位。
* SUBSTRC:以unicode字符为单位。
* SUBSTR2:以UCS2代码点为单位。
* SUBSTR4:以UCS4代码点为单位。
* char:原始字符串。
* position:要截取字符串的开始位置。初始为1，如果该值为负数，则表示从char的右边算起。
* substring\_length:截取的长度。

```sql
SELECT SUBSTR('ABCDE我FGHI',5,2),SUBSTR('ABCDE我FGHI',-5,2)FROM DUAL;

SUBSTR('ABCDE我FGHI',5,2)|SUBSTR('ABCDE我FGHI',-5,2)|
------------------------+-------------------------+
E我                      |我F                       |
```

#### 字符串连接函数

> CONCAT(char1.char2)函数

该函数连接两个参数并返回。char2将连接到char1的尾部。效果和连接符“11”相似。参数类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2、CLOB、NCLOB。示例脚本如下:

```sql
SELECT CONCAT('我的'，'测试!'),'我的'||'测试!' FROM DUAL;

CONCAT('我的'，'测试!')|'我的'||'测试!'|
------------------+-----------+
我的测试!             |我的测试!      |
```

#### 字符串搜索函数

> INSTR函数

该函数可以让我们在指定的字符串中搜索是否存在另一个字符串，其具体语句结构是{\[INSTR]\[INSTRB]I\[INSTRC]I\[INSTR2]IINSTR4]}(string,substring\[,position\[,occurrence]])。该函数也具有扩展形式，各项参数表示含义如下:

* INSTR:以字符为单位。
* INSTRB:以字节为单位。
* INSTRC:以unicode字符为单位。
* INSTR2:以UCS2代码点为单位。
* INSTR4:以UCS4代码点为单位。
* string:待搜索的字符串。
* substring:要搜索的字符串。
* position:搜索的开始位置，默认为1，表示字符串左边第一个位置;如果为负数，则表示字符串的右边位置为起始位置。
* occurrence:substring第几次出现，默认是1。

具体的示例脚本如下，这里仅以INSTR操作为例。

```sql
SELECT INSTR('this is a 测试!','测'),INSTR('this is a 测试:','S',-1) FROM DUAL;

INSTR('THISISA测试!','测')|INSTR('THISISA测试:','S',-1)|
-----------------------+--------------------------+
                     11|                         0|
```

#### 字母大小写转换函数

> UPPER(char)函数。

该函数将指定的参数全部转换成大写字母。参数类型可以是CHARVARCHAR2、NCHAR、NVARCHAR2、CLOB、NCLOB。示例脚本如下:

```sql
SELECT UPPER('c'),UPPER('abcd'),UPPER('this is a test') FROM DUAL;

UPPER('C')|UPPER('ABCD')|UPPER('THISISATEST')|
----------+-------------+--------------------+
C         |ABCD         |THIS IS A TEST      |
```

> LOWER(char)函数

该函数将指定的参数全部转换成小写字母。参数类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2、CLOB、NCLOB。示例脚本如下:

```sql
SELECT LOWER('A'),LOWER('ABCD'),LOWER('THIS IS A TEST') FROM DUAL;

LOWER('A')|LOWER('ABCD')|LOWER('THISISATEST')|
----------+-------------+--------------------+
a         |abcd         |this is a test      |
```

> INITCAP(char)函数

该函数参数的所有单词首字母转换成大写字母。参数类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2.示例脚本如下:

```sql
SELECT INITCAP('this is a test')FROM DUAL;

INITCAP('THISISATEST')|
----------------------+
This Is A Test        |
```

#### 带排序参数的字母大小写转换函数

**>**NLS\_INITCAP(char\[,nlsparam])函数

将指定参数的第一个字母转换成大写。nlsparam参数为可选参数，其设置可以到NLS\_DATABASE\_PARAMETERS表中查询。这两个参数类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2。如果该函数没有nlsparam参数，则它和INITCAP函数一样。示例脚本如下:

```sql
SELECT NLS_INITCAP('a test'),NLS_INITCAP('my test', 'NLS_SORT=SCHINESE_STROKE_M') FROM DUAL;

NLS_INITCAP('ATEST')|NLS_INITCAP('MYTEST','NLS_SORT=SCHINESE_STROKE_M')|
--------------------+--------------------------------------------------+
A Test              |My Test                                           |
```

其中'NLS\_SORT=SCHINESE\_STROKE\_M'指按笔画、部首排序。

> NLS\_UPPER(char\[,nlsparam])函数

将指定参数变成大写。nlsparam参数同NLSINITCAP函数设置。示例脚本如下:

```sql
SELECT NLS_UPPER('this is a test','NLS_SORT= SCHINESE_PINYIN_M')FROM DUAL;

NLS_UPPER('THISISATEST','NLS_SORT=SCHINESE_PINYIN_M')|
-----------------------------------------------------+
THIS IS A TEST                                       |
```

参数中'NLS SORT=SCHINESEPINYINM'表示按拼音排序。

> NLS\_LOWER(char\[,nlsparam])函数

将指定参数转换成小写。nlsparam参数同NLS INITCAP的数设置。示例脚本如下:

```sql
SELECT NLS_LOWER('ABC','NLS_SORT=XGerman')，NLS_LOWER('THIS IS A TEST','NLS_SORT= XGerman')FROM DUAL

NLS_LOWER('ABC','NLS_SORT=XGERMAN')|NLS_LOWER('THISISATEST','NLS_SORT=XGERMAN')|
-----------------------------------+-------------------------------------------+
abc                                |this is a test                             |
```

#### 为指定参数排序函数

> NLSSORT(char\[,nlsparam])函数

根据nlsparam指定的方式对char进行排序。示例脚本如下

```sql
SELECT * FROM PRODUCTINFO
ORDER BY NLSSORT(PRODUCTNAME,'NLS_SORT=SCHINESE_PINYIN_M')

PRODUCTID|PRODUCTNAME|PRODUCTPRICE|QUANTITY|CATEGORY|DESPERATION|ORIGIN|REMARK|
---------+-----------+------------+--------+--------+-----------+------+------+
2        |阿里         |           3|       4|5       |6          |7     |8     |
1        |周稳         |           3|       4|5       |6          |7     |8     |
2        |周周         |           3|       4|5       |6          |7     |8     |
```

#### 替换字符串函数

> REPLACE 函数

函数具体语法结构是REPLACE(char,search\_string\[,replacement\_string]),是一个替换字符串的函数。函数中有三个参数，具体代表的含义如下:

* char:表示搜索的目标字符串。
* search\_string:在目标字符串中要搜索的字符串。
* replacement\_string:该参数可选，用它可替代被搜索到的字符串，如果该参数不用，则表示从char参数中删除search\_string字符串。

具体的示例脚本如下:

```sql
SELECT REPLACE('this is atest','tes','resul') FROM DUAL;

REPLACE('THISISATEST','TES','RESUL')|
------------------------------------+
this is aresult                     |
```

#### 字符串填充函数

> RPAD函数

函数具体语法结构是RPAD(expr1,n\[,expr2])，该函数功能是在字符串expr1的右边用字符串expr2填充，直到整个字符串长度为n时为止。如果expr2不存在，则以空格填充。具体的示例脚本如下:

```sql
SELECT RPAD( 'test' ,8,'*rpad'),RPAD( 'test',15,'*rpad'),RPAD( 'test',4,'rpad') FROM DUAL;

RPAD('TEST',8,'*RPAD')|RPAD('TEST',15,'*RPAD')|RPAD('TEST',4,'RPAD')|
----------------------+-----------------------+---------------------+
test*rpa              |test*rpad*rpad*        |test                 |
```

> LPAD函数

函数具体语法结构是LPAD(expr1,n\[,expr2])，该函数功能是在字符串expr1的左边用字符串expr2填充，直到整个字符串长度为n时为止。如果expr2不存在，则以空格填充。具体的示例脚本如下:

```sql
SELECT LPAD( 'test' ,8,'*rpad' ),LPAD( 'test',15,'*rpad'),LPAD( 'test',4,'*rpad')FROM DUAL;

LPAD('TEST',8,'*RPAD')|LPAD('TEST',15,'*RPAD')|LPAD('TEST',4,'*RPAD')|
----------------------+-----------------------+----------------------+
*rpatest              |*rpad*rpad*test        |test                  |
```

> 需要注意的是，cxpr2总是从左到右填充，可以见示例中LPAD(test,8,\*rpad)部分的执行效果

#### 删除字符串首尾指定字符的函数

> TRIM函数

该函数将删除指定的前缀或尾随的字符，默认删除空格。其具体语法结构是TRIM(\[LEADING | TRAILING | BOTH]\[trim character FROM]trim\_source)，各参数介绍如下

* LEADING:删除trim\_source的前缀字符。
* TRAILING:删除trim\_source的后缓字符
* BOTH:删除trim\_source的前缀和后缀字符。
* trim\_character:删除的指定字符，默认删除空格，
* tim\_source:被操作的字符串。

具体的示例脚本如下:

```sql
SELECT TRIM(TRAILING 't' FROM 'test'),TRIM('test ') FROM DUAL;

TRIM(TRAILING'T'FROM'TEST')|TRIM('TEST')|
---------------------------+------------+
tes                        |test        |
```

> RTRIM(char\[,set])函数

与RPAD函数相反，该函数会提供将char右边出现在set中的字符删除掉。如果set没有，则默认删除空格。具体的示例脚本如下:

```sql
SELECT RTRIM('test'), RTRIM('test*ffs','fs*')FROM DUAL;

RTRIM('TEST')|RTRIM('TEST*FFS','FS*')|
-------------+-----------------------+
test         |test                   |
```

> LTRIM(char\[,set])函数

与RTRIM函数相似，该函数会提供将char左边出现在set中的字符删除掉。如果set没有，则默认删除空格。具体的示例脚本如下:

```sql
SELECT LTRIM('ftest','f')FROM DUAL;

LTRIM('FTEST','F')|
------------------+
test              |
```

#### 字符集名称和ID互换函数

> NLS\_CHARSET\_ID(string)函数

该函数可以得到字符集名称对应ID。string表示字符集名称。示例脚本如下:

```sql
SELECT NLS_CHARSET_ID('US7ASCII') FROM DUAL;

NLS_CHARSET_ID('US7ASCII')|
--------------------------+
                         1|
```

> NLS\_CHARSETNAME(number)函数

该函数可以根据字符集ID得到对应名称。number表示字符集ID。示例脚本如下:

```sql
SELECT NLS_CHARSET_NAME(1) FROM DUAL

NLS_CHARSET_NAME(1)|
-------------------+
US7ASCII           |
```

### 日期型函数

日期类型的函数操作日期、时间类型的相关数据，并返回日期或数字类型的数据。

```sql
yy      two digits      两位年              
yyy     three digits    三位年              
yyyy    four digits     四位年              
mm      number          两位月              
mon     abbreviated     字符集表示      英文版,显示nov
month   spelled out     字符集表示      英文版,显示november
dd      number          当月第几天      
ddd     number          当年第几天      
dy      abbreviated     当周第几天简写   星期五,英文版显示fri
day     spelled out     当周第几天全写   星期五,英文版显示friday 
hh      two digits      12小时进制      
hh24    two digits      24小时进制      
mi      two digits      60进制         
ss      two digits      60进制         
Q       digit           季度           
WW      digit           当年第几周      
W       digit           当月第几周      
```

#### 系统日期、时间函数

> SYSDATE函数

该函数没有参数，可以得到系统的当前日期，是很常用的函数。示例脚本如下:

```sql
SELECT TO_CHAR(SYSDATE,'YYYY-MM-DD HH24:MI:SS') FROM DUAL;

TO_CHAR(SYSDATE,'YYYY-MM-DDHH24:MI:SS')|
---------------------------------------+
2024-03-16 13:59:05                    |
```

> SYSTIMESTAMP函数

该函数没有参数，返回系统时间，该时间包含时区信息，精确到微秒。返回类型为带时区信息的TIMESTAMP类型。示例脚本如下:

```sql
SELECT SYSTIMESTAMP FROM DUAL;

SYSTIMESTAMP                 |
-----------------------------+
2024-03-16 14:00:28.488 +0800|
```

{% hint style="info" %}
该函数可以用于返回远端数据库服务器的时间。
{% endhint %}

#### 得到数据库时区函数

> DBTIMEZONE函数

该函数没有参数，返回数据库时区。示例脚本如下：

```sql
SELECT DBTIMEZONE FROM DUAL;

DBTIMEZONE|
----------+
+00:00    |
```

#### 为日期加上指定月份函数

> ADD\_MONTHS(date,integer)函数

该函数将返回在指定的日期上加一个月份数后的日期。个参数具体含义如下：

* date:指定的日期。
* integer:要加的月份数，该值如果为负数，则表示减去的月份数。

该函数有些地方需要注意，当指定的日期是月的最后一天时，最后函数返回的结果也将是新月的最后一天。而如果新的月份比指定日期月份的天数少，则函数将自动回调有效日期。示例脚本如下:

```sql
SELECT
	TO_CHAR(ADD_MONTHS(TO_DATE('2009-09-15','YYYY-MM-DD'), 1),'YYYY-MM-DD') AS "2009-09-15",
	TO_CHAR(ADD_MONTHS(TO_DATE('2024-02-29','YYYY-MM-DD'), -12),'YYYY-MM-DD')  AS "2024-02-29",
	TO_CHAR(ADD_MONTHS(TO_DATE('2023-02-28','YYYY-MM-DD'), 12),'YYYY-MM-DD')  AS "2023-02-28"
FROM DUAL;

2009-09-15|2024-02-29|2023-02-28|
----------+----------+----------+
2009-10-15|2023-02-28|2024-02-29|
```

> SESSIONTIMEZONE函数

该函数没有参数，可以返回当前会话的时区。示例脚本如下:

```sql
SELECT SESSIONTIMEZONE FROM DUAL;

SESSIONTIMEZONE|
---------------+
Asia/Shanghai  |
```

#### 返回指定月份最后一天函数

> LAST\_DAY(date)函数

该函数返回参数指定日期对应月份的最后一天。示例脚本如下：

```sql
SELECT LAST_DAY(SYSDATE) FROM DUAL;

LAST_DAY(SYSDATE)      |
-----------------------+
2024-03-31 14:16:38.000|
```

#### 返回指定日期后一周的日期函数

> NEXT\_DAY(date,char)函数

该函数返回当前日期向后的一周char的对应日期，char表示的是星期几，全程和缩写都允许。但必须有效，示例脚本如下：

```sql
SELECT SYSDATE, NEXT_DAY(SYSDATE,'星期一') FROM DUAL;

SYSDATE                |NEXT_DAY(SYSDATE,'星期一')|
-----------------------+-----------------------+
2024-03-16 14:19:00.000|2024-03-18 14:19:00.000|
```

#### 返回会话所在时区当前日期函数

> CURRENT\_DATE()函数

该函数得到会话时区的当前日期。示例脚本如下：

```sql
SELECT SESSIONTIMEZONE,TO_CHAR(CURRENT_DATE,'YYYY-MM-DD HH24:MI:SS') FROM DUAL;

SESSIONTIMEZONE|TO_CHAR(CURRENT_DATE,'YYYY-MM-DDHH24:MI:SS')|
---------------+--------------------------------------------+
Asia/Shanghai  |2024-03-16 14:21:05                         |
```

该脚本查询的是第8时区当前的系统时间，如果更改会话为第6时区，则可以利用如下脚本：

```sql
ALTER SESSION SET TIME_ZONE = '-6:0';
```

#### 提取指定日期特定部分的函数

> EXTRACT(datetime)函数

该函数可以从指定的时间当中提取到指定的日期部分，例如从给定的日期得到年月分等。示例脚本如下：

```sql
SELECT
	EXTRACT(YEAR FROM SYSDATE) YEAR,
	EXTRACT(MINUTE FROM TIMESTAMP '2010-06-18 12:23:10') MIN,
	EXTRACT(SECOND FROM TIMESTAMP '2010-06-18 12:23:10') SEC
FROM DUAL;

YEAR|MIN|SEC|
----+---+---+
2024| 23| 10|
```

#### 得到两个日期之间的月份数

> MONTHS\_BETWEEN(date1,date2)的函数

该函数返回date1和date2之间的月份数。函数两个参数都为日期型数据。当date1>date2时，如果两个参数表示日期是某月中的同一天，或它们都是某月中的最后一天，则该函数返回一整型数;否则，将返回小数。当datel\<date2时，则返回一负值。示例脚本如下:

```sql
SELECT 
	MONTHS_BETWEEN(TO_DATE('2010-7-1','YYYY-MM-DD'),TO_DATE('2010-6-1','YYYY-MM-DD')) ONE,
	MONTHS_BETWEEN(TO_DATE('2010-5-31','YYYY-MM-DD'),TO_DATE('2010-4-30','YYYY-MM-DD')) TWO,
	MONTHS_BETWEEN(TO_DATE('2010-5-31','YYYY-MM-DD'),TO_DATE('2010-9-30','YYYY-MM-DD')) THREE
FROM DUAL;

ONE|TWO|THREE|
---+---+-----+
  1|  1|   -4|
```

#### 时区时间转换函数

> NEW\_TIME(date,timezone1,timezone3)函数

该函数将返回时间date在时区timezone1转换到时区timezone2的时间。示例脚本如下：

```sql
SELECT
	TO_CHAR(SYSDATE,'YYYY-MM-DD HH24:MI:SS') ONE,
	TO_CHAR(NEW_TIME(SYSDATE,'PDT','EST'),'YYYY-MM-DD HH24:MI:SS') TWO
FROM DUAL;

ONE                |TWO                |
-------------------+-------------------+
2024-03-16 14:36:28|2024-03-16 16:36:28|
```

#### 日期四舍五入、截取函数

> ROUND(date\[,fmt])函数

该函数将date舍入到fmt指定的形式。如果参数fmt被省略，则date将被处理到最近一天。示例脚本如下：

```sql
SELECT TO_CHAR(ROUND(TO_DATE('2010-5-1 21:00:00','YYYY-MM-DD HH24:MI:SS')),'YYYY-MM-DD HH24:MI:SS') FROM DUAL;

TO_CHAR(ROUND(TO_DATE('2010-5-1 21:00:00','YYYY-MM-DDHH24:MI:SS')),'YYYY-MM-DDHH24:MI:SS')|
-----------------------------------------------------------------------------------------+
2010-05-02 00:00:00                                                                      |
```

> TRUNC(date\[,fmt])函数

该函数将date截取到fmt指定的形式。如果fmt省略，则截取到最近的日期。示例脚本如下：

```sql
SELECT TO_CHAR(TRUNC(TO_DATE('2010-5-1 21:00:00','YYYY-MM-DD HH24:MI:SS')),'YYYY-MM-DD HH24:MI:SS') FROM DUAL;

TO_CHAR(TRUNC(TO_DATE('2010-5-1 21:00:00','YYYY-MM-DDHH24:MI:SS')),'YYYY-MM-DDHH24:MI:SS')|
-----------------------------------------------------------------------------------------+
2010-05-01 00:00:00                                                                      |
```

### 转换函数

转换函数可以完成不同数据类型之间的转换，是平常使用比较多的函数类型之一。本节将介绍平常使用率较高的转换函数。

#### 字符串转ASCII类型字符串函数

> ASCIISTR(char)函数

该函数可将任意字符集的字符串转换为数据库字符集对应的ASCII字符串。char为字符类型。示例脚本如下:

```sql
SELECT ASCIISTR('这是测试!') FROM DUAL;

ASCIISTR('这是测试!')    |
---------------------+
\8FD9\662F\6D4B\8BD5!|
```

#### 二进制转十进制函数

> BIN\_TO\_NUM(data\[,data...])函数

该函数可以将二进制转换成对应的十进制。data表示二进制数，一位用“,”隔开。示例脚本如下:

```sql
SELECT BIN_TO_NUM(1), BIN_TO_NUM(1,0,0),BIN_TO_NUM(1,1,1) FROM DUAL;

BIN_TO_NUM(1)|BIN_TO_NUM(1,0,0)|BIN_TO_NUM(1,1,1)|
-------------+-----------------+-----------------+
            1|                4|                7|
```

#### 数据类型转换函数

> CAST(expr as type\_name)函数

该函数时进行类型转换的，可以把expr参数转成type\_name类型。基本上用于数字与字符之间以及字符与日期类型之前的转换。类型之前的转换Oracle有一套规则表。示例脚本如下：

```sql
SELECT 
	CAST('123' AS INTEGER) AS VHR,
	CAST(123 AS VARCHAR2(8)) AS NUM,
	CAST(SYSDATE AS VARCHAR2(12)) AS DT
FROM DUAL;

VHR|NUM|DT       |
---+---+---------+
123|123|16-3月 -24|
```

#### 字符串和ROWID相互转换函数

> CHARTOROWID(char)函数

该函数将字符串类型转成ROWID类型。char为待转的字符串，其类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2，但必须符合ROWID格式，长度为18。每一条记录都有一个rowid，rowid在整个数据库中唯一，可以利用SELECT查询该字段。示例脚本如下:

```sql
SELECT CHARTOROWID('AAARXnAABAAAVgggAB')FROM DUAL;

CHARTOROWID('AAARXNAABAAAVGGGAB')|
---------------------------------+
AAARXnAABAAAVggAAB               |
```

> ROWIDTOCHAR(rowid)函数

该函数将行记录的ROWID转成字符串。参数rowid长度为18，所以返回结果长度18。示例脚本如下:

```sql
SELECT ROWIDTOCHAR(ROWID) FROM DUAL;

ROWIDTOCHAR(ROWID)|
------------------+
AAAAB0AABAAAAOhAAA|
```

> ROWIDTONCHAR(rowid)函数

同ROWIDTOCHAR(rowid)操作相同，但返回类型是NVARCHAR2。

#### 字符串在字符集间转换函数

> CONVERT函数

该函数用于把字符串从一个字符集转到另一个字符集。函数的具体语法 结构是CONVERT(char,dest\_char sest\[,source\_char\_set])，各参数的表示含义如下:

* char:等待转换的字符。可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2CLOB、NCLOB类型。
* dest char sest:转变后的字符集。
* sourcecharset:原字符集，如果没有该参数，则默认数据实例字符集

示例脚本如下:

```sql
SELECT CONVERT('测试','US7ASCII','ZHS16GBK') FROM DUAL;

CONVERT('测试','US7ASCII','ZHS16GBK')|
-----------------------------------+
???                                |
```

#### 十六进制字符串与RAW类型相互转换函数

> HEXTORAW(char)函数

该函数把十六进制的字符串转换成raw类型的数据。其中参数char表示一个由十六进制字符组成的字符串。示例脚本如下:

```sql
SELECT HEXTORAW('4d') FROM DUAL;

HEXTORAW('4D')|
--------------+
M             |
```

> RAWTOHEX(raw)函数

与HEXTORAW函数相反，它把raw类型表示成一个由十六进制字符表示的串，返回VARCHAR2类型。示例脚本如下:

```sql
SELECT RAWTOHEX('4D') FROM DUAL;

RAWTOHEX('4D')|
--------------+
3444          |
```

> RAWTONHEX(raw)函数

同函数RAWTOHEX(raw)转换效果相同，不过返回的类型是NVARCHAR2类型，而不是VARCHAR2类型。这里不再给出示例。

#### 数值转换成字符型函数

> TO\_CHAR(number)函数

该函数将一个数值型参数转换成字符型数据。其具体语法结构是TO\_CHAR(n,\[,fmt\[,nlsparam]])，各参数表示含义如下:

* n:数值型数据。
* fmt:要转成字符的格式。
* nlsparam:由该参数指定fmt的特征。通常包括小数点字符、组分隔符、本地钱币符号。该函数如果想用的好需要了解多方面的知识，这里不做详细介绍。

示例脚本如下:

```sql
SELECT TO_CHAR(16.89,'99.9'),TO_CHAR(16.89) FROM DUAL;

TO_CHAR(16.89,'99.9')|TO_CHAR(16.89)|
---------------------+--------------+
 16.9                |16.89         |
```

TO\_CHAR(date)的数

该函数将一个日期型数据转换成一个字符型数据。它同前面介绍的同名函数一样，只不过转换的对象变化了。具体的语法结构是TO\_CHAR(n,,fmt \[,nlsparam]])，各参数具体含义如下:

* n:日期类型数据。
* fmt:要转成字符的格式。
* nlsparam:使用的语言类型。

示例脚本如下:

```sql
SELECT
	TO_CHAR(SYSDATE,'YYYY-MM-DD'),
	TO_CHAR(SYSDATE,'HH24:MI:SS'),
	TO_CHAR(SYSDATE,'Month','NLS_DATE_LANGUAGE=ENGLISH')
FROM DUAL;

TO_CHAR(SYSDATE,'YYYY-MM-DD')|TO_CHAR(SYSDATE,'HH24:MI:SS')|TO_CHAR(SYSDATE,'MONTH','NLS_DATE_LANGUAGE=ENGLISH')|
-----------------------------+-----------------------------+----------------------------------------------------+
2024-03-16                   |15:07:08                     |March                                               |
```

#### 字符转日期型函数

> TO\_DATE函数

该函数可将字符型数据转换成日期型数据。函数的具体语法结构是TO\_DATE(char\[,fmt\[,nlsparam]])。各参数的具体含义如下

* char:待转换的字符。类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2。
* fmt:表示转换的格式。
* nlsparam:控制格式化时使用的语言类型。

示例脚本如下：

```sql
SELECT TO_CHAR(TO_DATE('2010-7-1','YYYY-MM-DD'),'MONTH') FROM DUAL;

TO_CHAR(TO_DATE('2010-7-1','YYYY-MM-DD'),'MONTH')|
-------------------------------------------------+
7月                                               |
```

#### 字符串转数字函数

> TO\_NUMBER函数

该函数将字符串转成数字。语法结构是TO\_NUMBER (expr\[,fmmt\[,nlsparam]])，各参数表示的具体含义如下:

* expr:待转换的字符，其类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2。
* fmt:指定转换的数字格式。
* nlsparam:该参数指定fmt的特征。通常包括小数点字符、组分隔符、本地钱币符

示例脚本如下:

```sql
SELECT TO_NUMBER('2456.304','9999.999') FROM DUAL;

TO_NUMBER('2456.304','9999.999')|
--------------------------------+
                        2456.304|
```

#### 全角转半角函数

> TO\_SINGLE\_BYTE(char)函数

该函数将全角转为半角。char的类型可以是CHARVARCHAR2、NCHAR、NVARCHAR2。

示例脚本如下:

```sql
SELECT TO_SINGLE_BYTE('THIS IS A TEST') FROM DUAL;

TO_SINGLE_BYTE('THISISATEST')|
-----------------------------+
THIS IS A TEST               |
```

### NULL函数

NULL函数是用来处理空值时比较好的选择。本节介绍几个常用的空值处理函数。

#### 返回表达式为NULL的函数

> COALESCE(expr)函数

返回列表中第一个不为null的表达式。如果都为null，则返回一个null。示例脚本如下:

```sql
SELECT COALESCE(NULL,9-9,NULL) FROM DUAL;

COALESCE(NULL,9-9,NULL)|
-----------------------+
                      0|
```

#### 排除指定条件函数

> LNNVL(condition)函数

该函数可以得到除了condition要求条件之外的数据，包括NULL的条件，通常用于WHERE条件中。下面的示例将得到PRODUCTINFO表中数量低于70的产品 并包含数量为NULL的数据。示例脚本如下:

```sql
SELECT * FROM PRODUCTINFO WHERE LNNVL(QUANTITY>=70);
```

#### 替换NULL值函数

> NVL(expr1,expr2)函数

替换NULL值，表示如果expr1为NULL值，则返回expr2的值，否则返回expr的值。该函数要求两个参数类型一致，至少相互间能进行隐式的转换，否则会提示出错。

```sql
SELECT NVL(NULL,1) FROM DUAL;

NVL(NULL,1)|
-----------+
          1|
```

> NVL2(expr1,expr2,expr3)函数

该函数同NVL类似，不同的是当expr1为NULL时，函数返回expr3的值;当expr1不为空时，则返回expr2的值。

### 集合函数

集合的数经常配合GROUPBY或HAVING子句使用，当然它们也可以单独使用。该类型的函数中除了COUNT函数都会忽略列值为NULL的数据。

#### 求平均值函数

> AVG(\[distinctlall]expr)函数

该函数可求取指定列的平均值，表示某组的平均值，返回数值类型。各参数表示的具体含义如下:

* distinct:去除重复的值。
* all:表示所有的值，包括重复的值，也是默认值。
* expr:表达式。只能是数值类型。

```sql
SELECT AVG(ALL column_name) FROM DUAL;
```

#### 求记录数量函数

> COUNT(\* l \[distinct] \[all] expr)函数

该函数可以用来计算记录的数量或某列的个数。函数中必须指定列名，或全选使用星号。其中各参数表示的含义如下:

* \*:表示计算所有记录。
* distinct:表示去除重复的记录
* all:代表所有的，是默认选项。
* expr:要计算的对象，通常是表的列。

#### 返回最大、最小值函数

> MAX(\[distinct l all]expr)函数

该函数可以返回指定列中的最大值，通常都用在WHERE子句中的子查询。其中各参数表示的含义如下:

* distinct:表示去除重复的记录。
* all:代表所有的，是默认选项。
* expr:表的列。

同该函数效果相反但用法一致的有MIN(\[distinctlall]expr)函数，此函数获取指定列中的最小值。这里不再给出示例。

#### 求和函数

> SUM(\[distinct l all]expr)函数

该函数不同于COUNT函数，它分组计算指定列的和，如果不使用分组，则的数默认把整个表作为一组。各参数代表含义如下:

* distinct:表示去除重复的记录。
* all:代表所有的，是默认选项。
* expr:表的列。

### 其他函数

前面介绍了单行的数以及集合函数，下面介绍几个不属于前面类型的函数，主要是系统环境和编码方面的函数。

#### 返回登录名函数

> USER函数

该函数返回当前会话的登录名。演示脚本如下:

```sql
SELECT USER FROM DUAL;
```

> USERENV(parameter)函数

返回当前会话的信息。例如，当参数为Language时可以返回当前会话对应的语言、字符集等。SESSIONID可返回当前会话ID。ISDBA可返回当前用户是否DBA。

示例将演示返回当前用户是否DBA用户。脚本如下

```sql
SELECT 
    USERENV('ISDBA'),
    USERENV('LANGUAGE'),
    USERENV('SESSIONID')
FROM DUAL;

USERENV('ISDBA')|USERENV('LANGUAGE')              |USERENV('SESSIONID')|
----------------+---------------------------------+--------------------+
FALSE           |SIMPLIFIED CHINESE_CHINA.AL32UTF8|              182034|
```

> SYS\_CONTEXT(namespace,parameter)函数

该函数可以得到Oracle已经创建的context,名为USERENV的属性对应值。

```sql
SELECT SYS_CONTEXT('USERENV','SESSION_USER') FROM DUAL;

SYS_CONTEXT('USERENV','SESSION_USER')|
-------------------------------------+
DEMO                                 |
```

#### 表达式匹配函数

> DECODE函数

该函数的具体语法是DECODE(expr,search,result\[,searchl,result1]\[,defaut])。该函数的执行过程是，当expr符合条件search时就返回result的值，该过程可以重复多个，如果最后没有匹配的结果，可以返回默认值default，注意它是一对一的匹配过程。下面的示例将演示PRODUCTINFO中产品数量多于100的就显示“充足”，少于或等于100则显示“不足”。脚本如下:

```sql
SELECT
	DECODE(1,1,2,2,3,4),
	DECODE(2,1,2,2,3,4),
	DECODE(4,1,2,2,3,4),
	DECODE(0,1,2,2,3,4)
FROM DUAL;

DECODE(1,1,2,2,3,4)|DECODE(2,1,2,2,3,4)|DECODE(4,1,2,2,3,4)|DECODE(0,1,2,2,3,4)|
-------------------+-------------------+-------------------+-------------------+
                  2|                  3|                  4|                  4|
```

{% hint style="info" %}
从函数中可以看出它只能和单一的条件匹配，如果要想得到某个范围，可以利用其他的函数做辅助。
{% endhint %}

## PL/SQL基础

### 什么是PL/SQL

#### 认识PL/SQL

结构化查询语言(Structure Query Language，SQL)是用来访问和操作关系型数据库的-种标准通用语言，它属于第四代语言(4GL)，简单易学，使用它可以很方便地调用相应语句来取得结果。该语言的特点就是非过程化。也就是说，使用的时候不用指明执行的具体方法和途径，即不用关注任何的实现细节。但这种语言也有一个问题，就是在某些情况下满足不了复杂业务流程的需求，这就是第四代语言的不足之处。Oracle中的PL/SQL语言正是为了解决这一问题，PL/SQL属于第三代的语言(3GL)，也就是过程化的语言，同Java、C#一样可以关注细节，用它可以实现复杂的业务逻辑，是数据库开发人员的利器。

PL/SQL(Procedural Language/Structured Query Language)是Oracle公司在标准SQL语言基础上进行扩展而形成的一种可以在数据库上进行设计编程的语言，通过Oracle的PL/SQL引擎执行。PL/SQL完全可以像Java语言一样实现逻辑判断、条件循环以及异常处理等，这是标准的SQL很难办到的事情。由于它的基础是标准的SQL语句，这就使得数据库开发人员能快速地掌握并运用，相信这也是Oracle开发人员喜爱它的另一个重要原因。总的来说，PL/SQL有以下几个特点:

* 支持事务控制和SQL数据操作命令
* 支持SQL的所有数据类型，并且在此基础上扩展了新的数据类型，也支持SQL的函数以及运算符。
* PL/SOL可以存储在Oracle服务器中。
* 服务器上的PL/SQL程序可以使用权限进行控制。
* Oracle有自己的DBMS包，可以处理数据的控制和定义命令，

#### PL/SQL的优势

由于PL/SOL语言是从SOL语言扩展而来，所以PL/SOL除了支持SOL数据类型和函数外同时也支持Oracle对象类型。除此之外，同传统的SQL语言相比PL/SQL有以下几个优点:

1.  可以提高程序的运行性能

    标准的SQL被执行时，只能一条一条地向Oracle服务器发送。假如完成一个业务逻辑需要几条甚至几十条SQL语句，那么在这个过程中，客户端会几十次地连接数据库服务器，而连接数据库本身是一个很耗费资源的过程，当这个业务被完成时，会浪费大量的资源在网络连接上。

    如果此时换用PL/SQL语句，结果则不一样了。PL/SQL的语句块可以包含多条SQL语句，而语句块可以嵌入到程序中，甚至可以存储到Oracle服务器上。这样用户只需要连接一次数据库就可以把需要的参数传递过去，其他的部分将在Oracle服务器内部执行完成，然后返回最终的结果。这样就大大地节省了网络资源的开销。
2.  可以使程序模块化

    在程序块中可以实现一个或几个功能。例如，当想把一个动物的模型存到数据库里时，可能涉及几张表，如果使用标准的SQL完成该功能需要多条语句，而如果使用块，则可以把对多张表的操作都放到一个块内，而对外只提供一个调用方式和需要传入的参数。这对于编程开发人员是一个福音，他们不需要再写过多的SQL语句，只需要给出参数并调用一次PL/SQL的程 序块就好。这种操作的优势在介绍存储过程后显得尤其明显。使用块也可以把数据库数据同客户程序隔离开来，使得数据库表结构发生变化时，对调用者的影响减小到最低程度。
3.  可以采用逻辑控制语句来控制程序结构

    如果一个PL/SOL程序块中只能顺序地执行基本的SQL语句，那么它的意义实在有限。而实际当中PL/SQL可以利用条件或循环语句来控制程序的流程，这么做就大大地增加了PL/SQL的实用性，我们可以利用逻辑控制语句完成复杂的普通SQL语句完成不了的业务。
4.  利用处理运行时的错误信息

    标准的SQL在遇到错误时会提示异常。例如增加数据，一旦有异常就会终止，但是调用者却很难快速地发现错误点在哪儿，即使发现出问题的地方也只能是告诉开发人员该语句程序本身有问题，而不是逻辑上有问题。例如，在产品表里增加数据时，数量只是要求数值型，并没有更细的要求。假如增加的数据中该字段部分是一个负数，正常来说是可以进入数据库的，但这在逻辑上是不允许的，因为没有数量为负的产品。而利用PL/SQL就可以完全避免类似的问题，我们可以利用流程拒绝这部分记录进入数据库。利用PL/SQL还可以处理一些程序上的异常，不至于因终止SQL操作，而造成调用SQL的展示页面出现生硬的错误提示。
5.  良好的可移植性

    PL/SOL可以成功地运行到不同的服务器中。例如，从Windows的数据库服务器下移植到Linux的数据库服务器下。也可把PL/SQL从一个Oracle版本移植到其他版本的Oracle中。

#### PL/SQL的结构

PL/SQL程序的基本单位是块(block)，而PL/SQL块很明确地分三部分，其中包括声明部分、执行部分和异常处理部分。其中，声明部分以DECLARE作为开始标志，执行部分用BEGIN作为开始标志，而异常处理部分则以EXCEPTION为开始标志。其中的执行部分是必需的，而其余的两个部分则可选。下面的一段文字描述了PL/SQL块的三部分:

```sql
[DECLARE]  --声明开始关键字
           /*这里是声明部分，包括PL/SQL中的变量、常量以及类型等*/
BEGIN      --执行部分开始的标志
	   /*这里是执行部分，是整个PL/SQL块的主体部分，该部分在PL/SQL块中必须存在，可以是SQL语句或者程序流程控制语句等*/
[EXCEPTION]--异常开始部分的关键字
	   /*这里是异常处理部分，当出现异常时程序流程可以进入此处*/
END;       --执行结束标志
```

{% hint style="info" %}
需要记住:无论PL/SOL程序段的代码量有多少，它的基本结构只是由这三部分组成。
{% endhint %}

【示例1】只有执行体部分的结构 该示例只有执行体部分，也就是只有“BEGIN..END;”部分，该语句块中将输出一句话这已经是最简单的执行体了。脚本如下:

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('这是执行体部分。');
END;
```

【示例2】包含声明和执行体两部分的结构该示例除了执行体外还有声明部分，具体操作是声明一个变量，然后为变量赋值，最后输出该变量的值。脚本如下:

```sql
DECLARE
v_result NUMBER(8,2);
BEGIN
    v_result := 100/6;
    DBMS_OUTPUT.PUT_LINE('最后结果是:' || v_result);
END;
```

【示例3】包含声明、执行体和异常部分的结构

该示例将从产品类型表CATEGORYINFO中查询产品类型“雨具”对应的产品类型编码，并把该编码存储到变量中，最后输出到屏幕。脚本如下:

```sql
DECLARE
v_categoryid VARCHAR2(12);
BEGIN
    SELECT 
        CATEGORUID INTO V_CAtEGORYID
    FROM CATEGORYINFO
    WHERE CATEGORYNAME = '雨具';
    DBMS_OUTPUT.PUT_LINE('用具对应的编码是:' || v_categoryid);
    
    EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('没有对应的编码!')7
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('对应数据过多，请确认!’);
END;
```

{% hint style="info" %}
SELECT...INTO....语句是PL/SQL特有的赋值语句，该语句表示的意思是SELECT后面列出要查询的字段列表，INTO后面是变量名称，它表示把查询出来的值存储到变量中。这里有两个问题需要注意，就是SELECT列名顺序和INTO后面的变量名顺序要一一对应，还有就是该类型语句每次只能返回一条记录，如果返回记录超过一条或没有返回记录都会引发异常。
{% endhint %}

#### PL/SQL的基本规则

1.  PL/SQL中允许出现的字符集:

    字母，包括大写和小写。

    数字，即0\~9。

    空格、回车符以及制表符。

    符号包括+、-、\*、/、<、>、=、!、\~、^、;、:、.、'、@、%、,、"、#、$、&、\_、1、(、)、\[、]、{、}、?。
2.  下面列出一些PL/SQL必须遵守的要求:

    标识符不区分大小写。例如，TEST同Test、test是一样的。所有的名称在存储时都被修改成大写，这一点读者需要注意。

    标识符中只允许字母、数字、下划线，并且以字母开头。

    标识符最多30个字符。

    不能使用保留字。如与保留字同名必须使用双引号括起来。

    语句使用分号结束。即使多条语句在同一行，只要它们都正常结束，那么就没有问题而且在语句块的结束标志END后面同样需要使用分号。

    语句的关键词、标识符、字段的名称以及表的名称等都需要空格的分隔。

    字符类型和日期类型需要使用单引号括起。
3.  以下是为了增强代码的阅读性的相关建议，这些不是必须要遵守的，但通常情况下有些单位也可能把这些规范作为硬性要求。

    每行只写一条语句。

    全部的保留字、Oracle的内置函数、程序包以及用户定义的数据类型都用大写。

    所有的过程名称大写。

    所有的变量以及自建的过程或游标、触发器名称都要使用有意义的名称命名。

    命名应以“\_”的连接方式，而不是用大小写混合的方式(如果只为了方便自己的阅读,可以使用大小写混合)。

    变量前最好加上前缀，以表示该变量的数据类型、作用范围等。

    每个变量都应加上注释。

    在重要的程序段处都应加上注释。

    建议3个半角空格替代TAB键进行缩进。

    逗号后面以及操作符的前后都应加空格。

{% hint style="info" %}
以上只是比较基本的规则，可以提高代码的可读性，在企业的每个项目小组中会根据实际的情况做出更细的要求，甚至形成规范文档。在日常开发中应注意这些规范，形成良好的编程习惯。
{% endhint %}

#### PL/SQL的注释

单行注释：使用“--”两个短横线，可以注释掉后面的语句。

多行注释：使用“/\*..\*/”，可以注释掉这两部分包含的部分。

### PL/SQL变量的使用

所有的编程语言中变量是使用最频繁的。PL/SQL作为一个面向过程的数据库编程语言同样少不了变量，利用变量可以把PL/SQL块需要的参数传递进来，做到动态执行程序，同时也可以利用变量在PL/SOL内部进行值的相互传递，其至可以把值传递出去，最终返回给用户由此可见，变量是PL/SQL不可缺少的一部分。

#### 变量、常量的类型及语法

变量就是它所表示的值是可以变化的，而常量就是当初始化后，其值不可以再改变。PL/SQL是一种强类型的语言，所以当使用变量或常量的时候必须声明，否则提示错误。下面是变量和常量声明的语法介绍。

#### 变量声明语法结构

{% code lineNumbers="true" %}
```sql
variable_name datatype
[
    [NOT NULL]
    {:= | DEFAULT} expression
];

【语法说明】
variable_name:表示变量的名称。名称可以根据读者实际情况自行定义。
datatype:变量的数据类型。
第2~5行表示可选部分。如果没有这部分，那么只需要变量的名称以及对应的数据类型即可，声明的时候变量可以不赋值。
NOT NULL:表示非空约束。
{:= | DEFAULT}:当使用NOT NULL属性时，大括号里的内容为必需，表示二选一“:=”表示赋值;DEFAULT表示默认值。
expression:表示变量存储的值，该项可以是表达式。
```
{% endcode %}

#### 常量声明的具体语法结构

{% code lineNumbers="true" %}
```sql
constant_name CONSTANT datatype
[NOT NULL]
(:= | DEFAULT } expression;
 
【语法说明】
constant_name:表示声明的常量名称。
CONSTANT:如果表示常量，该项必须有，否则表示变量。
datatype:常量的数据类型。
NOT NULL:表示常量值非空。
{:= | DEFAULT}:表示常量必须显式地为其赋值，方式同变量一样。
expression:值或表达式。
```
{% endcode %}

变量和常量的语法结构相似，其中expression表示的含义完全一样，都可以是如下类型的表达式:

* 字符型表达式
* 数值型表达式
* 日期型表达式
* 布尔型表达式
* 直接的一个值

变量和常量的数据类型可以概括性地分为如下三种类型:

* 标量类型变量:单一类型，不存在组合。
* 复合类型变量:由几种单一类型组合而成的一个结构体
* 引用类型变量:使用一个其他数据项的引用。

#### 标量类型的变量

标量类型的变量是最简单类型的变量，也是普通开发者最常用的一种变量类型，它本身是单一的值，不包含任何的类型组合。标量类型主要包含数值类型、字符类型、布尔类型和日期类型。还有一种比较特殊的声明变量类型的方式，就是利用%TYPE。下面对这几种类型做详细的介绍。

> 数据类型

主要用来存放数字型的数据。最常用的就是NUMBER、PLSINTEGERBINARY\_INTENER类型，还有一个类型是SIMPLE\_INTEGER。

* NUMBER类型可以表示整数和浮点数，该类型以十进制存储。其通用格式是NUMBER(precision,scale)，其中的precision表示精度，也就是数字的位数，可达38位:scale表示小数点后的位数。例如，NUMBER(3,1)可以存储-99.9\~99.9之间的数值。该类型在定义的时候可以把precision和scale省略。例如，可以定义成NUMBER(8)或NUMBER。
* PLS\_INTEGER和BINARY\_INTENER类型通常可以认为是一样的类型。表示的范围是-2147483648\~2147483647之间。二者不一样的地方就是BINARY\_INTENER发生溢出的时候能为其指派一个NUMBER类型而不至于发生异常，但PLSINTEGER溢出会发生异常，建议使用PLS\_INTEGER类型。
* SIMPLE\_INTEGER类型属于PLS\_INTEGER的子类型，它的取值范围同PLSINTEGER一样，只是该类型不允许为空。如果数据本身不需要溢出检查而且也不可能是空，那么可以选择该类型，该类型的性能比PLS\_INTEGER高。

> 字符类型

可以用来存储单个的字符或字符串的类型。主要有CHAR、VARCHAR2(VARCHAR)、NCHAR、NVARCHAR2和LONG类型。

* CHAR类型，用来描述固定长度的字符串，最长为32767个字节，默认是长度1。该类型的字符中如果值的长度达不到定义的长度，那么将以空格补齐。通常定义格式为CHAR(maximum\_size)。一旦使用该类型，那么在数据库提取数据时可能要做空格处理。
* VARCHAR2类型，作为变量的时候最长为32767个字节，但作为字段存储的时候是4000个字节。该类型表示可变长度的字符串。也就是说，当值的长度达不到定义的长度时，不用空格补齐，这样就可以节省一定的空间。
* NCHAR、NVARCHAR2类型使用方式同CHAR和VARCHAR2相同，只不过它们与国家的字符集有关。
* LONG类型，以可变的方式存储数据，PL/SQL中作为变量可表示最长可达32760字节的字符串，如果作为存储字段则可达2GB。

> 布尔类型

它不能用做定义表中的数据类型。但PL/SQL中该类型可以用来存储逻辑上的值。它有3个值可选:TRUE、FALSE、NULL。

> 日期类型

主要有DATE和TIMESTAMP:

* DATE类型可以存储月、年、日、世纪、时、分和秒。
* TIMESTAMP类型由DATE演变而来，可以存储月、年、日、世纪、时、分和秒以及小数的秒。

> %TYPE

%TYPE方式定义变量类型，这种定义变量类型的方式和前面所介绍的直接定义变量类型有所不同，它利用已经存在的数据类型来定义新数据的数据类型。例如，当定义多个变量或常量时，只要前面使用过的数据类型，后面的变量就可以利用%TYPE引用。最常见的就是把表中字段类型作为变量或常量的数据类型。使用此种方式的好处有如下几点:

* 利用%TYPE定义的变量或常量数据类型都一致，当有变动的需求时，只要改变被引用的变量或常量的数据类型，其他引用处的数据类型自然就变了，避免了逐条修改的麻烦。
* 使用PL/SQL语句块通常都是操作数据库表的数据，操作过程中避免不了出现数据的传递，这时变量利用%TYPE就可以完全兼容提取的数据，而不至于出现数据溢出或不符的情况。
* 当利用%TYPE定义数据类型时，可以保证变量的数据类型和表中的字段类型同步，当表字段类型发生变化时，PL/SQL块变量的数据类型不需要修改。

#### 复合类型的变量

所谓复合类型的变量，就是每变量包含几个元素，可以存储多个值。这种变量类型同标量类型使用方式稍有差异，复合类型需要先定义，然后才能声明该类型的变量。最常用的是三种类型，一种是记录类型:一种是索引表类型，还有一种是VARRAY数组。

> PL/SQL记录类型

该类型可以包含一个或多个成员，而每个成员的类型可以不同，成员可以是标量类型，也可以是引用其他变量的类型(使用%TYPE)。该类型比较适合处理查询语句中有多个列的情况。最常用的就是在调用某张表中的一行记录时，利用该类型变量存储这行记录。如果想要调用其中的数据，可以用“变量名称,成员名称”的格式进行调用。“记录类型”有两种声明的方式。

第一种声明语法如下:

{% code lineNumbers="true" %}
```sql
TYPE type_name IS RECORD
(
	field_name datatype
    [
        [NOT NULL]
        {:= | DEFAULT} expression
    ]
    [, field_name datatype [[NOT NULL]{:= | DEFAULT} expression ]...]
)

【语法说明】
type_name表示定义的记录类型的名称。其余都是关键词。
field_name表示行记录的成员名称，datatype表示行记录成员数据类型。
[NOT NULL]表示可选部分，可以约束记录的成员非空。
{:=|DEFAULT)表示为记录成员赋值，expression为赋值表达式。
记录类型里可以有多个成员。
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
DECLARE
TYPE product_rec IS RECORD
(
    v_id productinfo.id%TYPE, --产品ID
    v_name VARCHAR2(20),      --产品名称
    v_price NUMBER(8,2)       --价格
);
v_p product_rec;
BEGIN
    SELECT id,name,price into v_p
    FROM productinfo
    WHERE id = '0240040001';
    DBMC_OUTPUT.PUT_LINE('id = ' || v_p.id);
    DBMC_OUTPUT.PUT_LINE('name = ' || v_p.name);
    DBMC_OUTPUT.PUT_LINE('price = ' || v_p.price);
END;
```
{% endcode %}

> 利用%ROWTYPE声明记录类型数据

前面已经介绍过“记录类型”的变量，该类型是提取行记录时常用的存储数据的方式。除了上面的直接声明方式外还有一种声明记录类型的方式，就是利用%ROWTYPE。这种声明方式可以直接引用表中的行作为变量类型。它同%TYPE类似，可以避免因表中字段的数据类型改变而导致PL/SQL块出错的问题。

{% code lineNumbers="true" %}
```sql
DECLARE
    v_p productinfo%ROWTYPE;
BEGIN
    SELECT * into v_p
    FROM productinfo
    WHERE id = '0240040001';
    
    DBMC_OUTPUT.PUT_LINE('id = ' || v_p.id);
    DBMC_OUTPUT.PUT_LINE('name = ' || v_p.name);
    DBMC_OUTPUT.PUT_LINE('price = ' || v_p.price);
END;
```
{% endcode %}

> PL/SQL索引表类型(关联数组)

该类型和数组相似，它利用键值查找对应的值。这里键值同真正数组的下标不同，索引表中下标允许使用字符串。数组的长度不是固定值，它可以根据需要自动增长。其中的键值是整数或字符串。而其中的值就是普通的标量类型，也可以是记录类型。可以利用“变量名称(值)”为其赋值或取值，如果某个键值的指向已经有数据了，那么该操作就是更改已有的数据。具体语法如下:

{% code lineNumbers="true" %}
```sql
TYPE type_name IS TABLE OF
{
    column_type |
    variable_name%TYPE |
    table_name.column_name%TYPE |
    table_name%ROWTYPE
}
[NOT NULL]
INDEX BY { PLS_INTEGER | BINARY_INTEGER | VARCHAR2(v_size) }
```
{% endcode %}

variable\_name就是变量的名称，而type\_name就是索引表的名称。日常开发中可以选其中，择用数字作为键值或以字符串作为键值。下面的两个示例将演示如何使用这两种方式操作。

{% code lineNumbers="true" %}
```sql
DECLARE
TYPE prodt_tab_fst IS TABLE OF productinfo%ROWTYPE
    INDEX BY BINARY_INTEGER;

TYPE prodt_tab_sec IS TABLE OF VARCHAR2(8)
    INDEX BY PLS_INTEGER;
    
v_prt_row prodt_tab_fst;
v_prt prodt_tab_sec;

BEGIN
    v_prt(1) := '整数';
    v_prt(-1) := '负数';
    
    SELECT * FROM v_prt_row(1)
    FROM productinfo
    WHERE productid = '0240040001';
    
    DBMS_OUTPUT.PUT_LINE('行数据：='|| v_prtow(1).id);
    DBMS_OUTPUT.PUT_LINE('行数据：='|| v_prt(1));
    DBMS_OUTPUT.PUT_LINE('行数据：='|| v_prt(-1));
END;
```
{% endcode %}

<pre class="language-sql" data-line-numbers><code class="lang-sql">--字符串为键值的索引表
DECLARE
TYPE prodt_tab_thd IS TABLE OF NUMBER(8)
  INDEX BY VARCHAR(20);
<strong>  v_prt_chr prodt_tab_thd;
</strong>
BEGIN
  v_prt_chr('test') := 123;
  v_prt_chr('test1') := 0;
  DBMS_OUTPUT.PUT_LINE('行数据：='|| v_prt_chr.first);
  DBMS_OUTPUT.PUT_LINE('行数据：='|| v_prt_chr(v_prt_chr.first));
  DBMS_OUTPUT.PUT_LINE('行数据：='|| v_prt_chr('test1'));
END;
</code></pre>

> VARRAY变长数组

该类型的元素个数是需要限制的，它是一个存储有序元素的集合。集合下标从1开始，比较适合较少的数据使用。声明语法如下:

{% code lineNumbers="true" %}
```sql
TYPE type_name IS { VARRAY | VARYING ARRAY} (size_limit)
	OF element_type [NOT NULL]

【语法说明】
type_name:表示该数组的名称。
{VARRAY | VARYING ARRAY}:必选项，二选一，表示数组类型
size_limit:该数组的长度。
element_type:数组里元素的类型。
```
{% endcode %}

```sql
DECLARE
    TYPE varr IS VARRAY(100) OF VARCHAR2(20);
    v_p varr:=varr('1','2','3');
BEGIN
    v_p(1):='THIS IS A';
    v_p(2):='TEST';
    DBMS_OUTPUT.PUT_LINE('行1数据：='|| v_p(1));
    DBMS_OUTPUT.PUT_LINE('行2数据：='|| v_p(2));
    DBMS_OUTPUT.PUT_LINE('行3数据：='|| v_p(3));
END;
```

### 表达式

数据库中经常使用表达式来计算结果，尤其在变量和常量的使用过程中。在前面已经接触过表达式的使用，它和普通编程语言的表达式很类似。本节将系统地介绍表达式的类型以及如何使用表达式。 表达式根据操作数据类型的不同可以分为如下几类:

* 数值表达式;
* 关系表达式;
* 逻辑表达式。

下面对这几个表达式进行讲解。

#### 数值表达式

数值表达式就是由数值类型的常量、变量以及函数，由算术运算符连接而成。在PL/SQL可以使用的算术运算符有:

* 加号+；减号-；乘号\*；除号/；乘方\*\*；

#### 关系表达式和逻辑表达式

由关系运算符连接起来的字符或数值称为关系表达式。其中关系运算符主要有以下几种:

* 等于号=；小于号<；大于号>；小于等于号<=；大于等于>=；不等于号!=和<>。

所谓逻辑表达式，就是由逻辑符号和常量或变量等组成的表达式。逻辑符号比较少，常见的有下面3种:

* 逻辑非NOT；逻辑或OR；逻辑与AND。

{% hint style="info" %}
关系表达式中的等于号和赋值符号“:=”的差异。
{% endhint %}

### PL/SQL结构控制

#### IF条件控制语句

> IF...结构

这是IF语句中最简单的结构方式，它只有一个IF语句，如果给定的表达式不成立，那么将继续向下执行。其语法结构如下:

```sql
IF condition THEN
    statements;
END IF;
```

> IF...ELSIF...ELSE...结构

该类型的结构表示不是选A就是选B。该结构表示要么执行后面的语句，要么执行ELSE后面的语句，是二选一的模式。该结构执行完成后，程序会继续向后执行。其语法结构如下:

{% code lineNumbers="true" %}
```sql
IF condition THEN
    statements;
ELSIF condition THNE
    statements;
ELSE
    statements;
END IF;
```
{% endcode %}

> 嵌套使用IF语句

IF语句可以嵌套使用，这使得判断的条件更加精细。

<pre class="language-sql" data-line-numbers><code class="lang-sql">IF condition THEN
    statements;
    IF condition THEN
        statements;
    ELSE
        statements;
<strong>    END IF;
</strong>ESLE
    statements;
END IF;
</code></pre>

#### CASE条件控制语句

CASE语句同IF语句类似，也是根据条件选择对应的语句执行。

CASE语句可以分为以下两种类型:

* 一种是简单的CASE语句。它给出一个表达式，并把表达式结果同提供的几个可预见的结果做比较，如果比较成功，则执行对应的语句序列。
* 另一种是搜索式的CASE语句。它会提供多个布尔表达式，然后选择第一个为TRUE的表达式，执行对应的脚本。

> 简单CASE语句

该类型CASE语句的语法结构如下:

{% code lineNumbers="true" %}
```sql
[ <<label_name>> ]
CASE case_operand
WHEN when_operand THEN
statement ;
[
    WHEN when_operand THEN
    statement ;
]...
[ELSE statement [statement]]...;
END CASE [label_name];
```
{% endcode %}

> 语法说明

* <\<labelname>>：这是一个标签，可以选择性添加。如果添加标签，建议在CASE语句结束时也标明该标签，表示结束的是某个CASE语句，使用该标签可以提高可阅读性。
* case\_operand：这是一个表达式，通常是一个变量。PL/SQL中除了BLOB、BFIFE、对象类型、PLSOL记录类型，索引表，变长数组或套表，其他类型都允许。
* when\_operand：它是case\_operand对应的结果。如果when\_operand的值同case\_operand的值相同，那么将执行该子句下的语句，也就是第4行的statement。CASE语句中包含一个或多个WHEN...THEN子句。
* \[ELSE statement \[statement]]：它所表示的含义是当所有的when\_operand值都不能对应case\_operand的值时，会执行ELSE处的语句。

> 搜索式的CASE语句

搜索式的CASE语句会依次检查布尔值是否为TRUE，一旦为TRUE，那么它所在的WHEN子句会被执行，而且它后面的布尔表达式将不被考虑。如果所有的布尔表达式都不为TRUE，那么程序将转到ELSE子句，如果没有ELSE子句，系统会给出CASE\_NOT\_FOUND异常。语法结构如下:

{% code lineNumbers="true" %}
```sql
[<<label_name>>]
CASE
WHEN boolean_expression THEN statement ;
[WHEN boolean_expression THEN statement ;]...
[ELSE statement [statment]...];
END CASE [<<label_name>>];

【代码解析】
从语法中可以看出搜索式的CASE语句在第2行处和简单CASE语句不同。它没有表达式而简单CASE语句里有表达式。
第3行的boolean_expression为布尔表达式，而且只能是布尔表达式。
语法中其他项代表的含义见“简单CASE语句”的语法说明。
```
{% endcode %}

#### LOOP循环控制语句

LOOP语句也叫循环语句，它能让我们重复地执行指定的语句块。LOOP语句有以下四种形式:

* LOOP；
* WHILE...LOOP；
* FOR...LOOP；
* CURSOR FOR LOOP；

> 基本的LOOP

{% code lineNumbers="true" %}
```sql
[<<label_name>>]
LOOP
    statement...
END LOOP [label_name];

【语法说明】
<<label_name >>:LOOP语句的标签，是可选项。
LOOP:LOOP循环的开始标志。
statement:LOOP语句中需要重复执行的语句
END LOOP:LOOP循环结束标志。
```
{% endcode %}

基本的LOOP语句需要和条件控制语句一起使用，否则会出现死循环的情况，直到内存溢出时抛出异常才能终止循环。通常情况下，正常终止循环的方式有以下两种:

*   在LOOP循环当中可以使用IF语句与EXIT的组合来结束循环。EXIT必须在循环体内部它可以使循环正常无条件终止，并且终止循环后程序会正常执行循环外的语句。它如果同IF语句一起使用，表示当满足某个条件时程序退出循环体，继续向后执行。

    > EXIT默认是终止退出当前的循环，但如果使用标签，可以终止并退出指定的LOOP循环。
* 使用EXIT...WHEN语句来结束循环。这种方式在游标中会经常使用，这种情况会在后面的章节见到。它所代表的含义是:当WHEN后面的条件为TRUE时，EXIT会被触发，终止退出指定的循环，如果EXIT后不加LOOP标签，则表示终止退出当前循环。该语句可以替换简单的IF语句，下面的示例将演示这一点。

> WHILE...LOOP语句

WHILE...LOOP结构的语句本身可以终止LOOP循环，当WHILE后面的布尔表达式为TRUE时，LOOP和END LOOP之间的语句集将执行一次，而后会重新判断WHILE后面的表达式是否为TRUE。

{% code lineNumbers="true" %}
```sql
[<<label_name>>]
WHILE boolean_expression
LOOP
    statement...
END LOOP [label_name];
```
{% endcode %}

> FOR...LOOP语句

FOR...LOOP语句循环遍历指定范围内的整数。该范围被FOR和LOOP关键词封闭。当第一次进入循环时，其循环范围会被确定，并且以后不会再次计算。每循环一次，其循环次数将会增加1。 FOR...LOOP语句的语法结构如下:

{% code lineNumbers="true" %}
```sql
[<<label_name>>]
FOR index_name IN [REVERSE] lower_bound .. upper_bound 
LOOP
    statement...
END LOOP [label_name];

【语法说明】
index_name:循环计数器，该变量可以得到当前的循环次数，但是不能为其赋值。
REVERSE:该项指定循环的方式。默认的循环方式是由下标界到上标界，也就是从lower_bound到upper_bound
    如果使用REVERSE关键词，那么循环方式正好相反，也就是从上标界到下标界。
lower_bound:循环范围的下标界
upper_bound:循环范围的上标界。
lower_bound和upper_bound两者需要用“..”连接费 
```
{% endcode %}

FOR...LOOP循环当中的循环范围可以动态地获取；下标界完全可以用数值型的变量替代。 需要说明的是，当lower\_bound和upper\_bound相等时，循环中的语句只能执行一次。

### PL/SQL使用DML和DDL语言

PL/SQL当中除了可以执行查询语句外，也允许执行DML语言和DDL语言。结构控制和DML操作相结合能帮助我们完成很多复杂的业务。不过在PL/SQL中DML语言和DDL语言这两者的执行方式有所差异。

#### DML语句的使用

由于PL/SQL对标准SQL的兼容，PL/SQL当中允许使用SQL命令，但有些命令在使用方式上有所改变。DML语句在PL/SQL中的使用方式和单独执行DML操作没有区别，而SELECT和DDL的使用方式都有所改变。&#x20;

> INSERT语句示例 要求判断产品类型编码是否存在，如果不存在，则增加该编码类型的记录。脚本如下:

{% code lineNumbers="true" %}
```sql
DECLARE
v_catgid VARCHAR2(10) := 0;
v_bol BOOLEAN := TRUE;
BEGIN
    SELECT CATEGORYID INTO v_catgid
    FROM CATEGORYINFO
    WHERE CATEGORUNAME = '电脑';
EXCEPTION
WHEN NOT_DATA_FOUND THEN
    IF v_bol THNE
        INSERT INTO CATEGORYINFO VALUES('01001','电脑');
        COMMIT;
    END IF;
END;
```
{% endcode %}

#### DDL语句的使用

PL/SQL中也允许使用DDL操作语句，但使用方法和DML有所差异。DDL语句要想在PL/SQL块中使用，需要一条命令来执行，该命令是EXECUTE\_IMMEDIATE，利用它可以执行动态的SQL语句，也就是利用它不仅仅可以执行DDL语句，也可以执行DML语句。该命令替代了DBMS\_SQL包，其性能也有所提高，但是依然不建议过程使用该命令。

{% code lineNumbers="true" %}
```sql
DECLARE
pc_createStr VARCHAR2(200);

BEGIN
pc_reateStr := 'CREATE TABLETAB TEST(
    OPERID VARCHAR2(10) PRIMARY KEY,
    OPERNAME VARCHAR2(30),
    OPERDATE DATE
)';

    EXECUTE IMMEDIATE pc_reateStr;
END;
```
{% endcode %}

> 通常情况下，利用EXECUTE IMMEDIATE命令执行DDL语句会用在存储过程当中。这样可以更好地实现代码可移植性。

### PL/SQL中的异常

同其他语言的程序一样，用PL/SQL编写的程序在运行过程当中也会出现各种错误，这些错误也可以称为异常。异常在PL/SQL中是很重要的一部分。

#### 什么是异常

PL/SQL运行过程中有可能会出现错误，这些错误有的来自程序本身，也有的来自开发人员自定义的数据，而所有的这些错误我们称之为异常(编译时的错误不能称为异常)。为了使程序有更好的阅读性和健壮性，PL/SQL采用了捕获并统一处理异常的方式。当异常发生时，程序会无条件跳转到异常块处，将控制权限交给异常处理程序。异常处理程序将进行异常匹配，如果当前的块内没有对应的异常名称，则异常会传至当前程序的上一层程序中查找，如果一直向上查找却依然没有找到对应的处理方式,则该异常会被传至当前的主机调用中，而运行的程序也会中断。

#### 处理异常的语法

{% code lineNumbers="true" %}
```sql
EXCEPTION
    WHEN exceptionl [OR exception2...] THEN --异常列表
        statement [ statement ] ...	    --语句序列
    [WHEN exception3 [OR exception4...]THEN --异常列表
        statement[ statement ]...] 
    [WHEN OTHERS THEN 
        statement [statement]..]

【代码解析】
EXCEPTION表示声明异常块部分，它是异常处理部分开始的标志。
WHEN后面接异常名称列表，
THEN后面接语句序列，也就是说发生的异常和异常列表里的异常相匹配时，可以执行指定的语句序列，以完成善后操作。
允许多个WHEN关键词。
WHEN OTHERS THEN语句通常是异常处理的最后部分，它表示如果抛出的异常在前面没有被捕获，那么将在这个地方被捕获。
该语句可以不用，但不被捕获的异常会传递到主机环境。
```
{% endcode %}

Oracle中的异常可以分为三类:

* 预定义异常
* 预定义异常
* 自定义异常

其中，预定义异常和非预定义异常都与Oracle中的错误有关，当出现错误时会自动触发，而自定义异常与Oracle的错误没有关系，它是人为的为某种特殊情况定义的异常，也不会自动触发，需要显式的操作来触发。

#### 预定义异常

Oracle中为每个错误提供一个错误号，而捕获异常则需要异常有名称。Oracle提供了一些已经定义好名称的常用异常，这就是预定义异常。例如，前面在使用SELECT...INTO语句时，如果返回超过一条记录就会触发TOO\_MANY\_ROWS异常。

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption><p>预定义异常</p></figcaption></figure>

Oracle一共提供了25种预定义异常。利用下面的查询语句可以查看Oracle的预定义异常:

```sql
SELECT * FROM DBA_SOURCE 
WHERE NAME='STANDARD' AND TEXT LIKE '%EXCEPTION_INIT%';
```

#### 非预定异常

Oracle中的异常更多都是非预定义异常。也就是说，它们只有错误编号和相关的错误描述，而没有名称的异常是不能被捕捉的。为了解决该问题，Oracle允许开发人员为这样的异常添加一个名称，使得它们能够被异常处理模块捕捉到。 为一个非预定义异常定义名称需要如下两步:

* 声明一个异常的名称。
* 把这个名称和异常的编号相互关联。

Oracle处理预定义异常和非预定义异常并没有区别。

> 关联非预定异常示例

由于产品表PRODUCTINFO中的产品类型引用了产品类型表CATEGROYINFO的编码，所以当修改产品类型编码时有可能造成PRODUCTINFO产生垃圾数据。为了避免这种情况，表PRODUCTINFO中可以使用外键约束。这时如果直接修改PRODUCTINFO表中的产品类型编码，就有可能导致ORA-02291错误。

<pre class="language-sql" data-line-numbers><code class="lang-sql"><strong>--创建外键表
</strong><strong>CREATE TABLE CATEGORY(
</strong>    ID VARCHAR2(10) NOT NULL,
    NAME VARCHAR2(200),
    CONSTRAINT PRK PRIMARY KEY(ID)
)

--创建外键
ALTER TABLE PRODUCTINFO 
ADD CATEGORY_ID VARCHAR2(10)
ADD CONSTRAINT CATEGORY_PRK FOREIGN KEY(CATEGORY_ID) REFERENCES CATEGORY(ID);

--关联非预定异常示例
DECLARE
v_ctgy VARCHAR2(10);
my_2291_exp EXCEPTION;
PRAGMA EXCEPTION_INIT(my_2291_exp,-2291);
BEGIN
    v_ctgy := '1111111111';
    UPDATE PRODUCTINFO SET CATEGORY_id = v_ctgy;
    EXCEPTION
    WHEN my_2291_exp THEN
        DBMS_OUTPUT.PUT_LINE('违反完整约束条件，未找到父项关键字!');
        DBMS_OUTPUT.PUT_LINE('SQLERRM:' || SQLERRM);
        DBMS_OUTPUT.PUT_LINE('SQLCODE:' || SQLCODE);
        ROLLBACK;
END;
</code></pre>

#### 自定义异常

如果开发当中遇到与实际业务相关的错误，例如产品数量不允许为负数，生产日期必须在质保日期之前等，这些和业务相关的问题并不能算系统错误，也不能使用预定义和非预定义异常来捕捉它们。如果要想用异常的方式处理这些问题，那么这样的异常需要开发人员自己编写，而且在调用的时候也需要显式的触发。

{% code lineNumbers="true" %}
```sql
DECLARE
exp EXCEPTION;
--异常关联一个错误号。范围是-20999~-20000的负整数，该范围内随便使用，不用担心被占用。
PRAGMA EXCEPTION_INIT(exp,-20001); 

BEGIN
    EXCEPTION
    WHEN exp THEN
    	DBMS_OUTPUT.PUTLINE('出现产品数量为空的数据，请核查!');
        ROLLBACK;
END;
```
{% endcode %}

### PL/SQL函数编写

函数由PL/SQL定义，可以操作各种数据项目完成计算并返回计算的结果值。利用函数可以把复杂的计算过程封装起来，避免所有的开发人员面对复杂的算法。

#### 函数的组成

函数主要由以下几个部分组成:

* 输入部分。函数允许有输入的参数，调用函数时需要给这些参数赋值。
* 逻辑计算部分。函数内部将完成对各种数据项目的计算，它可以进行很简单的算术表达式运算，也可以调用多个SQL内置函数或自定义函数进行运算。
* 输出部分。函数要求都有返回值。

#### 函数语法

{% code lineNumbers="true" %}
```sql
CREATE [ OR REPLACE ] FUNCTION [ schema. ] function_name
 [
     ( parameter_declaration [, parameter_declaration] )
 ]
 RETURN datatype
 { IS | AS }
 [ declare_section ]
 BEGIN
 	statement [ statement | pragma ]...
 	[ EXCEPTION exception_handler [ expcetion_handler ]...]
 END [ name ];
 
【语法说明】
[OR REPLACE]:覆盖同名函数。
FUNCTION:关键字，表示创建的是函数。
schema:模式名称。
function_name:函数名称。
parameter_declaration:的数的参数。参数有IN、OUT、IN OUT三种类型。
RETURN datatype:表示函数的返回类型。
{IS|AS}:二选一。该项之后是PL/SQL块。
declare_section:语句块部分的变量声明。
第9行表示语句或子程序。
第10行表示异常处理部分。
```
{% endcode %}

函数的作用是计算数据，并返回结果，所以在PL/SQL块中至少有一个RETURN语句。函数不能用来操作数据库，这一点和SQL内置函数一样。在当前模式下创建函数需要有CREATE PROCEDURE系统权限。

函数利用RETURN可以返回一个参数，某种情况下需要得到函数内的多个数据，那么可以采用OUT类型参数的方法，使其满足需求。

{% code lineNumbers="true" %}
```sql
CREATE FUNCTION AVG_PRIC(V_PRIC IN OUT VARCHAR2)
RETURN NUMBER IS V_Q NUMBER;

BEGIN 
    SELECT 
        AVG(PRODUCTPRICE),MIN(QUANTITY) INTO V_Q,V_PRIC
    FROM PRODUCTINFO
    WHERE PRODUCTRICE > V_PRIC;
    RETURN V_Q;
END;
```
{% endcode %}

【调用函数】 函数执行成功后，在PL/SQL块内调用，只有这样才能获取OUT类型参数的值，否则只能得到利用RETURN返回的值。调用脚本如下:

{% code lineNumbers="true" %}
```sql
DECLARE
V_PRIC VARCHAR2(20) := 1500;  --指定的价格
V_AVG VARCHAR2(20);	      --平均数
BEGIN
    V_AVG := AVG_PRIC(V_PRIC); --调用函数，函数内部会把V_PRIC重新赋值
    DBMS_OUTPUT.PUT_LINE('平均数:' || V_AVG);
    DBMS_OUTPUT.PUT_LINE('最低价格:' || V_PRIC);
END;
```
{% endcode %}

#### 查看函数

函数一旦创建成功，就会存储在Oracle服务器中，随时可以调用，也可以查看具体脚本。对于当前用户所在模式，用户可以在数据字典USER\_PROCEDURES中查看其属性，在数据字典USER\_SOURCE中查看其源脚本。这两个数据字典属于视图，利用这两个视图不仅可以查看函数的相关信息。也可以查看存储过程的相关信息。除了这两个视图以外，也可以在数据字典视图DBA\_PROCEDURES和数据字典视图DBA\_SOURCE查看同样的信息。

查看已有函数名称的示例

{% code lineNumbers="true" %}
```sql
SELECT OBJECT_NAME,OBJECT_ID,OBJECT_TYPE
FROM USER_PROCEDURES
ORDER BY OBJECT_TYPE;
```
{% endcode %}

查看已有函数的源脚本

```sql
SELECT NAME,LINE,TEXT FROM USER_SOURCE WHERE NAME = 'AVG_PRIC';
```

#### 函数的修改、删除

当函数创建完成后，如果业务发生变化，那么函数脚本也需要修改。函数内容的修改需要REPLACE关键词。

函数的删除同样简单。下面是删除函数的语法:

{% code lineNumbers="true" %}
```sql
DROP FUNCTION [schema.]function

schema:函数所属的模式名称。
```
{% endcode %}

### 小结

PL/SQL是操作Oracle的基础，日常开发中也是最常使用的部分。本章利用大量的篇幅介绍了PL/SQL基础方面的知识。下面概括了本章的主要知识点:

* 什么是PL/SQL，PL/SQL的优势和PL/SQL块的结构。这是PL/SQL的基础，在这部分应当了解PL/SQL块分为声明部分、执行体部分和异常处理部分。
* PL/SQL中的变量和常量。介绍了变量和常量的类型。概括来说，变量类型可分为标量类型和复合类型。&#x20;
* 有关表达式在PL/SQL编程中或标准SQL语句中都有使用。
* PL/SQL既然是编程语言，那么它就有结构控制语句，这里所涉及的结构控制语句有IF语句、CASE语句和LOOP语句。这3种结构中都有分支结构，是PL/SQL编程必不可少的一项。&#x20;
* PL/SQL块中允许执行SELECT...INTO语句为变量赋值，也允许执行DML和DDL语句，不过执行DDL需要相关的命令。&#x20;
* PL/SQL中很重要的一部分就是异常处理。Oracle采用了统一处理异常的方式，当PL/SQL块中发生异常时，程序会立刻无条件地转到异常处理部分，然后与异常名称列表进行匹配，如果匹配成功则表示异常被捕捉到，否则异常会一直向外层传递，直到主机调用界面。&#x20;
* 利用PL/SQL可以自定义函数，函数允许查询数据，但却不允许对数据库进行操作，使用函数主要对数据项进行计算。

## 游标-数据缓存区

### 什么是游标

游标的使用可以让用户像操作数组一样操作查询出来的数据集，这使得使用PL/SOL编程更加方便。实际上，它提供了一种从集合性质的结果中提取单条记录的手段。

#### 游标的概念

可以将游标(Cursor)形象地看成一个变动的光标。它实际上是一个指针，它在一段Oracle存放数据查询结果集或数据操作结果集的内存中，这个指针可以指向结果集中的任何一条记录。这样就可以得到它所指向的数据了，但初始时它指向首记录。这种模型很像编程语言中的数组。&#x20;

可以简单地理解游标为指向结果集记录的指针，利用游标可以返回它当前指向的行记录(只能返回一行记录)。如果要返回多行，那么需要不断地滚动游标，把想要的数据查询一遍。用户可以操作游标所在位置行的记录。例如，把返回记录作为另一个查询的条件等。

#### 游标的种类

Oracle中游标分为静态游标和REF游标两类。其中，静态游标就像一个数据快照，打开游标后的结果集是对数据库数据的一个备份，数据不随着对表执行DML操作后而改变。从这个特性来说，结果集是静态的。由于篇幅问题，本章将不对REF游标做介绍。

静态游标包含如下两种类型:&#x20;

* 显式游标：是指在使用之前必须有着明确的游标声明和定义，这样的游标定义会关联数据查询语句，通常会返回一行或多行。打开游标后，用户可以利用游标的位置对结果集进行检索，使之返回单一的行记录，用户可以操作此记录。关闭游标后，就不能再对结果集进行任何操作。显式游标需要用户自己写代码完成，一切由用户控制。
* 隐式游标：和显式游标不同，它被PL/SQL自动管理，也被称为SQL游标。由Oracle自动管理。该游标用户无法控制，但能得到它的属性信息。

### 显示游标

显式游标在PL/SQL编程当中有着重要的作用，通过显式游标用户可以操作返回的数据使得一些在编程语言中复杂的功能变得更容易实现。

#### 游标语法

既然要学习显式游标，就要学习创建它的语句。具体语法如下：

```plsql
CURSOR cursor_name
    [(parameter_name datatype, ...)]
    IS select_statement;

【语法说明】
CURSOR cursor_name:声明游标，cursor_name是游标的名称。
parameter_name:参数名称。
datatype:参数类型。
select_statement:游标关联的SELECT语句，但该语句不能是SELECT...INTO...语句。
```

#### 游标的使用步骤

显式游标的使用顺序可以明确地分成声明游标、打开游标、读取数据和关闭游标4个步骤下面就具体学习这4个步骤。

**1.声明游标**

声明游标主要用来给游标命名并且使得游标关联一个查询。具体语句如下:&#x20;

```plsql
DECLARE CURSOR cursor_name 
IS SELECT_STATEMENT
```

**2.打开游标**

游标中任何对数据的操作都是建立在游标被打开的前提下。打开游标初始化了游标指针游标一旦打开，其结果集都是静态的。也就是说，结果集此时不会反映出数据库中对数据进行的增加、删除、修改操作。具体语句如下:&#x20;

```plsql
OPEN cursor_name
```

**3.读取数据**

读取数据要利用FETCH语句完成，它可以把游标指向位置的记录放入到PL/SQL声明的变量当中。它只能取出指针当前行的记录。正常情况下，FETCH要和循环语句一起使用，这样指针会不断前进，直到某个条件不符合要求而退出。使用FETCH时游标属性%ROWCOUNT会不断累加。具体语句如下:&#x20;

```sql
FETCH cursor_name INTO record_name
```

**4.关闭游标**

关闭某个名称的游标。此时释放资源，结果集中的数据将不能做任何操作。具体语句如下:&#x20;

```plsql
CLOSE cursor_name
```

* 【示例1】创建一个简单游标并使用游标

<pre class="language-plsql" data-line-numbers><code class="lang-plsql"><strong>DECLARE
</strong>    CURSOR cur
    IS SELECT * FROM PRODUCTINFO;
    cur_p productinfo%ROWTYPE;
BEGIN
    OPEN cur
        FETCH cur INFO cur_p;
        DBMS_OUTPUT.PUT_LINE(cur_p.id||'-'||cur_p.name
                                     ||'-'||cur_p.price);
    CLOSE cur;
END;

【代码解析】
第1~2行是游标的声明，声明了一个名为cur的游标。
第3行是游标关联的查询。
第5行表示声明了一个变量，该变量类型是基于表PRODUCTINFO的行对象。
第8行表示打开游标。
第9行表示利用FETCH语句从结果集中提取指针指向的当前行记录。
第10行表示输出结果并换行，脚本中输出3个字段的值。
第12行表示关闭游标。
</code></pre>

* 【示例2】创建游标并在游标中声明变量

{% code lineNumbers="true" %}
```plsql
DECLARE
cur_productid varchar(10);
cur_productname productinfo.Productnamet%TYPE;
cur_productprice productinfo.Productprices%TYPE;

CURSOR pdct_cur
IS SELECT PRODUCTID,PRODUCTNAME,PRODUCTPRICE FROM PRODUCTINFOWHERE
 WHERE ROWNUM = 1;
BEGIN
  OPEN pdct_cur;
    FETCH pdct_cur INTO cur_productid ,cur_productname ,cur_productprice ;
    DBMS_OUTPUT.PUT_LINE(cur_productid ||'-'||cur_productname||'-'||cur_productprice );
  CLOSE pdct_cur;
END;

【代码解析】
第2行表示声明一个变量，名称为cur_productid，类型是长度为10的varchar类型，此变量中存放的数据长度不得超过10。此种声明方式更多当做中间变量使用。
第3~4行也是声明变量，实现的功能和第2行一样，但有一点区别。第3~4行的变量类型分别和表PRODUCTINFO中PRODUCTNAME、PRODUCTPRICE字段类型一样。
  这样声明有个优点，那就是如果表PRODUCTINFO中的这两个字段类型改变了，那么游标里声明的这两个变量类型可以不用修改。第2行中的声明方法就没有这种效果。
第6行是声明出来的游标名称。
第7行是游标关联的查询语句，表示查询PRODUCTINFO表中PRODUCTID、PRODUCTNAME、PRODUCTPRICE这3个字段。
第10行和第14行对应，可以保证代码的完整性。
第11行表示打开游标。
第12行表示利用FETCH从结果集中提取数据放到3个变量中。注意:这里给变量赋值的顺序和游标关联查询字段的顺序是一致的。第13行表示输出cur_productid变量里的内容并换行。
第14行表示关闭游标。
```
{% endcode %}

{% hint style="info" %}
**%TYPE** 类型用在变量的声明里，使用它可以取得表中的字段类型。&#x20;

**%ROWTYPE** 行类型可以声明基于某个表的行类型。
{% endhint %}

#### 游标中的LOOP语句

{% code lineNumbers="true" %}
```plsql
DECLARE
CURSOR pdct_loop_cur
IS SELECT PRODUCTID,PRODUCTNAME,PRODUCTPRICE FROM PRODUCTINFO
WHERE PRODUCTPRICE>2500;

cur_productid productinfo.Productid%TYPE;
cur_productname productinfo.Productname%TYPE;
cur_productprice productinfo.Productpricet%TYPE;

BEGIN
OPEN pdct_loop_cur;
    LOOP
        FETCH pdct_loop_cur INTO cur_productid,cur_productname,cur_productprice;
        EXIT WHEN pdct_loop_cur%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE('产品ID:'||cur_productid||'产品名称:'||
            cur_productname||'产品价格:'||cur_productprice);
    END LOOP;
CLOSE pdct_loop_cur;
END ;

【代码解析】
代码第1~4行表示声明游标并关联查询，查询出价格大于2500的产品。
第6~8行声明变量，各变量类型同表PRODUCTINFO的对应字段类型一致。
第12~18行就是使用的LOOP语句，利用该语句可以遍历结果集。
第13行利用FETCH...INTO...语句把取出的值放进变量中。
第14行利用游标属性实现没有记录退出循环。
第15~17行输出信息。
第19行关闭游标。
```
{% endcode %}

#### 使用BULK COLLECT和FOR语句游标

游标中通常使用FETCH...INTO..语句提取数据，这种方式是单条数据提取，在数据量很大的情况下执行效率不是很理想。而FETCH..BULKCOLLECTINTO语句可以批量提取数据在数据量大的情况下它的执行效率比单条提取数据的高。

此语句的用法可以参考如下示例。

<pre class="language-plsql" data-line-numbers><code class="lang-plsql">DECLARE
CURSOR pdct_collect_cur
IS SELECT * FROM PRODUCTINFO;

TYPE PDCT_TAB IS TABLE OF PRODUCTINFO%ROWTYPE;
pdct_rd PDCT_TAB;

BEGIN
    OPEN pdct_collect_cur;
    LOOP
    
        FETCH pdct_collect_cur BULK COLLECT INTO pdct_rd LIMIT 2
        FOR i in 1..pdct_rd.count LOOP
            DBMS_OUTPUT.PUT_LINE('产品ID:'||pdct_rd(i)productid
            ||'产品名称:'||pdct_rd(i).productname
            ||'产品价格:'||pdct_rd(i).productprice);
        END LOOP;
        EXIT WHEN pdct_collect_cur%NOTFOUND;
        
    END LOOP;
CLOSE pdct_collect_cur;
<strong>END ;
</strong>
【代码解析】
第1~3行声明游标名称，关联查询语句。
第5~6行表示的是定义和表PRODUCTINFO行对象一致的集合类型pdct_rd，该变量用于存放批量得到的数据。
第10行和第20行对应，这里是迭代从结果集取数据。
第12行表示从结果集批量提取数据，每次提取两条。
第13行表示遍历集合对象pdct_rd中的数据，该行的LOOP与第17行对应
第14~16行表示输出结果。
第18行判断游标是否到最尾端
</code></pre>

#### 使用CURSOR FOR LOOP

游标很多机会都是选代结果集，在PL/SQL这个过程中可以使用更简单的方式实现，CURSOR FOR LOOP不需要特别的声明变量，它可以提出行对象类型的数据。

{% code lineNumbers="true" %}
```plsql
DECLARE
    CURSOR cfl IS SELECT productname,productprice FROM PRODUCTINFO 
    WHERE productprice>1200;
BEGIN
    FOR curcfl IN cfl
    LOOP
    DBMS OUTPUT.PUT I('名称:'||curcfl.productname
                    ||'产品价格:'||curcfl.productprice);
END LOOP;
END ;

【代码解析】
第1-3行声明游标并关联查询。
第5行把游标返回数据放到curcfl中，该类型是个%ROWTYPE类型。
第6-9行迭代输出数据。
```
{% endcode %}

{% hint style="info" %}
这种方式在隐式游标中使用更显方便，如果没有特殊需求，可以尝试使用该语句。
{% endhint %}

#### 显式游标的属性

利用游标属性可以得到游标执行的相关信息。显式游标有以下4个属性:

* %ISOPEN:用于判断游标是否打开，如果已经打开则返回TRUE，如果游标未打开则返回FALSE。&#x20;
* %FOUND:此属性可用来检测行数据是否有效。如果有效该属性返回TRUE，否则返回FALSE。
* %NOTFOUND:与%FOUND属性恰好相反，如果没有提取出数据则返回TRUE，否则返回FALSE。&#x20;
* %ROWCOUNT:累计到当前为止使用FETCH提取数据的行数。

#### 带参数的游标

在使用显式游标时是可以指定参数的，指定的参数包括参数的顺序和参数的类型。参数可以传递给游标在查询中使用，这样就方便了用户根据不同的查询条件进行查询，也方便了游标在存储过程中的使用。

{% code lineNumbers="true" %}
```plsql
DECLARE
    cur_productid productinfo.Productid%TYPE := '0240';
    cur_productprice productinfo.Productprice%TYPE := 1200;
    cur_prodrcd productinfo%ROWTYPE;
    
    CURSOR pdct_parameter_cur (id VARCHAR,price NUMBER)
    IS SELECT * FROM PRODUCTINFO
    WHERE productid like id || '%'
        AND productprice > price;
BEGIN
    OPEN pdct_parameter_cur(cur_productid,cur_productprice);
        LOOP
            FETCH pdct_parameter_cur INTO cur_prodrcd;
            EXIT WHEN pdct_parameter_cur%NOTFOUND;
            DBMS_OUTPUT.PUT_LINE('产品ID:'||cur_prodrcd.productid
                || '产品名称:'||cur_prodrcd.productname || '产品价格：'
                || cur_prodrcd.productprice);
        EXIT LOOP;
    CLOSE pdct_parameter_cur;
END;

【代码解析】
第2~3行声明变量并赋值，这两个参数是要传递给游标的变量。
第4行声明表productinfo的行对象，用来存放FETCH提取的数据。
第6行声明游标，包括两个参数，参数需要说明类型。
第7~9行是游标关联的查询语句，可以看到第8和第9行的查询条件都使用了游标里的变量。
第12行表示打开游标，并把第2和第3行的变量传人游标中。
```
{% endcode %}

### 隐式游标

隐式游标和显式游标有所差异，它虽然没有显式游标一样的可操作性，但在实际的工作中也经常用到。

#### 隐式游标的特点

每当运行SELECT或DML语句时，PL/SQL会打开一个隐式的游标。隐式游标不受用户的控制，这一点和显式游标有明显的不同。下面列出了隐式游标和显式游标的不同处:&#x20;

* 隐式游标由PL/SQL自动管理;&#x20;
* 隐式游标的默认名称是SQL;&#x20;
* SELECT或DML操作产生隐式游标: 口隐式游标的属性值始终是最新执行的SQL语句的。

{% code lineNumbers="true" %}
```plsql
DECLARE
    c_pn productinfo.Productname%TYPE;
    c_pp productinfo.Productprice%TYPE;
BEGIN
    SELECT productname,productprice INTO c_pn,c_pp
    FROM PRODUCTINFO
    WHERE productid = '1';
    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('名称'||c_pn
            ||'价格'||c_pp);
    END IF;
END;

【代码解析】
第1~3行声明变量，这部分和显式游标没有区别。
第5~7行把查询的数据放进两个变量中。
第8行利用游标属性进入条件语句。关于隐式游标的属性后面会单独做介绍。
第9~10行输出结果。
第11行为IF语句的介绍标志。
第12行是BEGIN结束的标志。
```
{% endcode %}

{% hint style="info" %}
此示例演示了如何使用隐式游标。通过脚本可以看出，隐式游标没有像显式游标那样声明一个游标名称，而是直接使用了SQL名称。这是隐式游标的默认名称，可以直接使用。
{% endhint %}

#### 游标中使用异常处理

使用游标时，某些情况下得到的数据超出了控制范围，如果不加处理会出现脚本执行中断的情况。这种情况下，脚本开发者通常会使用异常处理来维护脚本的稳定性。

{% code lineNumbers="true" %}
```plsql
DECLARE
    c_pn productinfo.Productname%TYPE;
    c_pp productinfo.Productprice%TYPE;
BEGIN
    SELECT productname,productprice INTO c_pn,c_pp
    FROM PRODUCTINFO
    WHERE productid = '-1';
    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('名称'||c_pn
            ||'价格'||c_pp);
    END IF;
    EXCEPTION 
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('没有数据')
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('数据过多')
END;
```
{% endcode %}

{% hint style="info" %}
通过异常处理可以有效避免意外引起的脚本错误，在使用游标时需要注意这一点。
{% endhint %}

#### 隐式游标的属性

隐式游标的属性和显式游标的属性具体表示含义有区别，但属性种类没有变。

下面列出隐式游标的属性。&#x20;

* %ISOPEN属性:该属性永远返回FALSE，它由Oracle自己控制。
* %FOUND属性:此属性可以反应DML操作是否影响到了数据，当DML操作对数据有影响时该属性为TRUE，否则为FALSE。也可以反映出SELECT INTO语句是否返回了数据，当有数据返回时该属性为TURE。&#x20;
* %NOTFOUND属性:与%FOUND属性相反，当DML操作没有影响数据以及SELECTINTO没有返回数据时该属性为TRUE，其他为FALSE。
* %ROWCOUNT属性:该属性可以反映出DML操作对数据影响的数量。

{% hint style="info" %}
在SELECT INTO语句中%FOUND不会因语句是否发生异常而改变，只要有返回值该属性就为TRUE。但有异常发生时，执行流程会马上发生改变。

与显式游标不同的是隐式游标中%ROWCOUNT属性反映了DML操作影响的数据数量，而SELECT INTO语句如果发生TOO\_MANY\_ROWS异常，那么此属性依然是1，而不是实际符合要求的记录数。

%ROWCOUNT属性和事务没有关系，即使事务回滚，它的值也不会变成上次操作的值。
{% endhint %}

### 小结

游标是提取Oracle数据的重要手段，它可以把符合要求的数据检索出来并存储到缓冲区中利用循环语句遍历整个结果集，从而获取每条记录，开发人员可以对每条记录进行操作。游标有4个属性，可以通过这4个属性得到游标的执行信息，也可以利用这些信息进行流程控制。游标中也允许异常处理，这和PL/SQL块一致。

## 视图-虚拟表

视图在Oracle中应用相当普遍，所以也比较重要。视图在数据库中可以理解为一张虚拟的表，使用视图可以补充表结构在某些需求方面的不足,可以让开发人员更方便地查询复杂数据还可以缩短开发周期，节省公司成本。

### 什么是试图

#### 认识试图

根据官方的文档可以这样理解视图:它是一个基于一个表或多个表的逻辑表，视图本身不包含任何数据。通俗来说，可以把视图看成是虚拟的表，只是一个查询语句的结果，它的数据最终是从表中获取的，这些表通常称为源表或基表。当基表的数据发生变化时，视图里的数据同样发生变化。通常视图的数据源有下面三种情况:&#x20;

* 单一表的子集。&#x20;
* 多表操作结果集。&#x20;
* 视图的子集。

#### 试图的作用

* 使数据简化。在表中很多数据对业务来说是冗余的，这时开发者会使用比较复杂的SQL语句得到自己想要的。实际开发中不能要求每个人都能做到这一点，所以，通常情况下由一个人把该复杂语句做成视图，其他人员直接调用该视图即可。这样对视图使用人员就简化了数据，隐藏了数据的复杂性。
* 使数据更加独立。程序开发时，大多数是程序直接访问数据库的表，当这些表的结构随着业务的变化而不得不重新设计时会影响到程序(通常表一旦设计完成就很难再做修改)，所以可以使得程序直接访问视图。这样视图就可以把程序和数据库的表隔离开来，降低开发者的劳动成本。
* 增加安全性。视图可以查询表指定的列来展现给用户，而不必让使用者完全看见表的所有字段。这种情况很多是一个公司提供给其他合作伙伴查询数据的接口，而视图通常也会设成只读属性。

#### 试图的语法

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

### 试图的创建

#### 创建单表试图

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

#### 创建多表试图

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

### 操作试图数据的限制

视图允许做DML操作，但需要注意的地方比较多。因为视图增加或更新数据实际上是在操作视图的源表。除此之外，视图本身可以设置更新限制条件。

#### 试图READ ONLY设置

创建视图时为了避免用户修改数据，可以把视图设成只读属性，其操作比较简单

```sql
CREATE OR REPLACE VIEW VIEW_NAME AS
SELECT ID,NAME FROM TABLE_NAME
WITH READ ONLY;
```

{% hint style="info" %}
当插入或修改视图数据时，会提示“无法对只读视图执行DML操作"。
{% endhint %}

#### 试图CHECK OPTION设置

在某些情况下允许修改视图的数据，修改数据的本质是修改视图源表的数据。假如某个视图查询出来的是年龄大于20的所有数据，如果为该视图增加一条年龄为10的记录，那么该记录将不会出现在视图中。显然这是不符合逻辑的。为了避免这种情况的发生，可以利用CHECK OPTION选项来设置视图的检查约束。

CHECK OPTION选项表示视图启动了和子查询条件一样的约束。也就是说，如果对视图修改或插入的数据和查询条件不一致，那么该操作会被中止。

* 增加数据\
  新增的数据与试图查询条件不一致，会提示错误信息
* 修改数据\
  修改的数据与试图查询条件不一致，也会提示错误信息
* 删除数据\
  删除的数据与试图查询条件不一致；脚本执行后并没有出现错误提示，但提示0行被操作。这不但说明了视图过滤出来的数据和源表的数据在逻辑上彻底分离，也说明了CHECK OPTION项对删除没有作用(如果有作用，删除语句将无法执行)。

#### 试图创建语句对试图操作的影响

如果想要一个可以更新(这里的更新是指增加、删除、修改)的视图，源表应尽量是单表，否则限制比较多。下面的情况一旦出现在视图中，视图就不允许更新。&#x20;

* DISTINCT关键字。&#x20;
* 集合运算或分组函数，如INTERSECT、SUM、MAX、COUNT等函数。&#x20;
* 出现GROUP BY、ORDER BY、MODEL、START WITH等语句。&#x20;
* 出现伪列关键字，如ROWNUM。

除了以上情况外,还需要考虑基表的一些约束,这些约束对视图数据的更新都有一定影响。 如果需要创建可以更新的视图，可以使用INSTEAD OF触发器。

### 试图的修改

视图的修改比较特殊，不能像表一样修改，更准确地说它没有修改选项，可以覆盖原有视图，但这不会影响视图的使用。因为视图本身不包含数据，所以覆盖原有视图时就不存在数据丢失的问题。

#### 试图约束的修改

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

#### 试图的删除

视图删除和表删除操作方式一样，可以使用SQL语句删除，也可以使用PL/SQLDeveloper工具删除。在本节中就将分别讲述使用这两种方式删除视图的方法。

```sql
DROP VIEW [schema.]view [CASCADE CONSTRAINTS]

【语法说明】
CASCADE CONSTRAINTS:删除视图时删除约束。
```

### 小结

在Oracle中经常使用视图，视图可以根据业务的需要从不同的角度展示数据，它给数据库管理者和用户都提供了一定的便捷性，也提高了数据的安全性。视图本身不包含任何数据，它只是一个查询，所有的数据都是从其他表或视图中获取的，但数据有着逻辑独立性。

## 存储过程-提高程序执行效率

### 什么是存储过程

存储过程在数据库开发中使用比较频繁，它有着普通SQL语句不可替代的作用。

#### 认识存储过程

所谓存储过程，就是一段存储在数据库中执行某种功能的程序，其中包含一条或多条SQL语句，但是它的定义方式和PL/SQL中的块、包等有所区别。存储过程可以通俗地理解为是存储在数据库服务器中的封装了一段或多段SQL语句的PL/SQL代码块。在数据库中有一些是系统默认的存储过程，那么可以直接通过存储过程的名称进行调用。另外，存储过程还可以在编程语言中调用，如Java、C#、VB等编程语言。

#### 存储过程的作用

存储过程编写相对比较复杂，但很多单位或个人都在使用它。显然这不是因为存储过程编写简单，而是因为它有着一系列的优点:&#x20;

* 简化复杂的操作。存储过程可以把需要执行的多条SQL语句封装到一个独立单元中，用户只需调用这个单元就能达到目的。这样就实现了一人编写多人调用，同时缩短了平均开发周期，为公司节省了成本。&#x20;
* 增加数据独立性。与视图的效果类似，利用存储过程可以把数据库基础数据和程序(或用户)隔离开来，当基础数据的结构发生变化时，可以修改存储过程，这样对程序来说基础数据的变化是不可见的，也就不需要修改程序代码了。
* 提高安全性。使用存储过程有效地降低了错误出现的几率。如果不使用存储过程要想实现某项操作可能需要执行多条单独的SQL语句，而过多的执行步骤很可能造成更高的出错几率。不仅如此，实际工作中开发人员的水平参差不齐，由高水平的人编写存储过程，水平较低的人员直接调用，这样就能避免很多不必要的错误发生。此外，存储过程也可以进行权限设置。
* 提高性能。完成一项复杂的功能可能需要多条SQL语句，同时SQL每次执行都需要编译而存储过程可以包含多条SQL语句，而且创建完成后只需要编译一次，以后就可以直接调用，从这方面来看存储过程可以提高性能。如果程序语言要实现某项比较复杂的功能，它会多次连接数据库，在使用存储过程的情况下，程序只需连接一次就能达到目的。

#### 存储过程的语法

```sql
CREATE [ OR REPLACE ] PROCEDURE [schema.]procedure_name
[ parameter_name 
    [ 
        [ IN ] datatype [ { := | DEFAULT } expression ] 
        | { OUT | IN OUT } [ NOCOPY ] datatype     
    ] [,...] 
]
{ IS | AS }
BODY ;

OR REPLACE:表示如果指定的过程已经存在，则覆盖同名的存储过程。
schema:表示该存储过程的所属机构。
procedure_name:创建存储过程的名称。
parameter_name:表示存储过程中的参数名称。
[IN]datatype[{:= | DEFAULT} expression]:整个这段语法表示传入参数的数据类型以及默认值。
    其中，datatype项表示参数的数据类型，[{:=|DEFAULT}expression]项表示参数的默认值的写法。
{OUTIIN OUT}[NOCOPY]datatype:表示存储过程的参数类型，不过和上面介绍的IN有所区别。
    其中，OUT表示输出参数，INOUT表示既可输入也可输出的参数，datatype依旧表示参数类型。
{IS|AS}:连接词。
BODY:表示函数体，是存储过程的具体操作部分，通常在begin...end中。
```

{% hint style="info" %}
存储过程的参数默认类型是IN型的，也就是说是传入型的。当前模式下创建存储过程需要有CREATE PROCEDURE权限。
{% endhint %}

### 创建存储过程

#### 创建第一个存储过程

```sql
CREATE PROCEDURE TEST
AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('我的第一个过程!');
END;
```

执行存储过程

```sql
BEGIN
    TEST;
END;
```

#### 查看存储过程

存储过程一旦被创建就会存储到数据库服务器上，Oracle允许开发人员查看已经存在的存储过程脚本，这可以到视图USER\_SOURCE里查看。

```sql
SELECT * FROM USER_SOURCE WHERE NAME = "TEST" ORDER BY LINE;>
```

{% hint style="info" %}
当从视图USER\_SOURCE中查询过程或函数时需要把名称大写，小写会得不到想要的记录。至于USER\_SOURCE是当前用户的视图，如果想查看Oracle所有的存储过程则需要到ALL\_SOURCE视图中查询。
{% endhint %}

#### 显示存储过程的错误

```sql
SHOW ERRORS PROCEDURE procedure_name;
```

#### 无参存储过程

<pre class="language-sql"><code class="lang-sql">CREATE PROCEDURE PRODUCT_UPDATE_PRC
AS
BEGIN
<strong>    UPDATE PRODUCTINFO SET DESC = '促销产品' WHERE ID = '1';
</strong>    COMMIT;
END;
</code></pre>

#### 存储过程中使用游标

<pre class="language-sql"><code class="lang-sql">CREATE PROCEDURE TEST
AS
<strong>c_name productinfo.NAME%TYPE;
</strong>
BEGIN
    FOR res IN (SELECT * FROM PRODUCTINFO WHERE NAME = c_name)
    LOOP
        DBMS_OUTPUT.PUT_LINE(res.name||':'||res.price);
    END LOOP;
END;
</code></pre>

#### 存储过程中的DDL语句

开发过程中为了让数据操作起来更方便会用到临时表，而为了让存储过程更具有通用性，可以选择把创建临时表的步骤一并放到过程里。这样的操作会和前面介绍的两种示例写法有所不同，它会用到EXECUTE IMMEDIATE语句，存储过程中会使用它来执行DDL语句和动态SQL语句。

```sql
CREATE PROCEDURE TEST
AS
p_d VARCHAR2(50); --删除临时表
p_cr VARCHAR2(200);--创建临时表
p_c VARCHAR2(10); --临时表数量

BEGIN
    SELECT count(1) into p_count 
    FROM ALL_TABLES 
    WHERE TABLE_NAME = 'TEST';
    
    p_d := 'DELETE FROM TEST';
    p_cr := 'CREATE GLOBAL TEMPORARY TABLE TEST(
        ID VARCHAR2(32) NOT NULL,
        NAME VARCHAR2(200) NOT NULL
    ) ON COMMIT PRESERVE ROWS';
    
    if p_c=0 then
        EXECUTE IMMEDIATE p_cr;
        DBMS_OUTPUT.PUT_LINE('创建临时表成功');
    ELSE
        EXECUTE IMMEDIATE p_d;
        DBMS_OUTPUT.PUT_LINE('删除临时表成功');
    END IF;
END;
```

#### 有参存储过程

{% tabs %}
{% tab title="IN参数" %}
```sql
CREATE PROCEDURE TEST(name IN VARCHAR2)
AS
BEGIN
    DBMS_OUTPUT.PUT_LINE(name);
END;
```
{% endtab %}

{% tab title="默认参数" %}
```sql
CREATE OR REPLACE FUNCTION DEFT RETURN VARCHAR2
IS 
BEGIN
    RETURN '雨具';
END;
```

```sql
CREATE PROCEDURE TEST(name IN VARCHAR2 DEFAULT DEPT())
AS
BEGIN
    DBMS_OUTPUT.PUT_LINE(name);
END;
```
{% endtab %}

{% tab title="OUT参数" %}
```sql
CREATE PROCEDURE TEST(name OUT VARCHAR2)
AS
BEGIN
    SELECT '测试' INTO name FROM dual; 
END;
```
{% endtab %}

{% tab title="IN OUT参数" %}
```sql
CREATE PROCEDURE TEST(vname IN OUT VARCHAR2)
AS
BEGIN
    SELECT address FROM USER WHERE NAME = vname;
    IF SQL%FOUND THEN
        vname := SQL%ROWCOUNT;
    END IF;
END;
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
给存储过程的参数赋值时，除了以上接触到的按照存储过程的参数顺序赋值之外，还有另外一种赋值的方式。这种方式可以看做显式地为某个变量赋值，它指定了赋值的参数对象，因此这种方式不用考虑原存储过程中参数的顺序。

`TEST(parm2=>pric,parm1=>name);`
{% endhint %}

### 修改、删除存储过程

#### 修改存储过程

修改存储过程内容要利用REPLACE关键词，从而完成过程的修改也就是覆盖。

```sql
CREATE OR REPLACE PROCEDURE procedure_name
IS 
BEGIN
    ...
END;
```

#### 删除存储过程

```sql
DROP PROCEDURE [schema.]procedure_name

【说明】
schema:表示存储过程的所属机构。
procedure_name:表示要删除存储过程的名称
```

{% hint style="warning" %}
当存储过程有调用关系时，如果删除被调用者、那么重新编译调用者会出现错误。所以做删除操作时最好认清各过程之间的依赖关系。
{% endhint %}

### 小结

存储过程的使用十分普遍，它用做批量处理任务和内部数据转换时效率很高，也是Oracle的重点之一。存储过程的使用方式灵活多变，读者在实际开发中应多读代码学习不同的使用方式，达到融会贯通的目的。

## 触发器-保证数据的正确性

### 什么是触发器

#### 认识触发器

前面介绍过存储过程。其实触发器和存储过程比较类似，它由PL/SQL编写并存储在数据库中，它可以调用存储过程，但触发器本身的调用和存储过程调用却是不一样的。存储过程由用户、应用程序、触发器或其他过程调用。但触发器只能由数据库的特定事件来触发。所谓的特定事件主要包括如下几种类型的事件。

&#x20;用户在指定的表或视图中做DML操作，主要包括如下几种:&#x20;

* INSERT操作，在特定的表或视图中增加数据&#x20;
* UPDATE操作，对特定的表或视图修改数据。&#x20;
* DELETE操作，删除特定表或视图的数据，&#x20;

用户做DDL操作，主要包括如下几种:&#x20;

* CREATE操作，创建对象。&#x20;
* ALTER操作，修改对象。&#x20;
* DROP操作，删除对象。&#x20;

数据库事件，主要包括如下几种:&#x20;

* LOGON/LOGOFF用户的登录或注销。&#x20;
* STARTUP/SHUTDOWN数据库的打开或关闭。&#x20;
* ERRORS特定的错误消息等。

&#x20;在以上事件中的一种或多种发生时就能使触发器运行。

#### 触发器的作用

触发器可以根据不同的事件进行调用，它有着更加精细的控制能力，这种特性可以帮助开发人员完成很多普通PL/SQL语句完成不了的功能。下面介绍一下触发器的主要作用。

* 自动生成自增长字段。例如，在表中插入数据前得到序列的最大值和数据同时插入表中，避免该序列的重复。&#x20;
* 执行更复杂的业务逻辑。普通的操作方式只能完成固定的数据变动，而使用触发器则在完成的基础功能上做额外的操作，以达到完成特殊业务的目的。
* 防止无意义的数据操作。利用触发器可以把符合某些条件的数据加以限制，使其不能变动。
* 提供审计。利用触发器可以跟踪对数据库的操作，也可以在指定的表或视图记录改变时，利用触发器把数据变动日志记录下来。
* 允许或限制修改某些表。利用触发器可以限制表的变动。
* 实现完整性规则。当一个表中的数据有变动时可以利用触发器修改这些变动数据在其他表中的关联数据(正常情况下可以利用外键进行限制)。
* 保证数据的同步复制。

{% hint style="warning" %}
建议开发人员只在必要时使用触发器，因为触发器可能造成比较复杂的相关依性，这种情况在大型的数据库中可能会带来麻烦。例如，某个触发器的触发很可能造成多个触发器的连锁触发，一旦这种连锁触发超过32个就会出现异常。
{% endhint %}

#### 触发器的类型

触发器可分为5种类型，具体内容如下:&#x20;

* 数据操纵语言(DML)触发器。\
  此种类型触发器定义到表上，当对表执行INSERTUPDATE、DELETE操作时可以激发该类型的触发器。利用该类触发器可以复制、检查替换某种符合指定条件的数据。按照触发级别可以分为两种方式，第一种为行级触发器，此种类型表示每条记录修改时都会激发该触发器，第二种为语句级触发器，此种类型表示当SQL语句执行时会激发该触发器，与修改多少条记录没有关系。如果以数据的更改事件为准，则分为BEFORE和AFTER两种类型。
* 数据定义语言(DDL)触发器。\
  当CREATE、ALTER、DROP模式对象时会触发相关的触发器，在Oracle中可以简单地理解一个用户就有一个和它同名的模式，利用它可以使得某些表不能被修改或删除。 口复合触发器。此种类型的触发器是Oracle1lg的新特性，它相当于在一个触发器中包含了4种类型的触发器，其中包含了BEFORE类型的语句级、BEFORE类型的行级AFTER类型的语句级、AFTER类型的行级。这种把所有触发器都放到一个代码块中的 做法使得变量的传递变得更加方便。
* INSTEADOF触发器。\
  此种类型触发器通常作用在视图上。对由多个源表的视图做DML操作通常是不被允许的，如果遇到这种情况就可以利用INSTEAD OF类型触发器解决问题。利用它可以把对视图的DML操作转换成对多个源表进行操作。
* 用户和系统事件触发器。\
  作用在数据库上由数据库事件激发的触发器，如登录和注销事件的触发器。利用它可以记录数据库的登录情况。

{% hint style="info" %}
有些地方把DML和DDL类型的触发器直接分为行触发器、语句触发器、BEFORE触发器、AFTER触发器。
{% endhint %}

#### 触发器的语法

了解语法是使用触发器的第一步，它就像一个公式,能让我们迅速地创建出自己的触发器。 下面列出了几种触发器的语法。

> DML触发器的主要语法如下:

{% code lineNumbers="true" %}
```sql
CREATE [OR REPLACE] TRIGGER [schema.]trigger
{BEFORE | AFTER | INSTEAD OF}
  {DELETE | INSERT | UPDATE
    [OF column [, column] ...]
  }
  [OR {DELETE | INSERT | UPDATE [OF column [, column] ...]} ]...
{ON [schema.]table | {schema.}view}
  [FOR EACH ROW]
  [FOLLOWS [schema.]trigger [, [schema.]trigger]...]
  [ENABLE|DISABLE]
  [WHEN (condition)]
  trigger_body
```
{% endcode %}

{% code title=" 【语法说明】" overflow="wrap" lineNumbers="true" %}
```
OR REPLACE:新建的触发器可以覆盖原有同名触发器。
TRIGGER:创建触发器的关键词。
schema:触发器所属模式(可简单看成用户名)，如果不加该项则表示该触发器属于自己。
BEFORE:触发器类型为前触发。
AFTER:触发器类型为后触发。
INSTEAD OF:表示触发器类型为替换类型。
DELETE|INSERT|UPDATE:表示触发的事件。
[OF column[,column]:触发条件具体到的某列,可以追加多个条件。
ON[schema.]table | [schema.]view:该触发器作用的表或视图，INSTEAD OF类型可以作用在视图上。
FOR EACH ROW:表示行级触发器，省略则为语句级触发器。
FOLLOWS【schema.]trigger:触发器执行的顺序。
ENABLE|DISABLE:设置触发器是否可用状态。
WHEN(condition):触发该触发器的条件。
trigger_body:表示触发器的函数体。
```
{% endcode %}

> DDL和数据库事件触发器语法如下:

```sql
CREATE [OR REPLACE] TRIGGER [schema.]trigger
  {BEFORE | AFTER}
  {ddl_event [OR ddl_event]... | database_event [OR database_event]...}
ON { [schema.]SCHEMA | DATABASE}
  [FOLLOWS [schema.]trigger [,[schema.]trigger]...]
  [ENABLE|DISABLE]
  [WHEN (condition)]
  trigger_body
```

{% code title="【语法说明】" overflow="wrap" lineNumbers="true" %}
```
OR REPLACE:新建的触发器可以覆盖原有同名触发器。
TRIGGER:创建触发器的关键词。
schema:触发器所属模式，如果不加该项则表示该触发器属于自己。
BEFORE:触发器类型为前触发。
AFTER:触发器类型为后触发。
ddl_event [OR ddl_event]:DDL事件，用OR连接。
database_event[OR database_event]:数据库事件，用OR连接。
[schema.]SCHEMA|DATABASE:触发器可作用在模式上或数据库上。
FOLLOWS[schema.]trigger:触发器执行的顺序。
ENABLE|DISABLE:设置触发器是否可用状态。
WHEN(condition):触发该触发器的条件。
trigger_body:表示触发器的函数体。
```
{% endcode %}

> DDL事件列表

| DDL事件           | 简介                |
| --------------- | ----------------- |
| ALTER           | 修改对象，例如修改对象的名称约束等 |
| ANALYSE         | 用来分析统计信息          |
| AUDIT/ NO AUDIT | 启用或取消审计           |
| COMMENT         | 注解列或表的含义          |
| CREATE          | 创建对象              |
| DROP            | 删除对象              |
| GRANT           | 授权操作              |
| RENAME          | 修改对象名称            |
| TRUNCATE        | 取消权限              |
| REVOKE          | 删除行记录             |

触发器由三部分组成。它们分别是触发事件或语句、触发器限制、触发器动作。&#x20;

* 触发事件或语句是指激发触发器的动作。
* 触发器限制指的是WHEN后面的条件，当条件为TRUE时，该触发器会被激发。&#x20;
* 触发器动作就是一段过程，当触发器被激发时运行的trigger\_body部分。

> 数据库事件列表

| 数据库事件       | 简介               |
| ----------- | ---------------- |
| STARTUP     | 数据库打开后被触发，模式下不可以 |
| SHUTDOWN    | 数据库关闭前被触发，模式下不可以 |
| LOGON       | 客户程序登录后触发        |
| LOGOFF      | 客户程序注销前触发        |
| SERVERERROR | 错误消息出现后触发        |

> 复合触发器也是DML触发器的一种，他的主要语法如下

{% code lineNumbers="true" %}
```sql
CREATE [OR REPLACE] TRIGGER [schema.]trigger
FOR
{
    DELETE | INSERT | UPDATE [OF column [,column]...]
} [OR {DELETE | INSERT | UPDATE [OF column [,column]...]}]...
ON {
    [schema.]table | [schema.]view
}
COMPOUND TRIGGER
{  BEFORE STATEMENT IS tps_body END BEFORE STATEMENT
 | BEFORE EACH ROW IS tps_body END BEFORE EACH ROW
 | AFTER STATEMENT IS tps_body END AFTER STATEMENT
 | AFTER EACH ROW IS tps_body END AFTER EACH ROW
}
```
{% endcode %}

{% code title="【语法说明】" overflow="wrap" lineNumbers="true" %}
```
OR REPLACE:新建的触发器可以覆盖原有同名触发器。
TRIGGER:创建触发器的关键词。
schema:触发器所属模式，如果不加该项则表示该触发器属于自己。
DELETE|INSERT|UPDATE:表示触发事件。
COMPOUND TRIGGER:定义触发器时表示为复合类型触发器
BEFORE STATEMENT:前语句级触发。
BEFORE EACH ROW:前行级触发。
AFTER STATEMENT:后语句级触发。
AFTER EACH ROW:后行级触发。
tps_body:具体语句或程序。
```
{% endcode %}

{% hint style="info" %}
该类型是Oracle 11g新增加的触发器类型，比较方便地处理4个时间点。
{% endhint %}

### 操作触发器

#### 创建触发器

创建触发器的首要条件是要有相关的权限。用户模式下如果想在自己的对象上创建触发器，则必须具有CREATE TRIGGER系统权限，如果想在其他用户上创建触发器，则需要有CREATEANY TRIGGER权限。除此之外，如果在数据库上创建触发器，则需要有ADMINISTER DATABASE TRIGGER系统权限。

> 一个简单的触发器

```sql
CREATE TRIGGER FIRST_TGR
AFTER DELETE
ON PRODUCTINFO
BEGIN
    IF DELETING THEN
        DBMS_OUTPUT.put_line('删除数据操作!');
    END IF;
END;
```

{% code title="【代码解析】" overflow="wrap" lineNumbers="true" %}
```
创建名为FIRST_GR的触发器。
触发器类型为后触发，触发事件是删除操作。
触发器作用的表是PRODUCTINFO。一旦触发器创建成功，那么对表PRODUCTINFO执行删除操作则会激发
```
{% endcode %}

#### 查看触发器

> 查看已经存在的触发器的源代码，分为两个步骤。

1. 查看触发器的名称。执行如下脚本:&#x20;

```sql
SELECT OBJECT_NAME FROM USER_OBJECTS 
WHERE OBJECT_TYPE = 'TRIGGER';
```

2. 查看触发器内容。有了触发器名称就可以查看其具体内容，下面脚本中的FIRST\_TGR就是第一步查询出来的结果。执行如下脚本:&#x20;

```sql
SELECT * FROM USER_SOURCE
WHERE NAME='FIRST_TGR' ORDER BY LINE;
```

#### DML类型触发器

该类型触发器在日常开发中比较常用，前面已经介绍过DML触发器主要针对表的INSERT、UPDATE、DELETE操作。

> 创建行级触发器

创建操作事件记录表

```sql
CREATE TABLE LOG_TAB
(
    ID VARCHAR2(10) NOT NULL,
    OPER_TABLE VARCHAR2(20),
    OPER_KD VARCHAR2(10),
    OPER_TABLE_PRK VARCHAR2(50),
    OPER_DATE TIMESTAMP,
    constraint LOG_TAB_PRK primary key(ID)
);
```

创建用做LOG\_TAB表主键的自增长序列

```sql
CREATE SEQUENCE LOG_TAB_ID
MINVALUE 1000000000
MAXVALUE 9999999999
START WITH 1000000000
INCREMENT BY 1;
```

{% code title="【代码解析】" overflow="wrap" lineNumbers="true" %}
```
第1行表示创建名为LOG_TAB_ID的序列。
第2行表示该序列的最小值为1000000000。
第3行表示该序列的最大值为9999999999
第4行表示该序列的值从1000000000开始。
第5行表示该序列增长量为1。
```
{% endcode %}

创建触发器

```sql
CREATE OR REPLACE     TRIGGER PRODUCTINFO_OPER_TGR
    BEFORE INSERT
    ON PRODUCTINFO
    FOR EACH ROW
BEGIN
    IF INSERTING THEN
        INSERT INTO LOG_TAB
        VALUES(LOG_TAB_ID.NEXTVAL,'PRODUCTINFO','INSERT',:NEW.PRODUCTID,SYSDATE);
        DBMS_OUTPUT.PUT_LINE('插入数据主键是：'|| :new.PRODUCTID);
    END IF;
END;
```

{% code title="【代码解析】" overflow="wrap" lineNumbers="true" %}
```
创建名为PRODUCTINFO_OPER_TGR的触发器。
该触发器为前触发，触发事件是INSERT。
该触发器作用在表PRODUCTINFO上。
该触发器为行级触发，也就是说每增加一行就会触发一次。
判断如果是插入数据操作则进入IF语句。
向事件记录表中增加数据。
LOG_TAB_ID.NEXTVAL表示得到序列的下一个值。
SYSDATE表示增加数据时的系统时间。
输出增加数据的PRODUCTID字段值，该字段是'PRODUCTINFO'表的主键。
```
{% endcode %}

{% hint style="warning" %}
行级的触发器里使用:new或:old来访问变更前和变更后的数据。其中，如果是增加新记录操作，则只有:new可以访问;如果是修改操作，则:new和:old都可以访问，:new表示修改后的记录，:old表示修改前的记录，而删除则只有:old可以访问，因为该操作是删除已有的记录。
{% endhint %}

> 在触发器中使用多种触发事件

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_TGR
    BEFORE INSERT OR UPDATE OR DELETE
    ON PRODUCTINFO
    FOR EACH ROW
BEGIN
    CASE 
        WHEN INSERTING THEN --增加操作
            INSERT INTO LOG_TAB
            VALUES(LOG_TAB_ID.NEXTVAL,'PRODUCTINFO','INSERT',:NEW.PRODUCTID,SYSDATE);
            DBMS_OUTPUT.PUT_LINE('插入数据主键是：'|| :NEW.PRODUCTID);
        WHEN UPDATING THEN
            INSERT INTO LOG_TAB
            VALUES(LOG_TAB_ID.NEXTVAL,'PRODUCTINFO','UPDATE',:OLD.PRODUCTID,SYSDATE);
            DBMS_OUTPUT.PUT_LINE('修改数据主键是：'|| :OLD.PRODUCTID);
        WHEN DELETING THEN
            INSERT INTO LOG_TAB
            VALUES(LOG_TAB_ID.NEXTVAL,'PRODUCTINFO','DELETE',:OLD.PRODUCTID,SYSDATE);
            DBMS_OUTPUT.PUT_LINE('删除数据主键是：'|| :OLD.PRODUCTID);
    END CASE;
END;
```
{% endcode %}

> 在触发器中使用IF语句

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_TGR
   BEFORE UPDATE OF PRODUCTPRICE ON PRODUCTINFO
   FOR EACH ROW
BEGIN
   IF (TO_CHAR(SYSDATE,'dd') = 20) THEN
      RAISE_APPLICATION_ERROR(-20000,'今天不允许修改价格！');
   END IF;
   
   INSERT INTO LOG_TAB VALUES(LOG_TAB_ID.NEXTVAL,'PRODUCTINFO','UPDATE',:NEW.PRODUCTID,SYSDATE);
   DBMS_OUTPUT.PUT_LINE('修改数据主键是：'|| :NEW.PRODUCTID);
END;
```
{% endcode %}

> 使用WHEN限制条件的触发器

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_TGR
    BEFORE INSERT ON PRODUCTINFO
    FOR EACH ROW
    WHEN (NEW.PRODUCTID = '3')
BEGIN
    DBMS_OUTPUT.PUT_LINE('插入数据主键是：'|| :NEW.PRODUCTID);
END;
```
{% endcode %}

{% hint style="warning" %}
利用WHEN限制条件可以比较精确地限制触发器的激发条件，每当操作特定数据时可以考虑使用该条件限制，使得触发器更加灵活。需要说明的是，在WHEN条件中的:NEW或:OLD使用方式和通常使用方式上稍微有点差别，这里直接使用NEW或OLD关键词得到数据，没有前面的冒号。
{% endhint %}

> 语句级触发器

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_TGR
    BEFORE INSERT OR UPDATE OR DELETE ON PRODUCTINFO
BEGIN
    DBMS_OUTPUT.PUT_LINE('语句级触发器测试');
END;
```
{% endcode %}

#### 触发器执行顺序

在同一对象上可以作用多个触发器，因此触发器被激发的顺序是有先后关系的。其触发顺 序如下:

* 首先被触发的将是前语句级触发器(before statement trigger)，该触发器会被执行一次。
* 如果有行级的触发器则接下来执行前行级触发器(before row trigger)，行级触发器执行的次数同SOL修改的记录次数一致。&#x20;
* 当SQL修改记录完成后会触发行级触发器，这时候的行级触发器为后行级触发器(after row trigger)，该类型触发的次数同SQL修改记录的次数一致。&#x20;
* 执行一次语句级的触发器，此时的语句级触发器为后语句级触发器(after statement trigger)。

&#x20;如果多个相同类型的，相同事件触发器作用在同一个对象上，如果在Oracle11g之前，那么最终被执行的会有一定的随机性，而在Oracle 1lg中利用FOLLOWS可以控制其顺序。

> 控制触发器的顺序

```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_ORD_TGR
    BEFORE INSERT
    ON PRODUCTINFO
    FOR EACH ROW
    --在PRODUCTINFO_OPER_TGR 触发器之后被激活
    FOLLOWS PRODUCTINFO_OPER_TGR
BEGIN
    DBMS_OUTPUT.PUT_LINE('触发器顺序测试');
END;
```

#### 复合类型触发器

复合类型的触发器是Oracle11g的新特性，属于触发器的增强部分。复合类型的触发器相当于在一个触发器中包含了4种不同类型的触发器，分别是语句之前(before statement)、行之前(before row)、行之后(after row)、语句之后(after statement)。这么做可以很轻松地把变量在各状态之间传递，而在该类型触发器出现之前，变量在触发器之间的传递比较麻烦。

利用该类型的触发器还可以方便地解决ORA-04091错误，这里涉及一个变异表的概念，该者可以理解变异表是正在被DML操作修改的表，也是触发器的作用表。而触发器通常不能对变异表进行操作，下面一个示例将利用复合类型的触发器，解决ORA-04091错误。

> 复合型触发器

创建普通类型的触发器，并激发，查看是否存在ORA-04091错误。具体脚本如下:

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_ORD_TGR
    AFTER UPDATE ON PRODUCTINFO
    FOR EACH ROW
DECLARE
    v_avg NUMBER(10,2) := 0.0;
BEGIN
    SELECT AVG(PRODUCTPRICE) INTO v_avg FROM PRODUCTINFO
    WHERE PRODUCTPRICE < 2000;
    IF :NEW.PRODUCTPRICE - :OLD.PRODUCTPRICE > v_avg * 0.2 THEN
        RAISE_APPLICATION_ERROR(-20001,'数据修改错误！');
    END IF;
END;
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
UPDATE PRODUCTINFO SET PRODUCTPRICE = PRODUCTPRICE + 300
WHERE PRODUCTPRICE > 2000;
```
{% endcode %}

{% hint style="danger" %}
ORA-04091: 表 DEMO.PRODUCTINFO 发生了变化, 触发器/函数不能读它
{% endhint %}

创建复合类型的触发器，解决上一步出现的错误。具体脚本如下：

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_ORD_TGR
FOR UPDATE ON PRODUCTINFO COMPOUND TRIGGER
    v_avg NUMBER(10,2) := 0.0;
    BEFORE STATEMENT IS
    BEGIN
        SELECT AVG(PRODUCTPRICE) INTO v_avg FROM PRODUCTINFO
        WHERE PRODUCTPRICE < 2000;
    END BEFORE STATEMENT;
    AFTER EACH ROW IS
    BEGIN
        IF :NEW.PRODUCTPRICE - :OLD.PRODUCTPRICE > v_avg * 0.2 THEN
        RAISE_APPLICATION_ERROR(-20011,'数据修改错误！');
        END IF;
    END AFTER EACH ROW;
END PRODUCTINFO_OPER_ORD_TGR;
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
UPDATE PRODUCTINFO SET PRODUCTPRICE = PRODUCTPRICE + 300
WHERE PRODUCTPRICE > 2000;
```
{% endcode %}

{% hint style="info" %}
该触发器已经正常激发，避免了普通触发器出现的ORA-0409以后读者就可以不用费神地解决此类错误了。
{% endhint %}

{% code title="完整案例" lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_ORD_TGR
FOR UPDATE ON PRODUCTINFO COMPOUND TRIGGER
  BEFORE STATEMENT IS  -- 语句执行前触发（表级）
  BEGIN
    DBMS_OUTPUT.put_line('1、BEFORE STATEMENT .') ;
  END BEFORE STATEMENT;
  BEFORE EACH ROW IS  -- 语句执行前触发（行级）
  BEGIN
    DBMS_OUTPUT.put_line('2、BEFORE EACH ROW .') ;
  END BEFORE EACH ROW;
  AFTER EACH ROW IS  -- 语句执行后触发（行级）
  BEGIN
    DBMS_OUTPUT.put_line('3、AFTER EACH ROW .') ;
  END AFTER EACH ROW;
  AFTER STATEMENT IS  -- 语句执行后触发（表级）
  BEGIN
    DBMS_OUTPUT.put_line('4、AFTER STATEMENT .') ;
  END AFTER STATEMENT;
END PRODUCTINFO_OPER_ORD_TGR;
```
{% endcode %}

#### INSTEAD OF类型触发器

在该类型的触发器作用下，如果对作用对象执行DML操作，那么该操作会被触发器的内部操作所取代。触发器作用在视图当中，用于解决视图不可更新的问题。

> 创建视图

{% code lineNumbers="true" %}
```sql
CREATE VIEW PRODUCTINFO_VIEW AS 
SELECT DISTINCT * FROM PRODUCTINFO
```
{% endcode %}

> 创建触发器

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER INSERT_OF_TGR
    INSTEAD OF INSERT ON PRODUCTINFO_VIEW
BEGIN
    INSERT INTO PRODUCTINFO
    VALUES(:NEW.PRODUCTID,:NEW.PRODUCTNAME,:NEW.PRODUCTPRICE,:NEW.DESPERATION,:NEW.ORIGIN);
END;
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
INSERT INTO PRODUCTINFO_VIEW VALUES('5','2',2001,'5','6')

--查询插入结果
SELECT * FROM PRODUCTINFO_VIEW
```
{% endcode %}

#### DDL类型触发器

所谓DDL类型触发器，就是因DDL操作而激发的触发器，主要包括CREATE、ALTER、DROP等事件。

利用DDL类型的触发器可以限制和记录特定的DDL操作。例如，通过DDL触发器可以限制对数据库结构的修改，记录数据库中的更改事件，也可以在修改对象的时候根据实际情况做出其他相应动作。下面通过一个示例演示如何创建和使用该类型的触发器。

> 创建触发器

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER DDL_TGR
    BEFORE CREATE OR ALTER OR DROP OR RENAME ON SCHEMA
BEGIN
    IF SYSEVENT = 'CREATE' THEN 
        DBMS_OUTPUT.PUT_LINE(DICTIONARY_OBJ_NAME||'创建中……');
    ELSIF SYSEVENT = 'ALTER' THEN 
        RAISE_APPLICATION_ERROR(-20000,'不允许修改表！');
    ELSIF SYSEVENT = 'DROP' THEN 
        RAISE_APPLICATION_ERROR(-20000,'不允许删除表！');
    ELSIF SYSEVENT = 'RENAME' THEN 
        RAISE_APPLICATION_ERROR(-20000,'不允许修改表名称！');
    END IF;
END;
```
{% endcode %}

> 测试触发器

{% code lineNumbers="true" %}
```sql
--TEST创建中……
CREATE TABLE TEST(
    ID VARCHAR2(32) NOT NULL,
    NAME VARCHAR2(30) NULL,
    PRIMARY KEY(ID)
);

--ORA-20000: 不允许修改表名称！
RENAME TEST TO TEST2; 

--ORA-20000: 不允许修改表！
ALTER TABLE TEST ADD DESC VARCHAR2(200);

--ORA-20000: 不允许删除表！
DROP TABLE TEST;
```
{% endcode %}

> 常用事件属性

<table><thead><tr><th width="241">属性函数</th><th width="164">可用的事件</th><th>简介</th></tr></thead><tbody><tr><td>SYSEVENT</td><td>所有事件</td><td>返回激发触发器的事件名称</td></tr><tr><td>INSTANCE_NUM</td><td>所有事件</td><td>返回当前数据库的示例号</td></tr><tr><td>DATABASE_NAME</td><td>所有事件</td><td>返回当前数据库名</td></tr><tr><td>SERVER_ERROR</td><td>SERVERERROR</td><td>错误堆栈的指定位置返回错误号</td></tr><tr><td>LOGIN_USER</td><td>所有事件</td><td>返回激发触发器的用户名</td></tr><tr><td>DICTIONARY_OBJ_TYPE</td><td>CREATE、ALTER、DROP</td><td>返回激活触发器的DDL操作的对象类型</td></tr><tr><td>DICTIONARY_OBJ_NAME</td><td>CREATE、ALTER、DROP</td><td>返回激活触发器的DDL操作的对象名称</td></tr></tbody></table>

#### 用户和系统事件触发器

所谓系统事件触发器，就是基于Oracle系统事件而建立的触发器。该类型的触发器可以审计数据库的登录、注销以及关闭和启动等。

> 创建数据库级触发器

该示例将记录每个登录用户的时间，并把登录时间存放到用户登录记录表中。

{% code lineNumbers="true" %}
```sql
CREATE TABLE LOG_USER(
    LOGIN_ID VARCHAR2(50),
    LOGIN_NAME VARCHAR2(50),
    LOGIN_TIME TIMESTAMP,
    CONSTRAINT LOG_USER_PRK PRIMARY KEY(LOGIN_ID)
);
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER LOGIN_TGR
    AFTER LOGON ON DATABASE
BEGIN
    INSERT INTO LOGIN 
    VALUES(LOG_TAB_ID.NEXTVAL,SYS.LOGIN_USER,SYSDATE);
END;
```
{% endcode %}

> 测试触发器

{% code lineNumbers="true" %}
```sql
--重新登录后查看日志
SELECT * FROM LOG_USER;
```
{% endcode %}

#### 设置触发器是否可用

<pre class="language-sql" data-line-numbers><code class="lang-sql"><strong>ALTER TRIGGER [schema.]trigger DISABLE|ENABLE;
</strong><strong>
</strong><strong>--禁用 LOGIN_TGR 触发器
</strong><strong>ALTER TRIGGER LOGIN_TGR DISABLE;
</strong></code></pre>

### 修改、删除触发器

#### 修改触发器

触发器内容的修改同样要利用REPLACE关键词。创建触发器时带上OR REPLACE关键词，从而完成过程的修改，也就是覆盖。

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER LOGIN_TGR
...
```
{% endcode %}

#### 删除触发器

{% code lineNumbers="true" %}
```sql
DROP TRIGGER [schema.]trigger

--删除 LOGIN_TGR 触发器
DROP TRIGGER LOGIN_TGR;
```
{% endcode %}

### 小结

触发器是数据库的重要组成部分，利用触发器可以完成更复杂的业务逻辑，也可以对数据库的操作行为进行记录。DML类型触发器、DDL类型触发器、复合类型触发器、替代类型触发器、系统事件触发器，可以根据实际情况选择一种或几种使用，来实现更复杂的功能。

## 事务和锁-确保数据安全

事务和锁是两个联系非常紧密的概念，它们保证了数据库的一致性。由于数据库是一个可以由多个用户共享的资源，因此当多个用户并发地存取数据时，就要保证数据的准确性。事务和锁就完成了这项功能。

### 什么是事务

事务在数据库中主要用于保证数据的一致性，防止出现错误数据。在事务内的语句都会被看成一个单元，一旦有一个失败，那么所有的都会失败。

#### 认识事务

事务就是一组包含一条或多条语句的逻辑单元，每个事务都是一个原子单位，在事务中的语句被作为一个整体，要么一起被提交，作用在数据库上，使数据库数据永久的修改；要么一起被撤销，对数据库不做任何的修改。

对于这个问题比较经典的例子就是银行账户之间的汇款转账操作。该操作在数据库中由以下3步完成:

* 源账户减少存储金额，例如减少1000。
* 目标账户增加存储金额，增加1000。&#x20;
* 在事务日志中记录该事务。

整个交易过程，我们看做一个事务，如果操作失败，那么该事务就会回滚，所有该事务中的操作将撤销，目标账户和源账户上的资金都不会出现变化；如果操作成功，那么将是对数据库永久的修改，即使以后服务器断电，也不会对该修改结果有影响。&#x20;

事务在没有提交之前可以回滚，而且在提交前当前用户可以查看已经修改的数据，但其他用户查看不到该数据，一旦事务提交就不能再撤销修改了。Oracle的事务基本控制语句有如下几个:

* SETTRANSACTION:设置事务的属性。&#x20;
* COMMIT:提交事务。&#x20;
* SAVEPOINT:设置保存点。&#x20;
* ROLLBACK:回滚事务。&#x20;
* ROLLBACK TO SAVEPOINT:回滚至保存点。

{% hint style="info" %}
事务和程序不同，一条语句或者多条语句甚至一段程序都可能在一个事务中，而一段程序又可以包含多个事务。事务可以根据自己的需要把一段程序分成多个组，然后把每个组都当成一个单元，而这个单元就可以理解为一个事务。
{% endhint %}

#### 事务的类型

事务分为如下两种类型：显式方式，隐式方式。

> 显示方式

所谓的“显式方式”，就是利用命令完成。

{% code lineNumbers="true" %}
```sql
sql statement
...
COMMIT|ROLLBACK;
```
{% endcode %}

Oracle中的事务不需要设置开始标志。通常有下列情况之一时，事务会开启:

* 登录数据库后，第一次执行DML语句。&#x20;
* 当事务结束后，第一次执行DML语句。

> 隐式方式

该类型的事务没有明确的开始和结束标志。它由数据库自动开启，当一个程序正常结束或使用DDL语言时会自动提交，而操作失败时也会自动回滚。如果设置AUTOCOMMIT为打开状态(默认关闭)，则每次执行DML操作都会自动提交。命令语法如下:

```sql
SET AUTOCOMMIT ON/OFF
```

事务在什么条件下结束读者需要注意，否则有丢失数据的可能。当有下列情况之一时，事务会结束: 口

* 使用COMMIT事务提交，ROLLBACK事务回滚。
* 执行DDL语句，事务自动提交。例如，使用CREATE、DROP、GRANT、REVOKE等 命令。
* 正常退出时自动提交事务，非正常退出则ROLLBACK事务回滚。

事务可以保证数据的一致性。前面已经介绍过，事务没有提交时，当前会话所做的操作其他会话不会看到。

#### 事务的保存点

在事务中可以根据自己的需要设置保存点。保存点可以设置在事务中的任何地方，也可以设置多个点，这样就可以把比较长的事务根据需要分成较小的段，这样做的好处是当对数据的操作出现问题时可以不用全部回滚，只需要回滚到保存点处即可。例如，在一个事务内前十段数据操作确认准确，而后面的操作没有办法确认，开发人员就可以在第十段操作结束后设置保存点，这样即便后面的操作有错误，开发人员也可以利用保存点回滚到第十段处，而不用从头写。

一旦把事务回滚到某个保存点后，Oracle将把保存点之后持有的锁释放掉，这时先前等待被锁资源的事务就可以继续了。而使事务回滚到保存点，有下列几点需要了解:

* 事务只回滚保存点之后的操作。&#x20;
* 回滚到某保存点时，它以后的保存点将被删除，但保存点会被保留。&#x20;
* 保存点之后的锁将被释放，但之前的会被保留。&#x20;

保存点使用起来非常方便，只需一行脚本就能完成。

<pre class="language-sql" data-line-numbers><code class="lang-sql"><strong>INSERT INTO DEMO.PRODUCTINFO P 
</strong>VALUES('7','2',3,'5','6');

--设置保存点
SAVEPOINT FST;

SELECT * FROM DEMO.PRODUCTINFO;

INSERT INTO DEMO.PRODUCTINFO P 
VALUES('8','2',3,'5','6');

--回退到设置的保存点
ROLLBACK TO FST;

SELECT * FROM DEMO.PRODUCTINFO;

COMMIT;
</code></pre>

#### 事务的ACID特性

事务有4个特性，它们分别是原子性、一致性、分离性、持入性。&#x20;

* 原子性:事务的原子性是指，事务中程序是数据库的逻辑工作单位，它对数据的修改要么全部执行，要么完全不执行。原子也意味着不可分割，不管有多少程序，只要在同一个事务中，那么它们就是一个整体，如果都执行成功才意味着该事务成功，而有一个操作失败，那 么同一个事务中的其他操作即使执行成功也没有用，事务会使其全部撤销。
* 一致性:事务的一致性指事务执行的前后数据库都必须处于一致性状态，它是相对脏读而言的。只有在事务完成后才能被所有使用者看见，保证了数据的完整性。例如在银行转账时，从A账户取款但没有放到B账户中时数据是不一致的，同时也是不完整的，其他使用者此时不能看到A中修改后的数据，只有存到B账户中，交易完成并提交事务，这时才算数据一致，所有用户也会看到修改后的数据。&#x20;
* 分离性(隔离性):分离性是指并发事务之间不能相互的干扰。也就是说，一个事务操作的数据不会被其他事务看到和操作。&#x20;
* 持久性:持久性是指一旦事务提交完成，那么这将是对数据永久的修改，即使被修改后的数据遭到破坏，也不会出现回到修改之前的情况。

{% hint style="info" %}
事务的提交很重要，但不建议频繁地提交事务，因为每次提交事务都需要时间，如果10000行记录，每行记录都提交事务，那么事务本身将是性能的主要消耗者。所以，适当地减少事务提交次数比较重要。例如，可以每1000行提交一次。读者可以根据自己的实际情况自行决定。
{% endhint %}

### 什么是锁

数据库是一个庞大的多用户数据管理系统，由于在多用户的系统中，同一时刻多个用户同时操作某相同资源的情况时有发生，而在逻辑上这些用户想同时操作该资源是不可能的，而数据库中利用锁消除了多用户操作同一资源时可能产生的隐患。

#### 认识锁

锁出现在数据共享的环境中，它是一种机制，在访问相同资源时，可以防止事务之间的破坏性交互。例如，在多个会话同时操作某表时，优先操作的会话需要对其锁定。

事务的分离性要求当前事务不能影响其他的事务，所以当多个会话访问相同的资源时，数据库系统会利用锁确保它们像队列一样依次进行。Oracle处理数据时用到的锁是自动获取的，我们不用对此有过多的关注，但Oracle允许我们手动锁定数据。&#x20;

Oracle利用很低的约束提供了最大程度的并发性，例如某会话正在修改一条记录，那么仅仅该记录会被锁定。而其他会话可以随时做读取操作，但读取的依然是修改前的数据。

Oracle的锁保证了数据的完整性。例如，当一个会话对表A的某行记录进行修改时，另一个会话也来修改该行记录，在没有任何处理的情况下保留的数据会有随机性，而这种数据是没有任何意义的，为脏数据。如果此时使用了行级锁，第一个会话修改记录时封锁该行，那么第二个会话此时只能等待，这样就避免了脏数据的产生。

#### 锁的分类

Oracle中分为两种模式的锁，一种是排他锁(X锁)，另一种是共享锁(S锁)。

* 排他锁也可以叫写锁。这种模式的锁防止资源的共享，用做数据的修改。假如有事务T给数据A加上该锁，那么其他的事务将不能对A加任何的锁，所以此时只允许T对该数据进行读取和修改，直到事务完成将该类型的锁释放为止。
* 共享锁也可以叫读锁。该模式锁下的数据只能被读取，不能被修改。如果有事务T给数据A加上共享锁后，那么其他事务不能对其加排他锁，只能加共享锁。加了该锁的数据可以被并发地读取。 锁是实现并发的主要手段，在数据库中应用频繁，但很多都由数据库自动管理，当事务提交后会自动释放锁。

锁是实现并发的主要手段，在数据库中应用频繁，但很多都由数据库自动管理，当事务提交后会自动释放锁。

#### 锁的类型

Oracle为了使数据库实现高度的并发访问，它使用了不同类型的锁来管理并发会话对数据对象的操作。Oracle的锁按作用对象不同分为如下几种类型。&#x20;

* DML锁:该类型的锁被称为数据锁，用于保护数据&#x20;
* DDL锁:可以保护模式中对象的结构。
* 内部锁:保护数据库的内部结构，完全自动调用。

其中，DML锁主要保证了并发访问时数据的完整性。如果再细分，它又可以分为如下两种类型的锁:

* 行级锁(TX)，也可以称为事务锁。当修改表中某行记录时，需要对将要修改的记录加行级锁，防止两个事务同时修改相同记录，事务结束，该锁也会释放，是粒度最细的锁。该锁只能属于排他锁(X锁)。
* 表级锁(TM)，主要作用是防止在修改表的数据时，表的结构发生变化。例如，S在修改表A的数据时它会得到表A的TM锁，而此时将不允许其他会话对该表进行变更或操作。

> 验证过程

{% code lineNumbers="true" %}
```sql
--会话A,修改数据不提交
UPDATE DEMO.PRODUCTINFO SET PRODUCTNAME ='2' 
WHERE PRODUCTID = '8';

--会话B,删除表
DROP TABLE DEMO.PRODUCTINFO;

--提示：ORA-00054: 资源正忙, 但指定以 NOWAIT 方式获取资源, 或者超时失效
```
{% endcode %}

在执行DML操作的时候，数据库会先申请数据对象上的共享锁，防止其他的会话对该对象执行DDL操作。一旦申请成功，则会对将要修改的记录申请排他锁，如果此时其他会话正在修改该记录，那么等待其事务结束后再为修改的记录加上排他锁。

表级锁包含如下几种模式:&#x20;

* ROWSHARE，行级共享锁(RS)。该模式下不允许其他的并行会话对同一张表使用排他锁，但允许其利用DML语句或lock命令锁定同一张表中的其他记录。SELECTFROM FOR UPDATE语句就是给记录加上了RS锁。
* SHARE，共享锁(S)。该模式下，不允许会话更新表，但允许对表添加RS锁。
* ROWEXCLUSIVE，行级排他锁(RX)。该模式下允许并行会话对同一张表的其他数据进行修改，但不允许并行会话对同一张表使用排他锁。
* SHARE ROW EXCLUSIVE，共享行级排他锁(SRX)。该模式下，不能对同一张表进行DML操作，也不能添加S锁。&#x20;
* EXCLUSIVE，排他锁(X)。该模式下，其他的并行会话不能对表进行DML和DDL操作，该表只能读。

> 表级锁模式的相互兼容性

<table><thead><tr><th width="100"></th><th>RS</th><th>S</th><th>RX</th><th>SRX</th><th>X</th></tr></thead><tbody><tr><td>RS</td><td>✔</td><td>✔</td><td>✔</td><td>✔</td><td>❌</td></tr><tr><td>S</td><td>✔</td><td>✔</td><td>❌</td><td>❌</td><td>❌</td></tr><tr><td>RX</td><td>✔</td><td>❌</td><td>✔</td><td>✔</td><td>❌</td></tr><tr><td>SRX</td><td>✔</td><td>❌</td><td>❌</td><td>❌</td><td>❌</td></tr><tr><td>X</td><td>❌</td><td>❌</td><td>❌</td><td>❌</td><td>❌</td></tr></tbody></table>

> SQL语句所产生的表级锁情况

<table><thead><tr><th width="498">SQL语句</th><th>表锁模式</th><th>RS</th><th>S</th><th>RX</th><th>SRX</th><th>X</th></tr></thead><tbody><tr><td><code>SELECT...FROM table ...</code></td><td>NONE</td><td>Y</td><td>Y</td><td>Y</td><td>Y</td><td>Y</td></tr><tr><td><code>INSERT INTO table ...</code></td><td>RX</td><td>Y</td><td>N</td><td>Y</td><td>N</td><td>N</td></tr><tr><td><code>UPDATE table ...</code></td><td>RX</td><td>Y</td><td>N</td><td>Y</td><td>N</td><td>N</td></tr><tr><td><code>DELETE FROM table ...</code></td><td>RX</td><td>Y</td><td>N</td><td>Y</td><td>N</td><td>N</td></tr><tr><td><code>SELECT * FROM table FOR UPDATE</code></td><td>RX</td><td>Y</td><td>N</td><td>Y</td><td>N</td><td>N</td></tr><tr><td><code>LOCK TABLE table IN ROW SHADE MODE</code></td><td>RS</td><td>Y</td><td>Y</td><td>Y</td><td>Y</td><td>N</td></tr><tr><td><code>LOCK TABLE table IN ROW EXCLUSIVE MODE</code></td><td>RX</td><td>Y</td><td>N</td><td>Y</td><td>N</td><td>N</td></tr><tr><td><code>LOCK TABLE table IN SHADE MODE</code></td><td>S</td><td>Y</td><td>N</td><td>N</td><td>N</td><td>N</td></tr><tr><td><code>LOCK TABLE table IN SHADE ROW EXCLUSIVE MODE</code></td><td>SRX</td><td>Y</td><td>N</td><td>N</td><td>N</td><td>N</td></tr><tr><td><code>LOCK TABLE table IN EXCLUSIVE MODE</code></td><td>X</td><td>N</td><td>N</td><td>N</td><td>N</td><td>N</td></tr></tbody></table>

{% code lineNumbers="true" %}
```sql
LOCK TABLE [schema.]table IN
    [EXCLUSIVE]
    [SHARE]
    [ROW EXCLUSIVE]
    [SHARE ROW EXCLUSIVE]
    [ROW SHARE* | SHARE UPDATE*]
    MODE [NOWAIT]
```
{% endcode %}

如果要释放它们，只需使用ROLLBACK命令。&#x20;

DDL锁也可以称为数据字典锁，主要作用是保护模式中对象的结构。当执行DDL操作时首先Oracle会自动地隐式提交一次事务，然后自动地给处理对象加上锁；当DDL结束时Oracle会隐式地提交事务并释放DDL锁。与DML不同的是，用户不能显式地要求使用DDL锁。

DDL锁分为如下3类:&#x20;

* Exclusive DDL Lock，排他DDL锁定。如果对象加上了该类型的锁，那么对象不能被其他会话修改，而且该对象也不能再增加其他类型的DDL锁。如果是表，此时可以读取数据。
* Shared DDLLock，共享DDL锁定。保护对象的结构，其他会话不能修改该对象的结构，但是允许修改数据。
* Breakable Parsed Lock，能打破的解析锁定。该类型的锁可以被打断，不能禁止DDL操作。

#### 锁等待与死锁

在某些情况下由于占用的资源不能及时释放，而造成锁等待，也可叫锁冲突。锁等待会严重地影响数据库性能和日常工作。例如，当一个会话修改表A的记录时，它会对该记录加锁，而此时如果另一个会话也来修改此记录，那么第二个会话将因得不到排他锁而一直等待，此时会出现执行SQL时数据库长时间没有响应的现象。直到第一个会话把事务提交，释放锁，第二个会话才能对数据进行操作。

> 锁等待

{% code lineNumbers="true" %}
```sql
--会话A,修改后不提交
UPDATE DEMO.PRODUCTINFO P SET P.PRODUCTNAME ='1' 
WHERE P.PRODUCTID  = '6';

--会话B
UPDATE DEMO.PRODUCTINFO P SET P.PRODUCTNAME ='2' 
WHERE P.PRODUCTID  = '6';

会话B出现锁等待现象
```
{% endcode %}

此时的情况是因为第一个会话封锁了该记录，但事务没有结束，锁不会释放，而这时第一个会话也要修改同一条记录，但它却没有办法获得锁，所以只能等待。如果第一个会话修改数据的事务结束，那么第二个会话就会结束等待。及时地结束事务是解决锁等待情况发生的有效方法。&#x20;

死锁的发生和锁等待不同，它是锁等待的一个特例，通常发生在两个或多个会话之间。假设一个会话想要修改两个资源对象，可以是表也可以是字段，修改这两个资源的操作在一个事务当中。当它修改第一个对象时需要对其锁定，然后等待第二个对象，这时如果另外一个会话也需要修改这两个资源对象，并且已经获得并锁定了第二个对象，那么就会出现死锁，因为当前会话锁定了第一个对象等待第二个对象，而另一个会话锁定了第二个对象等待第一个对象。这样，两个会话都不能得到想要得到的对象，于是出现死锁。

> 死锁的发生

{% code lineNumbers="true" %}
```sql
--会话A,修改后不提交
UPDATE DEMO.PRODUCTINFO P SET P.PRODUCTNAME ='1' 
WHERE P.PRODUCTID  = '6';

--会话B
UPDATE DEMO.PRODUCTINFO P SET P.PRODUCTNAME ='2' 
WHERE P.PRODUCTID  = '7';

会话A锁定了PRODUCTID字段为6的记录，
会话B锁定了PRODUCTID字段为7的记录。

--在会话A中修改会话B的数据
UPDATE DEMO.PRODUCTINFO P SET P.PRODUCTNAME ='2' 
WHERE P.PRODUCTID  = '7';

--在会话B中修改会话A的数据
UPDATE DEMO.PRODUCTINFO P SET P.PRODUCTNAME ='1' 
WHERE P.PRODUCTID  = '6';

--最后执行的会话提示:ORA-00060:等待资源时检测到死锁
```
{% endcode %}

此时Oracle自动做出处理，并重新回到锁等待的情况。出现锁等待的情况时应尽快地找出错误原因并对其处理，避免影响数据库性能。实际开发中出现此类情况大致有以下几种原因:

* 用户没有良好的编程习惯，偶尔会忘记提交事务，导致长时间占用资源。
* 操作的记录过多，而且操作过程中没有很好地对其分组。前面介绍过，对于数据量很大的操作，可以将其分成几组提交事务，这样可以避免长时间地占用资源。
* 逻辑错误，两个会话都想得到已占有的资源。

### 小结

事务很重要，它保证了数据的一致性，而锁和事务两者联系紧密。









