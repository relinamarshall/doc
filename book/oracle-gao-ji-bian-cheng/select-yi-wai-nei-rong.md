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

# SELECT以外内容

本章是关于SQL语句中除SELECT以外话题的集合。这些语句通常被称为数据操作语言`(Data Manipulation Language,DML)`语句。我想要提供一些关于标准`DML`命令(具体来说包括`INSERT、UPDATE、DELETE`以及`MERGE`)的一些不那么广为人知的选项信息。本章还将集中研究改善性能的其他可选途径。

## INSERT

Oracle有两种基本方法来实现数据的插入。简单来说，将其称为**慢速方法**和**快速方法**。慢速方法通常也称为常规方法。在这种机制下，数据通过缓冲区缓存，已有块中的空白空间得到重用，默认为所有数据和元数据的变更创建撒销，并为所有变更创建重做。这个工作量是很大的，也就是为什么我将其称为慢速方法。快速方法也称为**直接路径法**。这种方法不去查找已有块中的空间，它直接从高水位线之上开始插入数据。它对元数据的变更通过撤销和重做来保护数据字典，但并不为数据的变更生成撒销。在某些情况下(也就是**nologging**运算中)它也可以避免为数据的变更生成重做。记住默认情况下通过直接路径插入进行加载的表上的索引仍然是会生成撤销和重做的。

### 直接路径插入

直接路径插入可以使用`APPEND`提示来调用(顺便说一下,这是**并行插入**的默认行为)。在Oracle数据库`11gR2`版本中，有一个新的`APPEND_VALUES`提示可以用来进行直接指定要插入的值而不是通过`SELECT`语句提供值的插入。下列代码清单分别给出了这两个提示的简单例子。

> 简单插入APPEND和APPEND\_VALUES

{% code lineNumbers="true" %}
```sql
insert /*+ append */ into kso.big_emp 
select * from hr.employees nologging;

insert /*+ append_values */ into dual (dummy) values ('Y');
```
{% endcode %}

然而，快速方法有下面这几个问题。

* 在任何给定的时点一张表上只能有一个直接路径写入。
* 数据将被插入到高水位线之上，因此任何高水位线之下的可用空间都不能在直接路径插入中使用。
* 在开始之后进行插入的会话不能对表做任何事情(甚至是对其进行`select`)，直到进行提交或回滚。
* 不支持一些不太常用的数据结构(对象类型、索引组织表等)。
* 不支持引用约束(也就是说它们将导致通过传统方法进行插入)。

上面列表中的第1条是最大的问题。在一个需要频繁进行小数据量插入的`OLTP`型系统中，直接路径机制就不可行。列出来的第2个问题也是个大问题。对于小数据量插入，直接插入到高水位线之上的空白块中毫无意义，这样只会导致空间的巨大浪费。事实上，在`Oracle`数据库`11g`版本中，将`APPEND`提示的行为修改为允许其在使用`Values`子句的`INSERT`语句中应用(在`11g`之前，除非`INSERT`语句有`SELECT`子句，否则这个提示将被忽略)。这一行为上的变化导致系统记录了一个缺陷(`bug`)，因为小数据量的插入使用了太多的空间。`Oracle`数据库`11gR2`中最终的解决方案是将`APPEND`提示还原为其原来的行为，并引入一个新的提示`APPEND_VALUES`。不管怎么样，你都要注意直接路径插入是仅为大数据量插入而设计的。

还要注意的是，`APPEND`与大多数提示一样，如果由于任何原因使得`Oracle`不能遵守，则这个提示将会被默默地忽略。如果`APPEND`提示被忽略了,插入就会使用常规机制来进行。

### 多表插入

尽管至少在`9i`版本中就出现了，多表插入很少被用到。对于首先对数据进行分级，然后在加载到固定结构中时重新分配的`ETL(Extract, Transform, Load 抽取、转换、加载)`类型处理这一结构很有用。在这些情况下，通常会将数据分级为非标准格式，然后分割到多张表或其他更标准化的结构中。多表插入是完成此类工作却不需要写大量的处理过程代码的非常方便的方法。语法也是很直接的:只要使用`INSERT ALL`然后提供多个`INTO`子句。

这些子句可以指定同一张或不同的表。只能使用一套用来插入的值(通过`Values`子句或者一个子查询都可以)，但单个的值可以被重复使用或者根本不用。下列代码清单给出了一个插入到单表的语法例子。

> 基本的插入到单表中的多表插入

{% code lineNumbers="true" %}
```sql
INSERT ALL
INTO people(person_id,first_name,last_name)
    VALUES(person_id,first_name,last_name)
INTO people(first_name,last_name,parent_id)
    VALUES(child1,last_name,person_id)
INTO people(first_name,last_name,parent_id)
    VALUES(child2,last_name,person_id)
INTO people(first_name,last_name,parent_id)
    VALUES(child3,last_name,person_id)
INTO people(first_name,last_name,parent_id)
    VALUES(child4,last_name,person_id)
INTO people(first_name,last_name,parent_id)
    VALUES(child5,last_name,person_id)
INTO people(first_name,last_name,parent_id)
    VALUES(child6,last_name,person_id)
SELECT person_id,first_name,last_name,
    child1,child2,child3,child4,child5,child6
FROM denormalized_people;
```
{% endcode %}

上面这个例子显示了可以使用多个`INTO`子句，尽管在这个例子中所有的`INTO`子句指向同一张表。你同样也可以尽可能简单地插入到多张表中(因此术语是多表插入),如下列代码中所示。

> 基本的多表插入

{% code lineNumbers="true" %}
```sql
INSERT ALL
INTO people(person_id,first_name,last_name)
    VALUES(person_id,first_name,last_name)
INTO children(first_name,last_name,parent_id)
    VALUES(child1,last_name,person_id)
INTO children(first_name,last_name,parent_id)
    VALUES(child2,last_name,person_id)
INTO children(first_name,last_name,parent_id)
    VALUES(child3,last_name,person_id)
INTO children(first_name,last_name,parent_id)
    VALUES(child4,last_name,person_id)
INTO children(first_name,last_name,parent_id)
    VALUES(child5,last_name,person_id)
INTO children(first_name,last_name,parent_id)
    VALUES(child6,last_name,person_id)
SELECT person_id,first_name,last_name,
    child1,child2,child3,child4,child5,child6
FROM denormalized_people;
```
{% endcode %}

### 条件插入

`INSERT`命令也具有处理条件的能力，就好像在`INSERT`语句中嵌入了`CASE`语句一样。

在前面的例子中，你为每一个子成员插入了一条记录，但在具有重复列的情况下，很可能其中一些子列将会是空值。因此如果你在不必写处理代码的情况下就可以避免创建这些空记录将是很美好的一件事情。这正是条件插入功能所适用的场景。顺便说一句，这种类型的数据布局在从外部系统中加载文件时经常见到。在文件上创建外部表是加载它们的很好的方法，也使得这些不那么常用的插入选项可以直接应用到数据加载过程中，而不是在数据被分批加载到一张`Oracle`表中之后。下列代码清单给出了一个条件插入的例子，其中父字段总是加载，但子字段只有在其中有数据的时候才进行加载。

> 条件插入

{% code lineNumbers="true" %}
```sql
INSERT ALL
WHEN 1=1 THEN
    INTO people(person_id,first_name,last_name)
        VALUES(person_id,first_name,last_name)
WHEN child1 is not null THEN
    INTO children(first_name,last_name,parent_id)
        VALUES(child1,last_name,person_id)
WHEN child2 is not null THEN
    INTO children(first_name,last_name,parent_id)
        VALUES(child2,last_name,person_id)
WHEN child3 is not null THEN
    INTO children(first_name,last_name,parent_id)
        VALUES(child3,last_name,person_id)
WHEN child4 is not null THEN
    INTO children(first_name,last_name,parent_id)
        VALUES(child4,last_name,person_id)
WHEN child5 is not null THEN
    INTO children(first_name,last_name,parent_id)
        VALUES(child5,last_name,person_id)
WHEN child6 is not null THEN
    INTO children(first_name,last_name,parent_id)
        VALUES(child6,last_name,person_id)
SELECT person_id,first_name,last_name,
    child1,child2,child3,child4,child5,child6
FROM denormalized_people;
```
{% endcode %}

### DML错误日志

现在要讨论的是一个非常酷的功能:DML错误日志。这个功能提供了一种机制来使得你的一百万行的数据插入不会仅仅由于几行有问题而失败。这个特性是在`10gR2`版本中引人的，类似于`SQL*Loader`的错误日志功能。它的基本原理是将任何可能导致语句失败的记录转移,放入到一张错误记录表中。这是一个非常有用但却很少用到的功能,这确实有点奇怪,因为它很容易实现。其性能也很好并且可以节约大量的代码。如果没有这个功能你将不得不创建一个坏记录表，编写过程代码来处理任何一条记录所引起的异常，将出现问题的记录放入到坏记录表中，还要在一个自治事务中来处理错误记录以保持事务完整性。这个工作量很大。顺便说一下，`LOG ERRORS`子句在其他`DML`语句(`UPDATE`、`DELETE`以及`MERGE`)中同样适用。&#x20;

* (1)使用`DBMS_ERRLOG.CREATE_ERROR_LOG`来创建错误日志表。
* (2)在`INSERT`语句中声明`LOG_ERRORS`子句。&#x20;

这样就可以了。下列代码清单示出了`CREATE_ERROR_LOG`存储过程是如何工作的。

> CREATE\_ERROR\_LOG

```sql
EXECUTE DBMS_ERRLOG.CREATE_ERROR_LOG('big_emp','big_emp_bad');
```

<figure><img src="../.gitbook/assets/image (77).png" alt=""><figcaption><p>CREATE_ERROR_LOG</p></figcaption></figure>

`Errors`表中的所有列都创建为`VARCHAR2(4000)`。这使得绝大多数的数据类型列可以插入到`Errors`表中,即使是由于记录太长而不能正常插入导致的错误或者因为数据类型不一致，如数值列中包含非数值数据所导致的问题也可以放进来。除此以外还有关于错误号、错误信息和行编号的几个附加列。最后，有一个被称为`ORA_ERR_TAGS`的列，允许放入用户自定义的数据以便进行调试(即`ETL`过程所处的步骤，或其他性质类似的内容)。

语法是非常直接的。你只需要加入关键字`LOGE RRORS INTO`并指定你的错误表名称就可以了或者，你也可以告诉`Oracle`在放弃插入并取消语句之前允许多少个错误出现。这是通过`Reject Limit`子句来实现的。你要注意的是`Reject Limit`的默认值设置为`0`，因此如果有一个错误，语句就会取消并进行回滚(只是这个语句，而不是事务)。不过这个单独的错误也将会放入到Errors`表`中。在大多数情况下，你可能需要将`Reject Limit`设置为`UNLIMITED`，从而允许插入语句完成而不管有多少条记录将会转移到`Errors`表中去。有点令人惊奇的是最常用的`UNLIMITED`并不是默认值。下列代码清单给出了一个简单的例子。

> 插入错误日志

{% code lineNumbers="true" %}
```sql
insert into big_emp(employee_id,first_name,
    last_name,hire_date,email,department_id)
values(300,'Bob','Loblaw','01-jan-10',
    'bob@yourfavoritelawyer.com',12345)
log errors info big_emp_bad;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```sql
insert into big_emp(employee_id,first_name,
    last_name,hire_date,email,department_id)
values(300,'Bob','Loblaw','01-jan-10','bob@yflawyer.com',12345)
log errors info big_emp_bad;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```sql
insert into big_emp(employee_id,first_name,
    last_name,hire_date,email,department_id,job_id)
values(300,'Bob','Loblaw','01-jan-10','bob@yflawyer.com',12345,1)
log errors info big_emp_bad;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```sql
insert into big_emp(employee_id,first_name,
    last_name,hire_date,email,department_id,job_id)
values(300,'Bob','Loblaw','01-jan-10','bob@yflawyer.com','2A45',1)
log errors info big_emp_bad;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

```sql
select ora_err_mesg$,ora_err_tag$,employee_id from big_emp_bad;
```

上面代码清单中的例子给出了多个执行失败的插入语句。插入操作中失败的记录，不管是什么原因导致的失败，都自动地插入到了`Errors`表中。由于我没有指定`Reject Limit`的值，每一条语句都是在遇到第1个错误的时候就进行了回滚。因此，实际上没有任何记录被插入到`BIG_EMP`表中，但所有的错误记录都被保留了下来。这样做的目的是要说明一个`Errors`表可以被多个加载过程重用,保留多个插入语句的错误记录。注意在实际当中错误日志很少被这样来用。在实际中通常都会将`Reject Limit`设置为`UNLIMITED`。下面代码清单给出了使用多行插人语句的更好的例子。

> 更多的插入错误日志

{% code lineNumbers="true" %}
```sql
create table test_big_insert as select * from dba_objects where 1=2;

desc test_big_insert;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (74).png" alt=""><figcaption><p>更多的插入错误日志</p></figcaption></figure>

{% code lineNumbers="true" %}
```sql
alter table test_big_insert modify object_id number(2);

insert into test_big_insert
select * from dba_objects
where object_id is not null;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (75).png" alt=""><figcaption><p>错误提示</p></figcaption></figure>

由于是特意设计的这种情况，我很清楚是哪一列引起的问题。是在代码清单中进行了修改的`object_id`这一列。但在实际当中，通常有问题的列不会这么显而易见。事实上，如果没有`Error Logging`子句，要确定是哪一行引起的问题是非常困难的。

错误消息并没有给我任何关于哪一列或哪条记录导致了失败方面的信息。我可以通过在`SELECT`语句中手工指定每一列的列名来确定是哪一列引起了问题。但根本不可能知道是哪一行引起的问题。下列代码清单中的`Error Logging`子句解决了这两个问题。记住所有列值和错误信息一起被存进了`Errors`表中，使得确定问题所在变得容易了。

{% code lineNumbers="true" %}
```sql
EXECUTE DBMS_ERRLOG.CREATE_ERROR_LOG('test_big_insert','tbi_errors');

insert into test_big_insert
select * from dba_objects
where object_id is not null
log errors into tbi_errors
reject limit unlimited;

--98
select count(*) from dba_objects
where object_id is not null and length(object_id) < 3;

--98
select count(*) from dba_objects
where object_id is not null
and length(object_id) < 3;

--72529
select count(*) from dba_objects
where object_id is not null
and length(object_id) > 2;

--72529
select count(*) from tbi_errors;
```
{% endcode %}

这个例子示出了`Reject Limit`为`UNLIMITED`的`Error Logging`子句，使得即使大多数的记录插入失败了，插入语句也可以完成。此外，你可以看到尽管回滚将记录从基表中移除掉了，但错误记录保留了下来。

尽管`DML`的错误记录非常强大，你需要注意下面这些警告。

* `LOG ERRORS`子句不引起隐式提交。错误记录的插入是由一个自治事务来处理的，也就是说即使返回了错误并将坏记录插入到了`Errors`表中，你也可以提交或回滚插入到基表中的所有记录(以及其他相关的修改)。即使事务进行了回滚，载入到Errors表中的记录也将会保留。
* `LOG ERRORS`子句并不会禁用`APPEND`提示。如果使用了`APPEND`提示，对于基表的插入将会使用直接路径写入机制来完成。但是，任何到`Errors`表的写入都不会使用直接路径写入。这通常并不是问题，因为你很少会将大量数据载入到`Errors`表中。
* 违反唯一键或索引约束的直接路径插入运算将会引起语句失败并进行回滚。
* 任何违反唯一键或索引约束的更新运算将会引起语句失败并进行回滚。
* 任何违反延迟约束的运算都将会引起语句失败并进行回滚。
* `LOG ERRORS`子句不会追踪`LOB`、`LONG`或对象类型列的值。它可以与包含这些不支持的数据类型列的表一起使用但是这些不支持的数据类型的列将不会插入到`Errors`表中。要想为一张包含不支持的数据类型列的表创建`Errors`表，你必须使用`CREATE_ERROR_LOG`存储过程的`SKIP_UNSUPPORTED`参数。这个参数的默认值为`FALSE`，这会在尝试为包含不支持的数据类型列的表创建`Errors`表时导致存储过程失败。

下列代码清单给出了当基表中含有不支持的数据类型的列时，创建`Errors`表的正确语法。

> DBMS\_ERRLOG.CREATE\_ERROR\_LOG参数

{% code lineNumbers="true" %}
```sql
exec DBMS_ERRLOG.CREATE_ERROR_LOG(
    err_log_table_owner => '&owner',
    dml_table_name => '&table_name',
    err_log_table_name => '&err_log_table_name',
    err_log_table_space => NULL,
    skip_unsupported => TRUE
);
```
{% endcode %}

正如你所看到的，`INSERT`语句有一些你很少使用的选项。在我看来，这些功能中最有用的就是`DML`错误日志(也可以与其他`DML`命令一起使用)。它可以使得非常困难的问题例如冲突问题很容易被识别，并且与否则要进行的一行一行处理的方式比起来性能要高得多。还要注意直接路径插入相比较于常规插入方法而言所带来的性能上的极大提升。尽管在可回收性和序列化方面有其缺点，但对于大数据量加载，优点要比缺点显著得多。

## UPDATE

对一张表更新超过10亿行的系统，一个对全年的预测,每个值每天晚上重新计算一遍。抛开对于一个具有90天轮回周期的事物做那么久的预测是否有必要不提,对于10亿条数据记录来说,从临时文件中加载比更新要快得多。

传统的做这种类型处理的方法是首先进行截断然后重新加载。但如果不能采用截断然后重新加载的方法呢?一种替代方法就是使用`Create Table As Select(CTAS)`来创建一张新表，然后用新创建的表替换原来那张表。如果你说得很快，这听起来似乎很容易。当然，还有很多细节必须要解决。下列代码清单快速说明了这两种方法可能存在的性能上的差异。

> UPDATE与CTAS之间的性能三角形

{% code lineNumbers="true" %}
```sql
update skew2 set col1 = col1*1;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>UPDATE</p></figcaption></figure>

{% code lineNumbers="true" %}
```sql
create table skew_temp as 
select pk_col,col1*1 col1,col2,col3,col4 from kso.skew2;
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>CTAS</p></figcaption></figure>

你可以看到，更新需要将近30分钟(1676.78秒)而`CTAS`仅需要不到1分钟(44.30秒)。因此很明显，重建表比更新所有记录能够得到显著的性能优势。并且可能你已经从前一个例子中预见到了，重建表可能也比更新相对较小比例的数据行效率更高。

## DELETE

与大量更新类似，大量删除通常也不是个好主意。通常重建一张表或一个分区(不包含你想要清除的数据行)比删除表中很大百分比的数据行要更快(可能也稍微更复杂一点)。重建方法的最大弊端就是在重建期间必须对对象进行保护以阻止其他修改。这种方法基本与我在前面一节中所描述的UPDATE命令中的方法相同，但DELETE可能会花费更多的时间。

基本思想与大数据量更新是一样的。

* (1)创建一张临时表。
* (2)将不需要删除的记录放到临时表中。
* (3)重建相关对象(索引、约束、授权以及触发器)。
* (4)重命名表。

{% hint style="info" %}
如果你需要删除一张表或分区内的所有数据行，可以使用`TRUNCATE`命令来进行。截断一张表是移动高水位线而不是实际修改保存数据记录的块。因此与使用`DELETE`命令比较起来它非常快。只有少数几点非常次要的缺点。

* 它是一条DDL命令，因此它引起一次隐式提交(一旦截断了一张表，就不能再还原了)。
* 你不能闪回到表截断之前的状态。
* 只能对整张表进行截断，或者不截断。

除了执行极其快以外，TRUNCATE命令还可能会对表以后的查询产生很大的影响。由于全表扫描需要读取所有高水位线以下的数据块，而DELETE命令对高水位线没有任何影响，你可能会为了以后的查询语句而放弃性能上的优势。
{% endhint %}

## MERGE

`MERGE`语句是在`ORACLE`数据库`9i`版本中引入的。它提供了经典的`UPSERT`功能。`MERGE`将会在已经存在这条记录的时候进行更新,或者在不存在这条记录的时候插入新记录(`Oracle`数据库`10g`版本对`MERGE`命令进行了增强，允许其删除记录)。其思想就是在必须使用附加的`SQL`语句的时候削减额外的用来进行错误检验以及与数据库之间往返(即写一小段代码来尝试进行更新，检查更新状态，并且如果更新失败了则进行插入)的代码。`MERGE`语句不需要任何额外的代码就可以在数据库级来完成所有这些工作。很显然，它的性能要比过程式版本的代码高。

### 语法和用法

典型的MERGE语句的语法相对简单。下面是一个MERGE语句的基本语法。

{% code lineNumbers="true" %}
```sql
MERGE INTO table_name
USING (subquery) ON (subquery.column = table.column)
WHEN MATCHED THEN UPDATE ...
WHEN NOT MATCHED THEN INSERT ...
```
{% endcode %}

`MERGE`语句的第一部分看上去就像一条`INSERT`语句,指定插入、更新或删除数据的表(或视图)。关键字`USING`声明了数据源(通常是一个子查询，尽管也可以是一张分级表)以及一个告诉`Oracle`如何来确定某条记录是否已经在目标表中存在的联结条件。除此以外，你必须加上一个`UPDATE`子句或`INSERT`子句或者两者都加上。在大多数情况下，你会看到两者都加上了，因为很少有使用`MERGE`语句而不用这两个子句的情况。现在让我们转移到`UPDATE`和`INSERT`子句上来(可能更应该将其称为`When Matched`和`When Not Matched`子句)。

`UPDATE`子句告诉`Oracle`当找到了一条相匹配的记录时要怎么做。在多数情况下，找到一条匹配记录将会对其进行更新。也可以有一个可选的`WHERE`子句来限制在找到了一条匹配记录的时候，哪些记录需要进行更新。或者，你可以使用另一条`WHERE`子句来将匹配的记录删除掉。注意被删除掉的记录必须通过主`WHERE`子句中的筛选标准以及`DELETE WHERE`子句中的筛选标准。实际上`DELETE`子句并不经常使用。但如果要做一些除了加载数据之外的工作,它也可以很方便地来进行。例如，某些`ETL`处理过程还有数据清理的任务。要使得`UPDATE`子句的`DELETE`部分生效，匹配的记录必须通过`UPDATE`子句中`WHERE`条件的筛选并且还要通过相应的`DELETE`子句中`WHERE`条件的选，下列代码清单给出了一个`MERGE`命令，其中的`UPDATE`子句含有`DELETE`。

> 具有UPDATE子句的MERGE

{% code lineNumbers="true" %}
```sql
MERGE INTO kso.big_emp t
USING (select * from hr.employees) s
ON (t.employee_id = s.employee_id)
WHEN MATCHED THEN UPDATE SET
    -- t.employee_id = s.employee_id ON clause columns not allowed
    t.first_name = t.first_name,
    t.last_name = s.last_name,
    t.email = s.email,
    t.phone_number = s.phone_number,
    t.hire_date = s.hire_date ,
    t.job_id = s.job_id ,
    t.salary = s.salary ,
    t.commission_pct = s.commission_pct ,
    t.manager_id = s.manager_id ,
    t.department_id = s.department_id 
    WHERE (s.salary <= 3000)
    DELETE WHEN (s.job_id = 'FIRED');
```
{% endcode %}

`INSERT`子句告诉`Oracle`当没有找到匹配记录的时候要怎么做。通常，这意味着“进行一次数据插入”。然而，`INSERT`子句可能会被完全停掉。也可以在其中应用一个可选的`WHERE`子句，从而在没有找到匹配的时候也不一定非要插入新的记录。下列代码清单给出了两个版本的含有`INSERT`子旬的`MERGE`语句。

> 具有INSERT子句的MERGE

{% code lineNumbers="true" %}
```sql
MERGE INTO bit_emp t
USING (select * from hr.employees) s
ON (t.employee_id = s.employee_id)
WHEN NOT MATCHED THEN INSERT(
    t.employee_id,
    t.first_name,
    t.last_name,
    t.email,
    t.phone_number,
    t.hire_date,
    t.job_id ,
    t.salary,
    t.commission_pct,
    t.manager_id,
    t.department_id)
    VALUES
    (s.employee_id,
    s.first_name,
    s.last_name,
    s.email,
    s.phone_number,
    s.hire_date,
    s.job_id ,
    s.salary,
    s.commission_pct,
    s.manager_id,
    s.department_id)
    WHERE (s.job_id != 'FIRED');
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
MERGE INTO bit_emp t
USING (select * from hr.employees where job_id != 'FIRED') s
ON (t.employee_id = s.employee_id)
WHEN NOT MATCHED THEN INSERT(
    t.employee_id,
    t.first_name,
    t.last_name,
    t.email,
    t.phone_number,
    t.hire_date,
    t.job_id ,
    t.salary,
    t.commission_pct,
    t.manager_id,
    t.department_id)
    VALUES
    (s.employee_id,
    s.first_name,
    s.last_name,
    s.email,
    s.phone_number,
    s.hire_date,
    s.job_id ,
    s.salary,
    s.commission_pct,
    s.manager_id,
    s.department_id);
```
{% endcode %}

两个语句实现了同样的功能但使用了稍微不同的机制。其中一个在`USING`子句的子查询中限定了用来进行合并的记录，而另一个在`INSERT`子句中的`WHERE`条件里限定进行合并的记录。需要知道的是这两种形式的性能特点可能会不同，甚至可能会产生不同的执行计划。下面代码清单给出了一个同时具有`INSERT`子句和`UPDATE`子句的更实际的例子。注意`UPDATE`子句也包含一个`DELETE WHERE`子句用来将已经被解雇的员工记录清除掉。

> 完整的MERGE

<pre class="language-sql" data-line-numbers><code class="lang-sql">MERGE /*+ */ INTO bit_emp t
USING (select * from hr.employees) s
ON (t.employee_id = s.employee_id)
WHEN MATCHED THEN UPDATE SET
    -- t.employee_id = s.employee_id ON clause columns not allowed
    t.first_name = t.first_name,
    t.last_name = s.last_name,
    t.email = s.email,
    t.phone_number = s.phone_number,
    t.hire_date = s.hire_date ,
    t.job_id = s.job_id ,
    t.salary = s.salary ,
    t.commission_pct = s.commission_pct ,
    t.manager_id = s.manager_id ,
    t.department_id = s.department_id 
    WHERE (s.salary &#x3C;= 3000)
    DELETE WHEN (s.job_id = 'FIRED')
<strong>WHEN NOT MATCHED THEN INSERT(
</strong>    t.employee_id,
    t.first_name,
    t.last_name,
    t.email,
    t.phone_number,
    t.hire_date,
    t.job_id ,
    t.salary,
    t.commission_pct,
    t.manager_id,
    t.department_id)
    VALUES
    (s.employee_id,
    s.first_name,
    s.last_name,
    s.email,
    s.phone_number,
    s.hire_date,
    s.job_id ,
    s.salary,
    s.commission_pct,
    s.manager_id,
    s.department_id)
    WHERE (s.job_id != 'FIRED');
</code></pre>

### 性能比较

那么`MERGE`语句与直接的`INSERT`或`CTAS`操作相比较怎么样呢?显然,`MERGE`语句中有一些固定的支出使得这样的一个比较并不是公平的测试。但是`MERGE`并不消极。记住正如`INSERT`命令一样进行大数据量载入最快的方法是使用`APPEND`提示来确保使用直接路径机制。下列代码清单对`INSERT`、`MERGE`和`CTAS`的性能进行了比较。其中还表明了它们都可以进行直接路径写人。

> INSERT、MERGE、CTAS性能比较

{% code lineNumbers="true" %}
```sql
truncate table skew3;
--00:00:32.92
create table skew3 as select * from skew;

truncate table skew3;
--00:00:31.23
INSERT /*+ APPEND */ INTO skew3 select * from skew;

truncate table skew3;
--00:00:49.07
MERGE /*+ APPEND */
INTO skew3 t
USING(select * from skew) s
ON (t.pc_col = s.pk_col)
WHEN NOT MATCHED THEN INSERT
(t.pk_col,t.col1,t.col2,t.col3,t.col4)
VALUES(s.pk_col,s.col1,s.col2,s.col3,s.col4);
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>INSERT、MERGE、CTAS性能比较</p></figcaption></figure>

在这个非常简单的测试中，你可以看到所有3种方法都可以应用直接路径写入，并且`CTAS`和`INSERT`的性能很接近。如你所预料的那样由于其额外的能力以及与这些能力相关的必要支出，`MERGE`语句要慢得多。但`MERGE`语句提供了最大的灵活性，因此不要忽略这个单独的`SQL`语句在一次执行过程中可以进行多种`DML`运算这样一个事实。

## 小结

有4个`SQL`命令可以用来修改数据:`INSERT`、`UPDATE`、`DELETE`和`MERGE`(最后一个实际上可以进行所有前面3种操作)。本章简要讨论了这些命令并聚焦于一个关键的性能概念:直接路径插入比常规的插人要快得多。对于这种性能上的差异有一个很好的解释--是因为直接路径插入做了少得多的工作。但是使用这种方法也有好几个弊端。最大的缺点就是它是一个串行的运算，在任何时候只有一个进程能够对表进行直接路径插入，其他任何想要做同样事情的进程则必须等待。另一个大的缺点就是直接路径插入将不会使用已经分配给表的空闲空间。由于这两个原因，它只适用于大批量数据的加载过程。尽管如此，它是将数据插入到表中最快的方法。也正因为如此，在任何性能是做出决定的最重要标准的时候都可以考虑使用。目前也已经开发出在进行更新和删除时使用的直接路径插入。在本章中你已经浏览过了几个这样的技术。最后，你还学到了一些不太著名的`DML`命令选项，包括可以在所有4种语句中进行应用的功能极其强大的`Error Logging`子句。
