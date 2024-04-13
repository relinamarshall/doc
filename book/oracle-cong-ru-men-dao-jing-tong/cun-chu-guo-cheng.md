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

# 存储过程

## 什么是存储过程

存储过程在数据库开发中使用比较频繁，它有着普通SQL语句不可替代的作用。

### 认识存储过程

所谓存储过程，就是一段存储在数据库中执行某种功能的程序，其中包含一条或多条SQL语句，但是它的定义方式和PL/SQL中的块、包等有所区别。存储过程可以通俗地理解为是存储在数据库服务器中的封装了一段或多段SQL语句的PL/SQL代码块。在数据库中有一些是系统默认的存储过程，那么可以直接通过存储过程的名称进行调用。另外，存储过程还可以在编程语言中调用，如Java、C#、VB等编程语言。

### 存储过程的作用

存储过程编写相对比较复杂，但很多单位或个人都在使用它。显然这不是因为存储过程编写简单，而是因为它有着一系列的优点:&#x20;

* 简化复杂的操作。存储过程可以把需要执行的多条SQL语句封装到一个独立单元中，用户只需调用这个单元就能达到目的。这样就实现了一人编写多人调用，同时缩短了平均开发周期，为公司节省了成本。&#x20;
* 增加数据独立性。与视图的效果类似，利用存储过程可以把数据库基础数据和程序(或用户)隔离开来，当基础数据的结构发生变化时，可以修改存储过程，这样对程序来说基础数据的变化是不可见的，也就不需要修改程序代码了。
* 提高安全性。使用存储过程有效地降低了错误出现的几率。如果不使用存储过程要想实现某项操作可能需要执行多条单独的SQL语句，而过多的执行步骤很可能造成更高的出错几率。不仅如此，实际工作中开发人员的水平参差不齐，由高水平的人编写存储过程，水平较低的人员直接调用，这样就能避免很多不必要的错误发生。此外，存储过程也可以进行权限设置。
* 提高性能。完成一项复杂的功能可能需要多条SQL语句，同时SQL每次执行都需要编译而存储过程可以包含多条SQL语句，而且创建完成后只需要编译一次，以后就可以直接调用，从这方面来看存储过程可以提高性能。如果程序语言要实现某项比较复杂的功能，它会多次连接数据库，在使用存储过程的情况下，程序只需连接一次就能达到目的。

## 存储过程的语法

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

## 创建存储过程

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

## 查看存储过程

存储过程一旦被创建就会存储到数据库服务器上，Oracle允许开发人员查看已经存在的存储过程脚本，这可以到视图USER\_SOURCE里查看。

```sql
SELECT * FROM USER_SOURCE WHERE NAME = 'TEST' ORDER BY LINE;
```

{% hint style="info" %}
当从视图USER\_SOURCE中查询过程或函数时需要把名称大写，小写会得不到想要的记录。至于USER\_SOURCE是当前用户的视图，如果想查看Oracle所有的存储过程则需要到ALL\_SOURCE视图中查询。
{% endhint %}

## 显示存储过程的错误

```sql
SHOW ERRORS PROCEDURE procedure_name;
```

## 无参存储过程

<pre class="language-sql"><code class="lang-sql">CREATE PROCEDURE PRODUCT_UPDATE_PRC
AS
BEGIN
<strong>    UPDATE PRODUCTINFO SET DESC = '促销产品' WHERE ID = '1';
</strong>    COMMIT;
END;
</code></pre>

## 存储过程中使用游标

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

## 存储过程中的DDL语句

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

## 有参存储过程

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

## 修改存储过程

修改存储过程内容要利用REPLACE关键词，从而完成过程的修改也就是覆盖。

```sql
CREATE OR REPLACE PROCEDURE procedure_name
IS 
BEGIN
    ...
END;
```

## 删除存储过程

```sql
DROP PROCEDURE [schema.]procedure_name

【说明】
schema:表示存储过程的所属机构。
procedure_name:表示要删除存储过程的名称
```

{% hint style="warning" %}
当存储过程有调用关系时，如果删除被调用者、那么重新编译调用者会出现错误。所以做删除操作时最好认清各过程之间的依赖关系。
{% endhint %}

## 小结

存储过程的使用十分普遍，它用做批量处理任务和内部数据转换时效率很高，也是Oracle的重点之一。存储过程的使用方式灵活多变，读者在实际开发中应多读代码学习不同的使用方式，达到融会贯通的目的。
