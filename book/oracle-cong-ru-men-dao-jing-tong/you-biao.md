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

# 游标

## 什么是游标

游标的使用可以让用户像操作数组一样操作查询出来的数据集，这使得使用PL/SOL编程更加方便。实际上，它提供了一种从集合性质的结果中提取单条记录的手段。

### 游标的概念

可以将游标(Cursor)形象地看成一个变动的光标。它实际上是一个指针，它在一段Oracle存放数据查询结果集或数据操作结果集的内存中，这个指针可以指向结果集中的任何一条记录。这样就可以得到它所指向的数据了，但初始时它指向首记录。这种模型很像编程语言中的数组。&#x20;

可以简单地理解游标为指向结果集记录的指针，利用游标可以返回它当前指向的行记录(只能返回一行记录)。如果要返回多行，那么需要不断地滚动游标，把想要的数据查询一遍。用户可以操作游标所在位置行的记录。例如，把返回记录作为另一个查询的条件等。

### 游标的种类

Oracle中游标分为静态游标和REF游标两类。其中，静态游标就像一个数据快照，打开游标后的结果集是对数据库数据的一个备份，数据不随着对表执行DML操作后而改变。从这个特性来说，结果集是静态的。由于篇幅问题，本章将不对REF游标做介绍。

静态游标包含如下两种类型:&#x20;

* 显式游标：是指在使用之前必须有着明确的游标声明和定义，这样的游标定义会关联数据查询语句，通常会返回一行或多行。打开游标后，用户可以利用游标的位置对结果集进行检索，使之返回单一的行记录，用户可以操作此记录。关闭游标后，就不能再对结果集进行任何操作。显式游标需要用户自己写代码完成，一切由用户控制。
* 隐式游标：和显式游标不同，它被PL/SQL自动管理，也被称为SQL游标。由Oracle自动管理。该游标用户无法控制，但能得到它的属性信息。

## 显示游标

显式游标在PL/SQL编程当中有着重要的作用，通过显式游标用户可以操作返回的数据使得一些在编程语言中复杂的功能变得更容易实现。

### 游标语法

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

### 游标的使用步骤

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

### 游标中的LOOP语句

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

### BULK COLLECT FOR语句游标

游标中通常使用`FETCH...INTO..`语句提取数据，这种方式是单条数据提取，在数据量很大的情况下执行效率不是很理想。而`FETCH..BULK COLLECT INTO`语句可以批量提取数据在数据量大的情况下它的执行效率比单条提取数据的高。

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

### 使用CURSOR FOR LOOP

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

### 显式游标的属性

利用游标属性可以得到游标执行的相关信息。显式游标有以下4个属性:

* %ISOPEN:用于判断游标是否打开，如果已经打开则返回TRUE，如果游标未打开则返回FALSE。&#x20;
* %FOUND:此属性可用来检测行数据是否有效。如果有效该属性返回TRUE，否则返回FALSE。
* %NOTFOUND:与%FOUND属性恰好相反，如果没有提取出数据则返回TRUE，否则返回FALSE。&#x20;
* %ROWCOUNT:累计到当前为止使用FETCH提取数据的行数。

### 带参数的游标

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

## 隐式游标

隐式游标和显式游标有所差异，它虽然没有显式游标一样的可操作性，但在实际的工作中也经常用到。

### 隐式游标的特点

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

### 游标中使用异常处理

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

### 隐式游标的属性

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

## 小结

游标是提取Oracle数据的重要手段，它可以把符合要求的数据检索出来并存储到缓冲区中利用循环语句遍历整个结果集，从而获取每条记录，开发人员可以对每条记录进行操作。游标有4个属性，可以通过这4个属性得到游标的执行信息，也可以利用这些信息进行流程控制。游标中也允许异常处理，这和PL/SQL块一致。
