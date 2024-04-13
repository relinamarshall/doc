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

# PL/SQL基础

## 什么是PL/SQL

### 认识PL/SQL

结构化查询语言(`Structure Query Language`，SQL)是用来访问和操作关系型数据库的-种标准通用语言，它属于第四代语言(4GL)，简单易学，使用它可以很方便地调用相应语句来取得结果。该语言的特点就是非过程化。也就是说，使用的时候不用指明执行的具体方法和途径，即不用关注任何的实现细节。但这种语言也有一个问题，就是在某些情况下满足不了复杂业务流程的需求，这就是第四代语言的不足之处。Oracle中的PL/SQL语言正是为了解决这一问题，PL/SQL属于第三代的语言(3GL)，也就是过程化的语言，同Java、C#一样可以关注细节，用它可以实现复杂的业务逻辑，是数据库开发人员的利器。

PL/SQL(Procedural Language/Structured Query Language)是Oracle公司在标准SQL语言基础上进行扩展而形成的一种可以在数据库上进行设计编程的语言，通过Oracle的PL/SQL引擎执行。PL/SQL完全可以像Java语言一样实现逻辑判断、条件循环以及异常处理等，这是标准的SQL很难办到的事情。由于它的基础是标准的SQL语句，这就使得数据库开发人员能快速地掌握并运用，相信这也是Oracle开发人员喜爱它的另一个重要原因。总的来说，PL/SQL有以下几个特点:

* 支持事务控制和SQL数据操作命令
* 支持SQL的所有数据类型，并且在此基础上扩展了新的数据类型，也支持SQL的函数以及运算符。
* PL/SOL可以存储在Oracle服务器中。
* 服务器上的PL/SQL程序可以使用权限进行控制。
* Oracle有自己的DBMS包，可以处理数据的控制和定义命令，

### PL/SQL的优势

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

## PL/SQL的结构

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

## PL/SQL的基本规则

1.  PL/SQL中允许出现的字符集:

    字母，包括大写和小写。

    数字，即0\~9。

    空格、回车符以及制表符。

    符号包括`+、-、*、/、<、>、=、!、~、^、;、:、.、'、@、%、,、"、#、$、&、_、1、(、)、[、]、{、}、?`。
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

## PL/SQL的注释

单行注释：使用“--”两个短横线，可以注释掉后面的语句。

多行注释：使用“/\*..\*/”，可以注释掉这两部分包含的部分。

## PL/SQL变量的使用

所有的编程语言中变量是使用最频繁的。PL/SQL作为一个面向过程的数据库编程语言同样少不了变量，利用变量可以把PL/SQL块需要的参数传递进来，做到动态执行程序，同时也可以利用变量在PL/SOL内部进行值的相互传递，其至可以把值传递出去，最终返回给用户由此可见，变量是PL/SQL不可缺少的一部分。

### 变量、常量的类型及语法

变量就是它所表示的值是可以变化的，而常量就是当初始化后，其值不可以再改变。PL/SQL是一种强类型的语言，所以当使用变量或常量的时候必须声明，否则提示错误。下面是变量和常量声明的语法介绍。

### 变量声明语法结构

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

### 常量声明语法结构

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

### 标量类型的变量

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

### 复合类型的变量

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

## 表达式

数据库中经常使用表达式来计算结果，尤其在变量和常量的使用过程中。在前面已经接触过表达式的使用，它和普通编程语言的表达式很类似。本节将系统地介绍表达式的类型以及如何使用表达式。 表达式根据操作数据类型的不同可以分为如下几类:

* 数值表达式;
* 关系表达式;
* 逻辑表达式。

下面对这几个表达式进行讲解。

### 数值表达式

数值表达式就是由数值类型的常量、变量以及函数，由算术运算符连接而成。在PL/SQL可以使用的算术运算符有:

* 加号+；减号-；乘号\*；除号/；乘方\*\*；

### 关系表达式和逻辑表达式

由关系运算符连接起来的字符或数值称为关系表达式。其中关系运算符主要有以下几种:

* 等于号=；小于号<；大于号>；小于等于号<=；大于等于>=；不等于号!=和<>。

所谓逻辑表达式，就是由逻辑符号和常量或变量等组成的表达式。逻辑符号比较少，常见的有下面3种:

* 逻辑非NOT；逻辑或OR；逻辑与AND。

{% hint style="info" %}
关系表达式中的等于号和赋值符号“:=”的差异。
{% endhint %}

## PL/SQL结构控制

### IF条件控制语句

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

### CASE条件控制语句

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

### LOOP循环控制语句

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

## PL/SQL的DML\DDL语言

PL/SQL当中除了可以执行查询语句外，也允许执行DML语言和DDL语言。结构控制和DML操作相结合能帮助我们完成很多复杂的业务。不过在PL/SQL中DML语言和DDL语言这两者的执行方式有所差异。

### DML语句的使用

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

### DDL语句的使用

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

## PL/SQL中的异常

同其他语言的程序一样，用PL/SQL编写的程序在运行过程当中也会出现各种错误，这些错误也可以称为异常。异常在PL/SQL中是很重要的一部分。

### 什么是异常

PL/SQL运行过程中有可能会出现错误，这些错误有的来自程序本身，也有的来自开发人员自定义的数据，而所有的这些错误我们称之为异常(编译时的错误不能称为异常)。为了使程序有更好的阅读性和健壮性，PL/SQL采用了捕获并统一处理异常的方式。当异常发生时，程序会无条件跳转到异常块处，将控制权限交给异常处理程序。异常处理程序将进行异常匹配，如果当前的块内没有对应的异常名称，则异常会传至当前程序的上一层程序中查找，如果一直向上查找却依然没有找到对应的处理方式,则该异常会被传至当前的主机调用中，而运行的程序也会中断。

### 处理异常的语法

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

### 预定义异常

Oracle中为每个错误提供一个错误号，而捕获异常则需要异常有名称。Oracle提供了一些已经定义好名称的常用异常，这就是预定义异常。例如，前面在使用SELECT...INTO语句时，如果返回超过一条记录就会触发TOO\_MANY\_ROWS异常。

<figure><img src="../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption><p>预定义异常</p></figcaption></figure>

Oracle一共提供了25种预定义异常。利用下面的查询语句可以查看Oracle的预定义异常:

```sql
SELECT * FROM DBA_SOURCE 
WHERE NAME='STANDARD' AND TEXT LIKE '%EXCEPTION_INIT%';
```

### 非预定异常

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

### 自定义异常

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

## PL/SQL函数编写

函数由PL/SQL定义，可以操作各种数据项目完成计算并返回计算的结果值。利用函数可以把复杂的计算过程封装起来，避免所有的开发人员面对复杂的算法。

### 函数的组成

函数主要由以下几个部分组成:

* 输入部分。函数允许有输入的参数，调用函数时需要给这些参数赋值。
* 逻辑计算部分。函数内部将完成对各种数据项目的计算，它可以进行很简单的算术表达式运算，也可以调用多个SQL内置函数或自定义函数进行运算。
* 输出部分。函数要求都有返回值。

### 函数语法

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

### 查看函数

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

### 函数的修改\删除

当函数创建完成后，如果业务发生变化，那么函数脚本也需要修改。函数内容的修改需要REPLACE关键词。

函数的删除同样简单。下面是删除函数的语法:

{% code lineNumbers="true" %}
```sql
DROP FUNCTION [schema.]function

schema:函数所属的模式名称。
```
{% endcode %}

## 小结

PL/SQL是操作Oracle的基础，日常开发中也是最常使用的部分。本章利用大量的篇幅介绍了PL/SQL基础方面的知识。下面概括了本章的主要知识点:

* 什么是PL/SQL，PL/SQL的优势和PL/SQL块的结构。这是PL/SQL的基础，在这部分应当了解PL/SQL块分为声明部分、执行体部分和异常处理部分。
* PL/SQL中的变量和常量。介绍了变量和常量的类型。概括来说，变量类型可分为标量类型和复合类型。&#x20;
* 有关表达式在PL/SQL编程中或标准SQL语句中都有使用。
* PL/SQL既然是编程语言，那么它就有结构控制语句，这里所涉及的结构控制语句有IF语句、CASE语句和LOOP语句。这3种结构中都有分支结构，是PL/SQL编程必不可少的一项。&#x20;
* PL/SQL块中允许执行SELECT...INTO语句为变量赋值，也允许执行DML和DDL语句，不过执行DDL需要相关的命令。&#x20;
* PL/SQL中很重要的一部分就是异常处理。Oracle采用了统一处理异常的方式，当PL/SQL块中发生异常时，程序会立刻无条件地转到异常处理部分，然后与异常名称列表进行匹配，如果匹配成功则表示异常被捕捉到，否则异常会一直向外层传递，直到主机调用界面。&#x20;
* 利用PL/SQL可以自定义函数，函数允许查询数据，但却不允许对数据库进行操作，使用函数主要对数据项进行计算。
