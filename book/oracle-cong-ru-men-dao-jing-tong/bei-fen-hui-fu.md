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

# 备份恢复

## 什么是数据库备份

数据库备份就是将数据库的内容全部复制出来保存到计算机的另一个位置或者其他存储设备上。数据库备份也分很多种，主要有物理备份和逻辑备份。 物理备份是指通常所说的**归档模式备份(又叫热备份)**和**非归档模式备份(冷备份)**，归档模式备份是当数据库的模式设置成归档模式时对数据库进行的备份，而非归档模式备份是当数据库的模式设置成非归档模式时对数据库的备份。逻辑备份主要是指对数据库的导入和导出操作，在Oracle 10g之前使用IMP/EMP的方式进行导人和导出操作，从`Oracle10g`开始引人了数据泵技术，使用`EXPDP/IMPDP`的方式对数据进行导入和导出的操作。

## 什么是数据库恢复

数据库恢复就是把从数据库中备份出来的数据重新还原给原来的数据库，数据库的恢复技术分为完全恢复和不完全恢复两种。完全恢复是指把数据库恢复到数据库失败时的数据库状态不完全恢复是指将数据库恢复到数据库失败前的某一时刻的数据库状态。数据库备份分物理备份和逻辑备份，数据库恢复也分物理恢复和逻辑恢复，物理恢复就是把从数据库中备份的文件重新复制到原来的数据库中;逻辑恢复就是把从数据库中导出的数据再导入原来的数据库。

## 物理备份和恢复数据库

### 对数据库进行脱机备份

脱机备份称为冷备份。首先，管理员身份的用户使用shutdown命令关闭数据库的服务，之后复制需要的文件，包括把数据文件和控制文件等相关的内容复制到其他磁盘的路径上。如果数据库出现问题，那么就可以把从数据库中复制出来的相关内容再复制回原来的数据库目录中。

### 对数据库进行联机备份

联机备份称为热备份，是在数据库的归档模式下进行的备份。查看数据库中日志的命令如下:

> 查询本机数据库的日志状态

需在SQLPLUS中查询

```sql
--sqlplust dba登录
sqlplus sys/password as sysdba

--查询
archive log list;

--输出日志
数据库日志模式             非存档模式
自动存档             禁用
存档终点            USE_DB_RECOVERY_FILE_DEST
最早的联机日志序列     97
当前日志序列           99
```

{% code lineNumbers="true" %}
```sql
--修改系统日志方式为归档模式
alter system set log_archive_start=true scope=spfile;
--关闭数据库
shutdown immediate;
--启动mount实例,但不启动数据库
startup mount;
--更新数据库为归档日志模式
alter database archivelog;

--查询
archive log list;

--输出日志
数据库日志模式            存档模式
自动存档             启用
存档终点            USE_DB_RECOVERY_FILE_DEST
最早的联机日志序列     97
下一个存档日志序列   99
当前日志序列           99
```
{% endcode %}

> 备份表空间TEST

```sql
alter database open;
```

> 开始备份表空间

```sql
alter tablespace DEMO begin backup;
```

> 打开数据库中的oradata文件夹(一般数据库对象都存放在该文件夹中)，把文件复制到磁盘中的另一个文件夹或其他磁盘上。&#x20;
>
> 结束表空间的备份 在完成前面的操作之后，执行下面的命令结束备份:

```sql
alter tablespace DEMO end backup;
```

至此，已经把表空间备份到其他位置了。

接下来是恢复表空间中的数据文件

> 恢复表空间中的数据文件

```sql
alter system archive log current;
```

> 切换日志文件

由于在一个数据库中一般有3个日志文件，所以需要使用3次下面的语句来切换日志文件:

```sql
alter system switch logfile;
```

> 关闭数据库服务

这里为了模拟TEST表空间中的数据文件丢失，先把数据库关闭，然后删除TEST表空间中的数据文件TESTONE.DBF。关闭数据库的命令如下:

```sql
shutdown immediate;
```

删除数据文件并重新启动数据库

删除数据文件首先要找到存放数据文件的位置，在默认情况下数据文件会存放在数据库的ORADATA文件夹中，也有在创建表空间添加数据文件时指定的目录。如果不清楚数据文件存放的位置，可以直接在DOS窗口下的v$datafile数据字典中査看表空间中数据文件的位置。找到数据文件后直接将其删除即可，然后启动数据库。启动数据库的命令如下:

```sql
startup;
```

在使用startup命令启动数据库后，会出现错误提示界面。可以看到提示缺少了编号为5的数据文件，也可以通过查看数据字典v$recover\_file确认缺少的数据文件。

```sql
select * from v$recover_file;

FILE#|ONLINE|ONLINE_STATUS|ERROR         |CHANGE#|TIME|
-----+------+-------------+--------------+-------+----+
    5|ONLINE|ONLINE       |FILE NOT FOUND|      0|    |
```

> 将数据文件设置成脱机状态并删除

在恢复数据文件之前需要先把数据文件设置成脱机状态(offline状态)，并且删除该数据文件。具体命令如下:

```sql
alter database datafile 5 offline drop;
```

> 把数据库的状态设置成OPEN

在完成上述操作后，就可以为恢复数据库做好准备，把数据库的状态设置成OPEN。具体命令如下:&#x20;

```sql
alter database open;
```

> 恢复数据文件

```sql
recover datafile 5;
```

这里的编号5仍然是之前查看到的数据文件的编号，这也是需要注意的问题。在恢复时数据文件的编号要一致。

> 设置数据文件为联机状态

```sql
alter database datafile 5 online;
```

至此，就完成了数据文件的恢复操作。为了验证数据文件是否恢复成功，可以重新启动数据库看一下效果。&#x20;

{% hint style="info" %}
在恢复数据库中的数据文件时，把数据库文件设置成脱机状态后，就需要把之前备份好的数据文件复制到原来的数据文件存放的位置
{% endhint %}

## 逻辑备份和恢复数据库

### 逻辑导出数据

导出数据可以使用EXP工具完成，也可以使用在Oracle10g以后出现的EXPDP工具完成。 下面分别使用这两种方式对数据库进行导出备份。

> **使用EXP工具备份**

EXP工具可以将数据库中的对象有选择性地备份出来，可以使用EXP工具导出的数据库对象有表、方案、表空间以及数据库。

> 导出表

{% code lineNumbers="true" %}
```powershell
EXP username/password

输入数组提取缓冲区大小: 4096 > 
导出文件: EXPDAT.DMP >
(1)E(完整的数据库), (2)U(用户) 或 (3)T(表): (2)U > 
导出权限 (yes/no): yes > 
导出表数据 (yes/no): yes > 
压缩区 (yes/no): yes > 
```
{% endcode %}

这里的username、password就是登录数据库的用户名和密码，但是这里的用户不能是SYS

如果要导出其他方案中的表，则需要在表名的前面加上方案名。假设要导出SYSTEM方案下的TEST表，那么就要写成SYSTEM.TEST。如果要导出多个表，可以在表名之间加上“,”间隔开。

{% hint style="info" %}
在导出语句后面一定不能加上分号，如果加上分号，那么系统就会认为导出的表是表名加上分号的
{% endhint %}

> 导出表空间

导出表空间与导出表不同，导出表空间的用户必须是数据库的管理员角色。导出表空间的命令如下:&#x20;

```sh
EXP username/password FILE="filename.dmp" TABLESPACES="tablespaces_name"
```

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```
username/password:登录数据库使用的用户名和密码，一定是具有数据库管理员权限的用户。 
filename.dmp:存放备份的表空间的数据文件。 
tablespaces_name:要备份的表空间名称。
```
{% endcode %}

> **使用EXPDP导出数据**

EXPDP是Oracle10g开始引入的数据泵技术，数据泵技术是在数据库之间或者在数据库与操作系统之间传输数据的工具。EXPDP是数据泵导出的工具，它可以把数据库中的对象导出到操作系统中。使用EXPDP工具与EXP不同的是，在使用EXPDP时要先创建目录对象，通过这个对象就可以找到要备份数据的数据库服务器，并且使用EXPDP工具备份出来的数据必须存放在目录对象对应的操作系统的目录中。

> 创建目录对象

```sql
CREATE DIRECTORY directory_name AS 'file_name'
```

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```
directory_name:创建的目录名称
file_name:存放数据的文件名称
```
{% endcode %}

> 给使用目录的用户赋权限

新创建的目录对象不是任何用户都可以使用的，只有拥有该目录使用权的用户才能使用所以要为使用该目录的用户赋一个权限。假设备份数据库的用户是DEMO，那么赋权限的i句如下:&#x20;

```sql
GRANT READ,WRITE ON DIRECTORY directory_name TO DEMO
```

这里，directory\_name就是创建的目录名称。

> 导出表

{% code overflow="wrap" lineNumbers="true" %}
```bash
EXPDP username/password DIRECTORY=directory_name DUMPFILE=file_name TABLES=table_name;
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```
directory_name:存放导出数据的目录名称。
file_name:导出数据存放的文件名。
table_name:准备导出的表名，对于多个表可以用逗号隔开。
```
{% endcode %}

### 逻辑导入数据

逻辑导人数据是逻辑导出数据的逆过程，导入数据可以使用与EMP对应的IMP工具，也可以使用与EMPDP对应的IMPDP工具。

> 使用IMP导入数据

```bash
imp username/password file=filename.dmp tables=table_name;
```

> 使用IMPDP导入数据

{% code overflow="wrap" %}
```bash
impdp username/password directory=dir dumpfile=file.dmp tables=table_name
```
{% endcode %}
