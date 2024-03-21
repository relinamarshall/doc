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

# MYSQL常用语句

## 查看相关

```sql
-- 查看所有库
show databases;

-- 查看所有表 
show tables;

-- 查看建表语句
show create table blog.tb_article;

-- 查看表索引
show index from blog.tb_article;
show keys from blog.tb_article;

-- 查看配置参数
show variables like '%buffer%';
-- 查看端口
show global variables like 'port';
-- 查看版本
show global variables like 'version';

-- 查看当前正在使用的表
show open tables;

-- 查看引擎状态,包含最后一次死锁原因记录
show engine innodb status;

-- 查看当前进程
-- 进程对应的sql可能显示不全
show processlist; 
show full processlist;

-- 查看执行计划
explain select * from blog.tb_article;
desc select * from blog.tb_article;
```

## 系统信息库

```sql
-- 所有数据库的信息
select * from information_schema.schemata;
-- show databases的结果取之此表
show databases;

-- 数据库中的表的信息(包括视图)
select * from information_schema.tables;
-- show tables from schemaname的结果取之此表
show tables from bank;

-- 表中的列信息
select * from information_schema.columns;
-- show columns from schemaname.tablename的结果取之此表
show columns from bank.t_admin;

-- 表索引的信息
select * from information_schema.statistics;
-- show index from schemaname.tablename的结果取之此表
show index from bank.t_admin;

-- 用户权限的信息
select * from information_schema.user_privileges;

-- 方案（数据库）权限的信息
select * from information_schema.schema_privileges;

-- 表权限的信息
select * from information_schema.table_privileges;

-- 列权限的信息
select * from information_schema.column_privileges;

-- mysql实例可用字符集的信息
select * from information_schema.character_sets;
-- show character set结果集取之此表
show character set;

-- 各字符集的对照信息
select * from information_schema.collations;
-- show collation结果集取之此表;
show collation;

-- 可用于校对的字符集
-- show collation的前两个显示字段
select * from information_schema.collation_character_set_applicability;

-- 描述了存在约束的表,以及表的约束类型
select * from information_schema.table_constraints;

-- 描述了具有约束的键列
select * from information_schema.key_column_usage;

-- 提供了关于存储子程序(存储程序和函数)的信息
select * from information_schema.routines;

-- 数据库中的视图的信息 show views权限,否则无法查看视图信息
select * from information_schema.views;

-- 触发程序的信息 super权限才能查看该表
select * from information_schema.triggers;

-- 案例
select 
    c.table_schema as '库名' ,
    t.table_name as '表名' ,
    t.table_comment as '表注释' ,
    c.column_name as '列名' ,
    c.column_comment as '列注释' ,
    c.ordinal_position as '列的排列顺序' ,
    c.column_default as '默认值' ,
    c.is_nullable as '是否为空' ,
    c.data_type as '数据类型' ,
    c.character_maximum_length as '字符最大长度' ,
    c.numeric_precision as '数值精度(最大位数)' ,
    c.numeric_scale as '小数精度' ,
    c.column_type as 列类型,
    c.column_key 'KEY' ,
    c.extra as '额外说明'
from information_schema.tables t left join information_schema.columns c 
    on t.table_name = c.table_name and t.table_schema = c.table_schema
where t.table_schema = 'bank'
order by c.table_name, c.ordinal_position;
```

## 索引

```sql
-- 添加索引
ALTER TABLE table_name ADD INDEX index_name (column_list);

ALTER TABLE table_name ADD UNIQUE (column_list);

ALTER TABLE table_name ADD PRIMARY KEY (column_list);

CREATE INDEX index_name ON table_name (column_list);

CREATE UNIQUE INDEX index_name ON table_name (column_list);

CREATE TABLE `pack_group` (
  PRIMARY KEY (`id`),
  UNIQUE `idx_shop_beehive_packable_time` (`shop_id`,`beehive_id`,`packable_time`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='pack_group';

-- 删除索引
DROP INDEX index_name ON talbe_name;
ALTER TABLE table_name DROP INDEX index_name;
ALTER TABLE table_name DROP PRIMARY KEY;
```

## 变量

```sql
-- 第一种用法 这里要使用变量来保存数据，直接使用@num变量
set @num=1;
set @num:=1;
select @num;

-- 第二种用法 注意上面赋值符号，使用set时可以用“=”或“:=”，但是使用select时必须用“:=赋值”
select @num:=1; 
select @num:=nums from (select 2 as nums from dual) t;
select @num;

-- 第三种用法
select nums1,nums2 into @num1,@num2 from (select 1 as nums1,2 as nums2 from dual) t;
select @num1, @num2;
```

## 一行分割多行

<pre class="language-sql"><code class="lang-sql">-- 需要注意mysql.help_topic.help_topic_id的最大值为686,分割行最大不能超过这个数
select
<strong>    distinct(substring_index(substring_index(t.content,',',mht.help_topic_id+1), ',', -1)) as split_content
</strong>from mysql.help_topic mht,
    (select 'filed1,field2,field3' as content from dual) t
where mht.help_topic_id &#x3C;= (length(t.content) - length(replace(t.content, ',', '')) + 1 );
</code></pre>

## 多行合并一行

```sql
select group_concat(t.content separator ',') concant_content from (
    select 'field1' as content from dual
    union
    select 'field2' as content from dual
    union
    select 'field3' as content from dual
) t
```
