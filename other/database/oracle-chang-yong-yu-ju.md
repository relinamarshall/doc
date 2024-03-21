---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# ORACLE常用语句

## 常用函数

```sql
--字符串函数
select 
    ascii('a') as "返回字符的ASCII码",
    concat('hello',',world') as "连接字符串",
    instr('hello world','or') as "字符串查找",
    length('hello') as "返回长度",
    lower('hello') as "转小写",
    upper('hello') as "转大写",
    ltrim('=hello=','=') as "去除左边字符串",
    rtrim('=hello=','=') as "去除右边字符串",
    trim('='from'=hello=') as "去除两边字符串",
    replace('abcde','cd','aaa') as "字符串替換",
    substr('abcde',2,3) as "字符串截取,2开始位置,3长度" 
from dual;

--数值函数
select 
    abs(-3) as "x的绝对值",
    acos(1) as "x的反余弦",
    cos(1) as "x的余弦",
    ceil(5.4) as "大于或等于x的最小值",
    floor(5.8) as "小于或等于x的最大值",
    log(2,4) as "x为底y的对数",
    mod(8,3) as "x除以y的余数",
    power(2,3) as "x的y次幂",
    round(3.456,2) as "x在第y位四舍五入",
    sqrt(4) as "x的平方根",
    trunc(3.456,2) as "x在第y位截断"
from dual;

--日期函数
select 
    add_months(sysdate,1) as "添加月份",
    last_day(sysdate) as "当月最后一天",
    extract (year from sysdate) as "获取年",
    extract (month from sysdate) as "获取月",
    extract (day from sysdate) as "获取日",
    extract (hour from systimestamp) as "获取时",
    extract (minute from systimestamp) as "获取分",
    extract (second from systimestamp) as "获取秒"
from dual;

--单行函数
select 
    '"'|| '||' ||'"' as "||",
    NVL(null,'default') as "nvl",
    NVL2(null,'not null','null') as "nvl2" 
from dual;
```

## 建表相关

```sql
--创建表
create table demo_test(
  pk_id varchar2(32) not null,
  context varchar2(32) not null,
  primary key (pk_id)
);

--设置注释
comment on table demo_test is '测试创建数据表';
comment on column demo_test.pk_id is '主键id';
comment on column demo_test.context is '测试内容';

--新增字段
alter table demo.demo_test add (test_add varchar2(100) null);

--修改字段
alter table demo.demo_test modify (test_add varchar2(256) not null);

--删除字段
alter table demo_test drop column test_add;
```

## 日期相关

```sql
--日期
--yy      two digits      两位年                显示值:07
--yyy     three digits    三位年                显示值:007
--yyyy    four digits     四位年                显示值:2007
--mm      number          两位月                显示值:11
--mon     abbreviated     字符集表示            显示值:11月,若是英文版,显示nov
--month   spelled out     字符集表示            显示值:11月,若是英文版,显示november
--dd      number          当月第几天            显示值:02
--ddd     number          当年第几天            显示值:02
--dy      abbreviated     当周第几天简写         显示值:星期五,若是英文版,显示fri
--day     spelled out     当周第几天全写         显示值:星期五,若是英文版,显示friday 
--hh      two digits      12小时进制            显示值:01
--hh24    two digits      24小时进制            显示值:13
--mi      two digits      60进制               显示值:45
--ss      two digits      60进制               显示值:25
--Q       digit           季度                 显示值:4
--WW      digit           当年第几周            显示值:44
--W       digit           当月第几周            显示值:1

select
    to_date('2003-10-17 21:15:37','yyyy-mm-dd hh24:mi:ss') as today ,
    to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') as nowTime ,
    to_char(sysdate,'yyyy')  as nowYear ,
    to_char(sysdate,'mm')    as nowMonth ,
    to_char(sysdate,'dd')    as nowDay ,
    to_char(sysdate,'hh24')  as nowHour ,
    to_char(sysdate,'mi')    as nowMinute ,
    to_char(sysdate,'ss')    as nowSecond ,
    -- 无时分秒
    trunc(sysdate) as 返回当前日期 ,
    trunc(sysdate,'year') as 返回当前年开始 ,
    trunc(sysdate,'month') as 返回当前月开始 ,
    trunc(sysdate,'day')  as 返回当前星期开始 ,
    floor(sysdate - to_date('20000510','yyyymmdd')) as 两个日期相隔多少天,
    to_char(sysdate,'DDD') as 今年第几天,
    next_day(sysdate,6) as 下一个星期五
from dual;
```

## 表信息相关

```sql
--获取表
-- 当前用户拥有的表
select * from user_tables;
select * from dba_tables where owner=UPPER('demo');
--所有用户的表
select * from all_tables;
--包括系统表
select * from dba_tables;

--获取表字段
--当前用户拥有的表
select * from user_tab_columns where Table_Name='DEMO_TEST';
--所有用户的表
select * from all_tab_columns where Table_Name='DEMO_TEST';
--包括系统表
select * from dba_tab_columns where Table_Name='DEMO_TEST';

--获取注释
select * from user_tab_comments;
select * from user_col_comments;
```

## 一行分割多行

```sql
-- 一行分割多行
select regexp_substr(t.content,'[^,]+',1,level) as split_content
from dual
    ,(select 'filed1,field2,field3' as content from dual) t
connect by regexp_substr(t.content,'[^,]+',1,level) is not null;
```

## 多行合并一行

```sql
-- 多行合并一行
select listagg(t.content , ',') within group (order by t.content) as concant_content 
from (
    select 'field1' as content from dual
    union
    select 'field2' as content from dual
    union
    select 'field3' as content from dual
) t
```

## 表空间相关

{% code overflow="wrap" %}
```sql
--查询表空间的类型
SELECT tablespace_name, contents FROM dba_tablespaces;

--创建表空间
--CREATE TABLESPACE tablespace_name
--    DATAFILE 'path/to/datafile' SIZE file_size [ REUSE ]
--    [ EXTENT MANAGEMENT <LOCAL | DICTIONARY> ]
--    [ AUTOEXTEND ON NEXT auto_extend_size MAXSIZE max_size ]
--    [ DEFAULT STORAGE (storage_clause)]
--    [ LOGGING | NOLOGGING]
--    [ ONLINE | OFFLINE]
--    [ PERMANENT | TEMPORARY]
--    [ SEGMENT SPACE MANAGEMENT { MANUAL | AUTO } ];
--tablespace_name:创建的表空间名称，即新表空间的名字。
--DATAFILE: 物理文件路径及名称，这个文件将作为表空间的数据容器（可以有多个），用于指定数据文件的路径和文件名。
--SIZE: 指定新创建的数据文件的初始大小，默认单位为 M。
--REUSE: 若数据文件已存在，可使用 REUSE 参数来重复利用该数据文件，不做任何命名修改操作。
--EXTENT MANAGEMENT: 指定表空间的分配方式。DICTIONARY 表示使用数据字典管理， LOCAL 表示使用局部管理方式，可以继承 DEFAULT 数据库设置。
--AUTOEXTEND: 对于需要自动扩展的数据文件，启用自动扩展功能。
--NEXT: 指定自动扩展的下一个增量大小，默认单位为 M。
--MAXSIZE: 数据文件最大允许的大小限制，默认也是以 M 作为单位。
--DEFAULT STORAGE (storage_clause)：指定表空间的默认存储属性。
--LOGGING：设置并激活日志记录，将数据更改所导致的事务记录在 Oracle 的重做日志文件中。
--NOLOGGING：不启用日志记录，不写事务日志，提高性能。
--ONLINE：允许并发连接到表空间中的对象，在创建表空间时，将其设置为在线状态。
--OFFLINE ：禁止对表空间的所有活动，在创建表空间时，将其设置为离线状态。
--PERMANENT：创建永久表空间。
--TEMPORARY：创建临时表空间。
--SEGMENT SPACE MANAGEMENT: 指定表空间中的段管理方式，MANUAL 表示手动管理，AUTO 表示自动管理。
CREATE TABLESPACE demo LOGGING DATAFILE 'F:\oracle\oradata\demo\demo.dbf' SIZE 100M AUTOEXTEND ON NEXT 32M MAXSIZE unlimited;

--删除表空间
drop tablespace demo;

--移动表空间
alter table DEMO.DEMO_TEST move tablespace demo;
```
{% endcode %}

## 用户相关

```sql
--创建用户 demo 密码 Zhouwen0510
CREATE USER demo IDENTIFIED BY Zhouwen0510 DEFAULT TABLESPACE demo;

-- 删除用户
drop user demo;

--查看用户 用户名数据库大写
--OPEN 为开启状态. EXPIRE 为密码过时状态. LOCKEN 为锁定状态,有密码.
select USERNAME, USER_ID, ACCOUNT_STATUS, DEFAULT_TABLESPACE 
from dba_users WHERE USERNAME = upper('demo');

-- 手工设置过期
alter user demo password expire;

-- 解锁用户
alter user demo account unlock;

-- 锁定用户就是修改密码
alter user demo identified by Zhouwen0510;
```

## 权限相关

{% code overflow="wrap" %}
```sql
--创建的用户没有任何权限, 连登陆数据库的权限都没有.
--Oracle 为了兼容以前的版本, 提供了三种标准角色: connect、resource 和 dba
--1. connect (连接角色): 这种角色下只可以登录 Oracle, 不可用创建实体, 也不可用创建数据库结构, 即只能对其他人创建的表中的数据进行操作.
--2. resource(资源角色): 该角色可以创建实体, 但是不可以创建数据库结构.可以创建表、序列 (sequence)、运算符 (operator)、过程 (procedure)、触发器 (trigger)、索引 (index)、类型 (type) 和簇 (cluster).
--3. dba (数据库管理员权限): 该角色拥有系统最高权限, 只有 DBA 才可以创建数据库结构. 包括无限制的空间限额和给其他用户授予各种权限的能力. 
grant connect,resource,dba to demo;
grant create session to demo;

-- 撤销授权
revoke connect, resource ,dba from demo;
```
{% endcode %}
