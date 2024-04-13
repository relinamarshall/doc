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

# 内置函数

Oracle有多种内置函数，本章将重点介绍其中的两种，它们分别是单行函数和集合函数。这两种类型的函数使用频率比较高。

单行函数是指当查询表或视图时每行都能返回一个结果，可用于SELECT、WHERE ORDER BY等子句中。而集合函数是作用在多行记录上返回一个结果，可用于带GROUPBY或HAVING子句的查询中。单行函数数量比较多，这里介绍其中常用的几种类型，它们分别是数值型函数、字符型函数、日期型函数、转换函数等。

介绍函数之前先简单介绍一下Oracle的DUAL表。该表是Oracle中真实存在的一个表，任何用户都可以读取，多数情况下可以用在没有目标的SELECT查询语句中。它本身只包含了一个DUMMY字段。DUAL表对Oracle很重要，用户不要试图删除该表，一旦删除，Oracle将无法启动。下面的函数讲解中会以DUAL表作为测试语句的目标表。

## 数值型函数

### 绝对值\取余\判断正负

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

### 三角函数

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

### 取整数

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

### 指数\对数

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

### 四舍五入截取

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

## 字符型函数

### ASCI码与字符转换

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

### 获取字符串长度

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

### 字符串截取

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

### 字符串连接

> CONCAT(char1.char2)函数

该函数连接两个参数并返回。char2将连接到char1的尾部。效果和连接符“11”相似。参数类型可以是CHAR、VARCHAR2、NCHAR、NVARCHAR2、CLOB、NCLOB。示例脚本如下:

```sql
SELECT CONCAT('我的'，'测试!'),'我的'||'测试!' FROM DUAL;

CONCAT('我的'，'测试!')|'我的'||'测试!'|
------------------+-----------+
我的测试!             |我的测试!      |
```

### 字符串搜索

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

### 字母大小写转换

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

### 带排序参数的字母大小写转换

> NLS\_INITCAP(char\[,nlsparam])函数

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

### 为指定参数排序

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

### 替换字符串

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

### 字符串填充

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

### 删除字符串首尾指定字符

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

### 字符集名称和ID互换

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

## 日期型函数

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

### 系统日期\时间

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

### 数据库时区

> DBTIMEZONE函数

该函数没有参数，返回数据库时区。示例脚本如下：

```sql
SELECT DBTIMEZONE FROM DUAL;

DBTIMEZONE|
----------+
+00:00    |
```

### 日期加上指定月份

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

### 获取月份最后一天

> LAST\_DAY(date)函数

该函数返回参数指定日期对应月份的最后一天。示例脚本如下：

```sql
SELECT LAST_DAY(SYSDATE) FROM DUAL;

LAST_DAY(SYSDATE)      |
-----------------------+
2024-03-31 14:16:38.000|
```

### 获取下周日期

> NEXT\_DAY(date,char)函数

该函数返回当前日期向后的一周char的对应日期，char表示的是星期几，全程和缩写都允许。但必须有效，示例脚本如下：

```sql
SELECT SYSDATE, NEXT_DAY(SYSDATE,'星期一') FROM DUAL;

SYSDATE                |NEXT_DAY(SYSDATE,'星期一')|
-----------------------+-----------------------+
2024-03-16 14:19:00.000|2024-03-18 14:19:00.000|
```

### 会话所在时区当前日期

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

### 提取指定日期特定部分

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

### 日期之间的月份数

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

### 时区时间转换

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

### 日期四舍五入\截取

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

## 转换函数

转换函数可以完成不同数据类型之间的转换，是平常使用比较多的函数类型之一。本节将介绍平常使用率较高的转换函数。

### 字符串转ASCII类型

> ASCIISTR(char)函数

该函数可将任意字符集的字符串转换为数据库字符集对应的ASCII字符串。char为字符类型。示例脚本如下:

```sql
SELECT ASCIISTR('这是测试!') FROM DUAL;

ASCIISTR('这是测试!')    |
---------------------+
\8FD9\662F\6D4B\8BD5!|
```

### 二进制转十进制

> BIN\_TO\_NUM(data\[,data...])函数

该函数可以将二进制转换成对应的十进制。data表示二进制数，一位用“,”隔开。示例脚本如下:

```sql
SELECT BIN_TO_NUM(1), BIN_TO_NUM(1,0,0),BIN_TO_NUM(1,1,1) FROM DUAL;

BIN_TO_NUM(1)|BIN_TO_NUM(1,0,0)|BIN_TO_NUM(1,1,1)|
-------------+-----------------+-----------------+
            1|                4|                7|
```

### 类型转换

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

### 字符串转ROWID

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

### 字符集间转换

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

### 十六进制转RAW

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

### 数值转换成字符型

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

### 字符转日期型

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

### 字符串转数字

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

### 全角转半角

> TO\_SINGLE\_BYTE(char)函数

该函数将全角转为半角。char的类型可以是CHARVARCHAR2、NCHAR、NVARCHAR2。

示例脚本如下:

```sql
SELECT TO_SINGLE_BYTE('THIS IS A TEST') FROM DUAL;

TO_SINGLE_BYTE('THISISATEST')|
-----------------------------+
THIS IS A TEST               |
```

## NULL函数

NULL函数是用来处理空值时比较好的选择。本节介绍几个常用的空值处理函数。

### 返回不为NULL表达式

> COALESCE(expr)函数

返回列表中第一个不为null的表达式。如果都为null，则返回一个null。示例脚本如下:

```sql
SELECT COALESCE(NULL,9-9,NULL) FROM DUAL;

COALESCE(NULL,9-9,NULL)|
-----------------------+
                      0|
```

### 排除指定条件

> LNNVL(condition)函数

该函数可以得到除了condition要求条件之外的数据，包括NULL的条件，通常用于WHERE条件中。下面的示例将得到PRODUCTINFO表中数量低于70的产品 并包含数量为NULL的数据。示例脚本如下:

```sql
SELECT * FROM PRODUCTINFO WHERE LNNVL(QUANTITY>=70);
```

### 替换NULL值

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

## 集合函数

集合的数经常配合`GROUP BY`或`HAVING`子句使用，当然它们也可以单独使用。该类型的函数中除了`COUNT`函数都会忽略列值为`NULL`的数据。

### 求平均值

> AVG(\[distinctlall]expr)函数

该函数可求取指定列的平均值，表示某组的平均值，返回数值类型。各参数表示的具体含义如下:

* distinct:去除重复的值。
* all:表示所有的值，包括重复的值，也是默认值。
* expr:表达式。只能是数值类型。

```sql
SELECT AVG(ALL column_name) FROM DUAL;
```

### 求记录数量

> COUNT(\* l \[distinct] \[all] expr)函数

该函数可以用来计算记录的数量或某列的个数。函数中必须指定列名，或全选使用星号。其中各参数表示的含义如下:

* \*:表示计算所有记录。
* distinct:表示去除重复的记录
* all:代表所有的，是默认选项。
* expr:要计算的对象，通常是表的列。

### 返回最大\小值

> MAX(\[distinct l all]expr)函数

该函数可以返回指定列中的最大值，通常都用在WHERE子句中的子查询。其中各参数表示的含义如下:

* distinct:表示去除重复的记录。
* all:代表所有的，是默认选项。
* expr:表的列。

同该函数效果相反但用法一致的有MIN(\[distinctlall]expr)函数，此函数获取指定列中的最小值。这里不再给出示例。

### 求和

> SUM(\[distinct l all]expr)函数

该函数不同于COUNT函数，它分组计算指定列的和，如果不使用分组，则的数默认把整个表作为一组。各参数代表含义如下:

* distinct:表示去除重复的记录。
* all:代表所有的，是默认选项。
* expr:表的列。

## 其他函数

前面介绍了单行的数以及集合函数，下面介绍几个不属于前面类型的函数，主要是系统环境和编码方面的函数。

### 返回登录名

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

### 表达式匹配

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
