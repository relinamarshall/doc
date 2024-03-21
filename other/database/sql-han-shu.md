---
description: 公共函数支持Mysql，Oracle，PostgreSQL，SQLServer
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# SQL函数

## 公共函数

### ABS

```sql
-- ABS(x) 返回绝对值 
select area from (
    select 7500 as area from dual
    union
    select 8500 as area from dual
    union
    select 8600 as area from dual
) t
where abs(t.area-8000)<=500;
```

```
area|
----+
7500|
8500|
```

### AVG、COUNT

```sql
-- AVG(x) 返回平均数 忽略null值
-- COUNT(column_name) 查找列中非 null 值的数目。COUNT(*),COUNT(1) 会计算 null 值
select AVG(area),COUNT(1), COUNT(*),COUNT(area) from (
    select 7500 as area from dual
    union
    select 8500 as area from dual
    union
    select 8600 as area from dual
    union
    select null as area from dual
) t;
```

```
AVG(area)|COUNT(1)|COUNT(*)|COUNT(area)|
---------+--------+--------+-----------+
8200.0000|       4|       4|          3|
```

### CASE WHEN

```sql
-- CASE 允许在不同条件下返回不同的值
SELECT t.area,
    CASE WHEN t.area<=7500 THEN 'small'
    WHEN t.area<=8500 THEN 'medium'
    ELSE 'large' 
    END as 'size'
FROM (
    select 7500 as area from dual
    union
    select 8500 as area from dual
    union
    select 8600 as area from dual
    union
    select null as area from dual
) t;
```

<pre><code><strong>area|size  |
</strong>----+------+
7500|small |
8500|medium|
8600|large |
    |large |
</code></pre>

### CAST

```sql
-- CAST 从一种类型转换为另一种类型 Decimal类型四舍五入 通常，CAST 是隐式的
-- 如果将字符串与数字连接起来，则数字将自动更改为字符串。但是，有时您需要使 CAST 显式化
SELECT 
    CAST(1000000.1234 AS DECIMAL(10,3)) as a,
    CAST(1000000.1235 AS DECIMAL(10,3)) as b
FROM dual;
```

```
a          |b          |
-----------+-----------+
1000000.123|1000000.124|
```

### COALESCE

```sql
-- COALESCE 接受任意数量的参数，并返回第一个不为 null 的值
select 
    coalesce('a','b','c') as '1',
    coalesce(null,'b','c') as '2',
    coalesce(null,null,'c') as '3',
    coalesce(null,null,null) as '4'
from dual;
```

```
1|2|3|4|
-+-+-+-+
a|b|c| |
```

### REPLACE

```sql
-- REPLACE s1 的所有出现都替换为 s2
select REPLACE('vessel','e','a') from dual;
```

```
REPLACE('vessel','e','a')|
-------------------------+
vassal                   |
```

### ROUND

```sql
-- ROUND（f,p） 返回四舍五入到小数点后 p 位的 f
select ROUND(7253.86, 0), ROUND(7253.86, 1), ROUND(7253.86,-3) from dual;
```

```
ROUND(7253.86, 0)|ROUND(7253.86, 1)|ROUND(7253.86,-3)|
-----------------+-----------------+-----------------+
             7254|           7253.9|             7000|
```

### SIN、COS、TAN

```sql
-- SIN（f） 返回 f 的正弦值，其中 f 以弧度为单位
-- COS（f） 返回 f 的余弦值，其中 f 以弧度为单位
-- TAN（f） 返回 f 的正切值，其中 f 以弧度为单位
select 
    SIN(pi() / 6),
    COS(pi() / 3),
    TAN(pi() / 4)
from dual;
```

```
SIN(pi() / 6)      |COS(pi() / 3)     |TAN(pi() / 4)     |
-------------------+------------------+------------------+
0.5                |0.5               |1                 |
```

### MAX、MIN、SUM

```sql
-- MAX 最大值
-- MIN 最小值
-- SUM 将添加一整列值
select Max(area),Min(area),Sum(area) from (
    select 7500 as area from dual
    union
    select 8500 as area from dual
    union
    select 8600 as area from dual
) t;
```

```
Min(area)|Max(area)|Sum(area)|
---------+---------+---------+
     7500|     8600|    24600|
```

### NULLIF

```sql
-- NULLIF 如果两个参数相等，则 NULLIF 返回 NULL;否则 NULLIF 返回第一个参数
select NULLIF('1','2'), NULLIF('2','2') from dual;
```

```
NULLIF('1','2')|NULLIF('2','2')|
---------------+---------------+
1              |               |
```

## 非公共函数

### 余数

```sql
-- Mysql,Oracle,PostgreSql
select MOD(9,2) from dual;

-- Mysql,PostgreSql,SQLServer
select 9%2 from dual;
```

### 删除头尾空串

```sql
-- Mysql,Oracle,PostgreSql 
select TRIM(' Hello world ') from dual;

-- SQLServer 
select LTRIM(RTRIM(' Hello world ')) from dual;
```

### 向上取整

```sql
-- Mysql,Oracle,PostgreSQL CEIL 向上取整
SELECT
    CEIL(2.7) AS a,
    CEIL(-2.7) AS b
FROM dual;

-- SQLServer CEILING 向上取整
SELECT CEILING(2.7),CEILING(-2.7) FROM dual;
```

### 左截取

```sql
-- Mysql,SQLServer
select LEFT('Hello world', 5) from dual;

-- Oracle
select SUBSTR('Hello world', 1, 5) from dual;

-- PostgreSql
select SUBSTRING('Hello world', 1, 5) from dual;
```

### 右截取

```sql
-- Mysql,SQLServer
select RIGHT('Hello world', 5) from dual;

-- Oracle
select SUBSTR('Hello world', -5) from dual;

-- PostgreSql
select SUBSTRING('Hello world', 1+length('Hello world')-5, 5) from dual;
```

### 字符串截取

```sql
-- Mysql,Oracle SUBSTR 允许您提取字符串的一部分 位置,长度
select SUBSTR('Hello world', 2, 3) from dual;

-- Mysql,PostgreSql,SQLServer
select SUBSTRING('Hello world', 2, 3) from dual;
```

### 字符串连接

```sql
-- Oracle,PostgreSQL || 串联字符串
select 'Hello' || ' World!' from dual;

-- Mysql CONCAT 字符串连接
select concat('Hello',' World!') from dual;

-- SQLServer + 连接
select 'Hello' + ' World!' from dual;
```

### 字符串长度

```sql
-- Mysql,Oracle,PostgreSql
select length('123') from dual;

-- SQLServer
select len('123') from dual;
```

### 日期提取

```sql
-- Mysql,Oracle,PostgreSQL 日期提取
SELECT
    EXTRACT(YEAR FROM current_timestamp) YEAR,
    EXTRACT(MONTH FROM current_timestamp) MONTH,
    EXTRACT(DAY FROM current_timestamp) DAY,
    EXTRACT(HOUR FROM current_timestamp) HOUR,
    EXTRACT(MINUTE FROM current_timestamp) MINUTE,
    EXTRACT(SECOND FROM current_timestamp) SECOND,
    EXTRACT(HOUR FROM current_timestamp) HOUR,
    EXTRACT(MINUTE FROM current_timestamp) MINUTE,
    EXTRACT(SECOND FROM current_timestamp) SECOND
from dual;

-- SQLServer DATEPART 返回代表指定日期的指定日期部分的整数
select datepart(year,getdate()), datepart(yyyy,getdate())
select datepart(quarter,getdate()), datepart(qq,getdate())
select datepart(month,getdate()), datepart(mm,getdate())
select datepart(dayofyear,getdate()), datepart(dy,getdate())
select datepart(day,getdate()), datepart(dd,getdate())
select datepart(week,getdate()), datepart(wk,getdate())
select datepart(weekday,getdate()), datepart(dw,getdate())
select datepart(Hour,getdate()), datepart(hh,getdate())
select datepart(minute,getdate()), datepart(mi,getdate())
select datepart(second,getdate()), datepart(ss,getdate())
select datepart(millisecond,getdate()), datepart(ms,getdate())
```

### 日期检索

```sql
-- MYSQL
SELECT
	YEAR(CURRENT_TIMESTAMP()),
	MONTH(CURRENT_TIMESTAMP()),
	DAY(CURRENT_TIMESTAMP()),
	HOUR(CURRENT_TIMESTAMP()),
	MINUTE(CURRENT_TIMESTAMP()),
	QUARTER(CURRENT_TIMESTAMP()),
	SECOND(CURRENT_TIMESTAMP())
FROM DUAL;

-- ORACLE
SELECT
	TO_CHAR(SYSDATE, 'YYYY'),
	TO_CHAR(SYSDATE, 'MM'),
	TO_CHAR(SYSDATE, 'DD'),
	TO_CHAR(SYSDATE, 'HH24'),
	TO_CHAR(SYSDATE ,'MI'),
	TO_CHAR(SYSDATE, 'MON'),
	TO_CHAR(SYSDATE ,'Q'),
	TO_CHAR(SYSDATE ,'SS')
FROM DUAL;

-- POSTGRESQL
SELECT 
	EXTRACT(YEAR FROM CURRENT_TIMESTAMP()),
	EXTRACT(MONTH FROM CURRENT_TIMESTAMP()),
	EXTRACT(DAY FROM CURRENT_TIMESTAMP()),
	EXTRACT(HOUR FROM CURRENT_TIMESTAMP()),
	EXTRACT(MINUTE FROM CURRENT_TIMESTAMP),	
	EXTRACT(QUARTER FROM CURRENT_TIMESTAMP()),
	EXTRACT(SECOND FROM CURRENT_TIMESTAMP())
FROM DUAL;

-- SQLSERVER
SELECT
	DATEPART(YEAR,GETDATE()),
	DATEPART(MONTH,GETDATE()),
	DATEPART(DAY,GETDATE()),
	DATEPART(HOUR,GETDATE()),
	DATEPART(MINUTE,GETDATE()),
	DATEPART(QUARTER,GETDATE()),
	DATEPART(SECOND,GETDATE())
FROM DUAL;
```

### 日期相加

```sql
-- Mysql
select current_date() + 7 from dual;
```

### 查找字符串位置

```sql
-- Mysql,Oracle INSTR 第一个字符位于位置。如果 s2 未出现在 s1 中，则返回 0
select INSTR('Hello world', 'll') from dual;

-- Mysql,PostgreSql POSITION 第一个字符位于位置。如果 s2 未出现在 s1 中，则返回 0
select POSITION('ll' in 'Hello world') from dual;

-- SQLServer PATINDEX 第一个字符位于位置。如果 s1 未出现在 s2 中，则返回 0。匹配不区分大小写
select PATINDEX('%'+'ll'+'%','Hello world') from dual;
```

### 除法取整

```sql
-- Mysql 除法取整
select 8 DIV 3 from dual;

-- Oracle,PostgreSql,SQLServer FLOOR 给出等于或小于 f 的整数
select floor(8/3) from dual;
```

### 返回不为NULL的值

```sql
-- Mysql
select ifnull('1',null), ifnull(null,'2') from dual;

-- Oracle
select nvl('1',null), nvl(null,'2') from dual;
```
