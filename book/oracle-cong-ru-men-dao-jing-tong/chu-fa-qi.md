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

# 触发器

## 什么是触发器

### 认识触发器

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

### 触发器的作用

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

### 触发器的类型

触发器可分为5种类型，具体内容如下:&#x20;

* **数据操纵语言(DML)触发器。**\
  此种类型触发器定义到表上，当对表执行INSERTUPDATE、DELETE操作时可以激发该类型的触发器。利用该类触发器可以复制、检查替换某种符合指定条件的数据。按照触发级别可以分为两种方式，第一种为行级触发器，此种类型表示每条记录修改时都会激发该触发器，第二种为语句级触发器，此种类型表示当SQL语句执行时会激发该触发器，与修改多少条记录没有关系。如果以数据的更改事件为准，则分为BEFORE和AFTER两种类型。
* **数据定义语言(DDL)触发器。**\
  当CREATE、ALTER、DROP模式对象时会触发相关的触发器，在Oracle中可以简单地理解一个用户就有一个和它同名的模式，利用它可以使得某些表不能被修改或删除。 口复合触发器。此种类型的触发器是Oracle1lg的新特性，它相当于在一个触发器中包含了4种类型的触发器，其中包含了BEFORE类型的语句级、BEFORE类型的行级AFTER类型的语句级、AFTER类型的行级。这种把所有触发器都放到一个代码块中的 做法使得变量的传递变得更加方便。
* **INSTEAD OF触发器。**\
  此种类型触发器通常作用在视图上。对由多个源表的视图做DML操作通常是不被允许的，如果遇到这种情况就可以利用`INSTEAD OF`类型触发器解决问题。利用它可以把对视图的DML操作转换成对多个源表进行操作。
* **用户和系统事件触发器。**\
  作用在数据库上由数据库事件激发的触发器，如登录和注销事件的触发器。利用它可以记录数据库的登录情况。

{% hint style="info" %}
有些地方把DML和DDL类型的触发器直接分为行触发器、语句触发器、BEFORE触发器、AFTER触发器。
{% endhint %}

## 触发器语法

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

### DDL事件列表

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

### 数据库事件列表

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

## 操作触发器

### 创建触发器

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

### 查看触发器

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

### 设置触发器可用

<pre class="language-sql" data-line-numbers><code class="lang-sql"><strong>ALTER TRIGGER [schema.]trigger DISABLE|ENABLE;
</strong><strong>
</strong><strong>--禁用 LOGIN_TGR 触发器
</strong><strong>ALTER TRIGGER LOGIN_TGR DISABLE;
</strong></code></pre>

### 修改触发器

触发器内容的修改同样要利用REPLACE关键词。创建触发器时带上OR REPLACE关键词，从而完成过程的修改，也就是覆盖。

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER LOGIN_TGR
...
```
{% endcode %}

### 删除触发器

{% code lineNumbers="true" %}
```sql
DROP TRIGGER [schema.]trigger

--删除 LOGIN_TGR 触发器
DROP TRIGGER LOGIN_TGR;
```
{% endcode %}

## DML类型触发器

该类型触发器在日常开发中比较常用，前面已经介绍过DML触发器主要针对表的INSERT、UPDATE、DELETE操作。

### 创建行级触发器

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

### 使用多种触发事件

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

### 使用IF语句

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

### 使用WHEN限制条件

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

### 语句级触发器

{% code lineNumbers="true" %}
```sql
CREATE OR REPLACE TRIGGER PRODUCTINFO_OPER_TGR
    BEFORE INSERT OR UPDATE OR DELETE ON PRODUCTINFO
BEGIN
    DBMS_OUTPUT.PUT_LINE('语句级触发器测试');
END;
```
{% endcode %}

## 触发器执行顺序

在同一对象上可以作用多个触发器，因此触发器被激发的顺序是有先后关系的。其触发顺 序如下:

* 首先被触发的将是前语句级触发器(`before statement trigger`)，该触发器会被执行一次。
* 如果有行级的触发器则接下来执行前行级触发器(`before row trigger`)，行级触发器执行的次数同SOL修改的记录次数一致。&#x20;
* 当SQL修改记录完成后会触发行级触发器，这时候的行级触发器为后行级触发器(`after row trigger`)，该类型触发的次数同SQL修改记录的次数一致。&#x20;
* 执行一次语句级的触发器，此时的语句级触发器为后语句级触发器(`after statement trigger`)。

&#x20;如果多个相同类型的，相同事件触发器作用在同一个对象上，如果在Oracle11g之前，那么最终被执行的会有一定的随机性，而在Oracle 11g中利用`FOLLOWS`可以控制其顺序。

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

## 复合类型触发器

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

## INSTEAD OF类型触发器

在该类型的触发器作用下，如果对作用对象执行DML操作，那么该操作会被触发器的内部操作所取代。触发器作用在视图当中，用于解决视图不可更新的问题。

### 创建视图

{% code lineNumbers="true" %}
```sql
CREATE VIEW PRODUCTINFO_VIEW AS 
SELECT DISTINCT * FROM PRODUCTINFO
```
{% endcode %}

### 创建触发器

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

## DDL类型触发器

所谓DDL类型触发器，就是因DDL操作而激发的触发器，主要包括CREATE、ALTER、DROP等事件。

利用DDL类型的触发器可以限制和记录特定的DDL操作。例如，通过DDL触发器可以限制对数据库结构的修改，记录数据库中的更改事件，也可以在修改对象的时候根据实际情况做出其他相应动作。下面通过一个示例演示如何创建和使用该类型的触发器。

### 创建触发器

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

### 测试触发器

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

### 常用事件属性

<table><thead><tr><th width="241">属性函数</th><th width="228">可用的事件</th><th>简介</th></tr></thead><tbody><tr><td>SYSEVENT</td><td>所有事件</td><td>返回激发触发器的事件名称</td></tr><tr><td>INSTANCE_NUM</td><td>所有事件</td><td>返回当前数据库的示例号</td></tr><tr><td>DATABASE_NAME</td><td>所有事件</td><td>返回当前数据库名</td></tr><tr><td>SERVER_ERROR</td><td>SERVERERROR</td><td>错误堆栈的指定位置返回错误号</td></tr><tr><td>LOGIN_USER</td><td>所有事件</td><td>返回激发触发器的用户名</td></tr><tr><td>DICTIONARY_OBJ_TYPE</td><td>CREATE\ALTER\DROP</td><td>返回激活触发器的DDL操作的对象类型</td></tr><tr><td>DICTIONARY_OBJ_NAME</td><td>CREATE\ALTER\DROP</td><td>返回激活触发器的DDL操作的对象名称</td></tr></tbody></table>

## 用户和系统事件触发器

所谓系统事件触发器，就是基于Oracle系统事件而建立的触发器。该类型的触发器可以审计数据库的登录、注销以及关闭和启动等。

### 创建数据库级触发器

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

### 测试触发器

{% code lineNumbers="true" %}
```sql
--重新登录后查看日志
SELECT * FROM LOG_USER;
```
{% endcode %}

## 小结

触发器是数据库的重要组成部分，利用触发器可以完成更复杂的业务逻辑，也可以对数据库的操作行为进行记录。DML类型触发器、DDL类型触发器、复合类型触发器、替代类型触发器、系统事件触发器，可以根据实际情况选择一种或几种使用，来实现更复杂的功能。
