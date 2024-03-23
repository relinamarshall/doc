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

# 数据库管理篇

## 控制文件和日志文件

控制文件和日志文件是Oracle数据库中存储信息的重要文件。控制文件(Control File)主要用来存放数据库的名字、数据文件的位置等信息;日志文件(Log File)主要用来存放数据库中数据变化的操作。

### 控制文件与日志文件概述

控制文件和日志文件是数据库中两个主要的文件，没有控制文件数据库就无法启动，没有日志文件数据库的信息就无法完全恢复。

#### 什么是控制文件

控制文件是数据库中的一个二进制文件，它主要用来记录数据库的名字、数据库的数据文件存放的位置等信息。因此，有人也把控制文件比做数据库的心脏，那么心脏如果丢失或者损坏了，数据库就将不复存在。对于控制文件来说，保护是至关重要的。

每个数据库都存在控制文件，但是一个控制文件只属于一个数据库。这就像每个人都有身份证，但是一个身份证只属于一个人。控制文件在创建数据库时自动被创建，当数据库的信息发生改变时，控制文件也随之被改变，控制文件不能手动修改，只能由Oracle数据库本身自己来修改。控制文件在数据库启动和关闭时都要使用，如果没有控制文件，数据库将无法工作。那么，控制文件究竟是什么样的呢?使用下面的语句就可在数据字典中查看控制文件的信息:&#x20;

<pre class="language-sql"><code class="lang-sql"><strong>desc v$controlfile;
</strong></code></pre>

{% hint style="info" %}
desc命令需在SQLPlus中执行
{% endhint %}

```
 名称                                      是否为空? 类型
 ----------------------------------------- -------- -------
 STATUS                                    VARCHAR2(7)
 NAME                                      VARCHAR2(513)
 IS_RECOVERY_DEST_FILE                     VARCHAR2(3)
 BLOCK_SIZE                                NUMBER
 FILE_SIZE_BLKS                            NUMBER
```

{% hint style="info" %}
数据字典就是一组表和视图结构，存放有数据库所用的有关信息，对用户来说是一组只读的表。在数据字典前面加上V$，代表的是当前实例的动态视图。
{% endhint %}

#### 什么是日志文件

日志文件在Oralce数据库中分为**重做日志**文件和**归档日志**文件两种。其中，重做日志文件是Oracle数据库正常运行不可缺少的文件。重做日志文件(Redo Log File)主要记录了数据库操作的过程。在需要恢复数据库时，重做日志文件可以将日志从备份还原的数据库上再执行一遍，以达到数据库的最新状态。&#x20;

Oracle系统在运行时有归档模式和非归档模式两种模式。在非归档模式下，所有的日志文件都写在重做日志文件中，如果重做日志文件写满了，那么就把前面的日志文件覆盖了。例如,一共有a、b、c三个日志文件，在非归档模式下，如果a文件写满了，就开始写b文件，如果b文件也写满了，就开始写c文件，如果三个文件都写满了，就把a文件中的内容覆盖掉重新写入。在归档模式下，如果重做日志文件全部写满后,就把第一个重做日志文件写入归档日志文件中再把日志写到第一个重做日志文件中，使用归档日志的方式就可以方便以后做恢复操作。

```sql
desc v$logfile;
```

```
 名称                                      是否为空? 类型
 ----------------------------------------- -------- ----------
 GROUP#                                    NUMBER
 STATUS                                    VARCHAR2(7)
 TYPE                                      VARCHAR2(7)
 MEMBER                                    VARCHAR2(513)
 IS_RECOVERY_DEST_FILE                     VARCHAR2(3)
```

{% code lineNumbers="true" %}
```sql
desc v$database;

--name 数据库名字
--log_mode 日志模式
select name,log_mode from v$database;

--NAME               LOG_MODE
------------------ ------------------------
--ORCL               NOARCHIVELOG
```
{% endcode %}

{% hint style="info" %}
日志的状态NOARCHIVELOG指的是非归档模式，而ARCHIVELOG指的是归档模式。
{% endhint %}

### 初始控制文件

#### 控制文件的内容

控制文件的存放位置和状态可以从数据字典v$controlfile中查询

```sql
select name,status from v$controlfile;
```

从上面的查询结果中可以看出数据库的控制文件的扩展名是.ctl。在v$controlfile中status列一般都为空。每一个控制文件中都记录数据库的名称、创建数据库的时间、数据文件的名字及数据库存储的位置、重做日志文件的名字及位置、归档日志的信息、表空间信息、日志历史记录、备份信息、当前的日志序列号、最近检查点的信息等数据库使用信息。这些信息都是在操作数据库时自动写入到控制文件中的，而不是手动写入的。例如，在创建一个数据库时，控制文件中就会自动写入数据库的名称、数据库的创建时间等信息:在增加或者删除表空间时，表空间的信息也会自动写入控制文件中。&#x20;

控制文件的大小取决于创建数据库时所提供的参数信息，因此当添加数据库中的文件时，控制文件的大小保持不变。

> 创建数据库参数说明

| 参数名            | 说明                                                                                                              |
| -------------- | --------------------------------------------------------------------------------------------------------------- |
| MAXLOGFILES    | 指定数据库可建的日志文件组的最大值，Oracle利用这个值决定在控制文件中使用的空间大小，该空间用于存储日志文件名。其默认值、最大值和最小值的变化依赖于OS                                  |
| MAXLOGMEMBERS  | 指定日志文件组的最大的成员数。Oracle使用该值决定在控制文件中分配给日志文件名的空问大小。最小值为1，最大值和默认值是变化的，依于OS                                           |
| MAXLOGHISTORY  | 指定归档日志文件的最大数目，用于并行服务器选项的介质自动恢复。Oracle利用该值决定在控制文件中分配给归档日志文件名的空间大小。默认值为MAXINSTANCES值的倍数，是可变的，决定于OS。最大值受控制文件的大小所限制 |
| MAXDATAFILES   | 指定数据库可建立的最大数据文件数，其最小值为1，最大值和默认值决定于OS。实例可存取的数据文件的数目受初始参数DBFILES的限制                                               |
| MAXINSTANCES   | 指定可同时装配或打开该数据库的实例的最大数目。这个值优先于初始化参数INSTANCES、最小值为1，最大值和默认值依赖于OS                                                  |

#### 更新控制文件

当增加、重命名、删除一个数据文件时，Oracle服务器进程(Server Process)会立即更新控制文件以反映数据库结构的这种变化。每次在数据库的结构发生变化后，为了防止数据丢失都要备份控制文件。按照各进程分工不同分别把数据库的更改后信息写入到控制文件中:日志写入进程(LGWR)负责把当前日志序列号记录到控制文件中，校验点进程(CKPT)负责把校验点的信息记录到控制文件中，归档进程负责把归档日志的信息记录到控制文件中。 通常情况下，数据库管理员会使用镜像来管理控制文件，把每个控制文件分布到不同的物理磁盘，发生灾难时，即使其中一个控制文件损坏，数据也不会丢失，也不会使整个数据库陷于瘫痪。

{% hint style="info" %}
当打开数据库提示“文件比控制文件更新一旧的控制文件”，此时的问题就是数据文件里面的控制文件序列号比控制文件的高，那么这个问题的出现可能是更改控制文件的位置所造成的。补救的方法可以用数据库中的日志文件重新创建一下控制文件。
{% endhint %}

### 控制文件的多路复用

控制文件在数据库中的作用是不可比拟的。保护控制文件实际就是在保护数据库。Oracle的多路复用的特性就可以帮助数据库管理员保护好控制文件。多路复用的特性可以把控制文件的副本创建到不同的磁盘上，这样即使一个磁盘发生故障，Oracle仍然可以从其他磁盘上恢复，从而达到保护数据库的作用。

#### init.ora多路复用控制文件

可以在数据库初始化文件中修改控制文件存放的位置，init.ora就是数据库初始化文件，它是在创建数据库时自动创建的。要修改init.ora，首先要找到它的存放位置，这个文件存储在安装文件目录下的`oracle\admin\orcl\pfile`里面。在修改init.ora文件之前，通过复制把控制文件复制到不同的位置，然后再修改init.ora中的control\_files参数。

{% code title="" overflow="wrap" lineNumbers="true" %}
```
##############################################################################
# Copyright (c) 1991, 2001, 2002 by Oracle Corporation
##############################################################################
 
###########################################
# Shared Server
###########################################
dispatchers="(PROTOCOL=TCP) (SERVICE=orclXDB)"
 
###########################################
# Miscellaneous
###########################################
compatible=11.2.0.0.0
diagnostic_dest=F:\oracle
memory_target=4972347392
 
###########################################
# Security and Auditing
###########################################
audit_file_dest=F:\oracle\admin\orcl\adump
audit_trail=db
remote_login_passwordfile=EXCLUSIVE
 
###########################################
# Database Identification
###########################################
db_domain=""
db_name=orcl
 
###########################################
# File Configuration
###########################################
control_files=("F:\oracle\oradata\orcl\control01.ctl", "F:\oracle\flash_recovery_area\orcl\control02.ctl")
db_recovery_file_dest=F:\oracle\flash_recovery_area
db_recovery_file_dest_size=4102029312
 
###########################################
# Cursors and Library Cache
###########################################
open_cursors=300
 
###########################################
# System Managed Undo and Rollback Segments
###########################################
undo_tablespace=UNDOTBS1
 
###########################################
# Processes and Sessions
###########################################
processes=150
 
###########################################
# Cache and I/O
###########################################
db_block_size=8192
```
{% endcode %}

按照现有的control\_files形式进行修改，在修改时需要注意的是每一个控制文件之间是通过逗号分隔的，并且每一个控制文件都是用双引号括起来的。另外，在进行控制文件路径修改之前，最好把控制文件复制一份保存，以免数据库无法启动。

#### SPFILE多路服用控制文件

使用SPFILE方式实现多路复用控制文件的原理和修改init.ora初始化参数的文件中control\_files参数是一样的，只是修改方法不同。只需要在SQLPlus中写语句就可以完成control files参数的修改。

> 修改control\_files参数

<pre class="language-sql"><code class="lang-sql">ALTER system SET
<strong>    control_files='文件路径1','文件路径2','文件路径n'
</strong>    scope=spfile;
</code></pre>

> 关闭数据库

在数据库打开时,数据库中的任何文件都是无法操作的。关闭数据库命令如下:

```sql
shutdown immediate;
```

> 在DOS下复制文件到指定位置

在DOS窗口下使用复制命令在指定位置增加一个控制文件。复制命令语法如下:

```powershell
copy 旧文件, 新文件
```

> 启动数据库实例并验证

在文件复制完成后，使用startup重新启动数据库的实例。在数据库字典controlfile中重新查询现存的控制文件。

```sql
select name,status from v$controlfile;
```

{% hint style="info" %}
在Oracle中的控制文件个数不能少于2个;复制文件时一定要使用命令复制，否则数据库实例无法重新启动。
{% endhint %}

### 创建控制文件

控制文件在数据库中的作用前面已经讲过了，但是无论怎么保护都有可能出现控制文件丢 失和损坏的问题。本节将学习创建控制文件。控制文件在创建数据库时就自动创建了,并且在配置文件`init.ora`中已经记录了文件的路径。前面已经学习过控制文件记录的内容以及控制文件的重要性，控制文件一般情况下是不需要手工创建的。只有在两种情况下才考虑用手工的方式来创建控制文件，

* 一种是当控制文件全部损坏，无法恢复时
* 一种是需要永久地修改数据库的参数设置时

因为手工创建控制文件会出现一些无法预计的问题，只能说是一种补救办法。

> 手工创建控制文件

手工创建控制文件分为找原有的数据文件和重做日志的路径、关闭数据库、创建新的控制 文件、修改init.ora中controlfiles参数、验证控制文件5个步骤。

> 找原有的数据文件和重做日志的路径

数据文件和重做日志信息可以通过两种途径获取，一种方法是当控制文件没有损坏时，从控制文件中直接获取:一种方法是控制文件损坏了，从数据字典`v$datafile`中获取数据文件的信息，从数据字典`v$logfile`中获取日志文件的信息。前面的两种途径都是在数据库能够正常启动时使用的方法，如果数据库不能够正常启动，那么此时就需要根据系统提供的错误信息来查找原因。在创建新的控制文件并且使用它打开数据库之后，Oracle会对数据字典和控制文件的内容进行检查。如果发现数据字典包含了某个数据文件而控制文件中没有列出这个数据文件，Oracle数据库就会报错。数据库管理员就只能凭借这些信息来判断是否缺少必要的数据文件，一步一步查找直到找到真正的数据文件。

本例是在数据库能正常启动的情况下，通过datafile和logfile数据字典分别获得数据文件列表和日志文件列表。

<pre class="language-sql"><code class="lang-sql"><strong>--数据文件
</strong><strong>select name from v$datafile;
</strong></code></pre>

```sql
--日志文件
select member from v$logfile;
```

> 关闭数据库

在创建控制文件之前要关闭数据库，最好是正常关闭数据库。采用正常模式关闭，可以减少数据库重新启动过程中可能出现的问题。为保证数据库的安全，在关闭数据库后，应该把数据库的日志文件、数据文件、参数文件等备份到其他磁盘上。关闭数据库使用如下语句:&#x20;

```sql
shutdown immediate;
```

> 创建新的控制文件

在创建新的控制文件之前，最好把原来的控制文件备份到其他位置，这样创建的效果比较明显。此外，还要启动一个数据库的实例，这个数据库实例是安装数据库时自带的。

启动实例后就可以创建控制文件了，在创建控制文件时就需要用到日志文件和数据文件的列表。创建控制文件的语句如下:

{% code lineNumbers="true" %}
```sql
create controlfile
reuse database "数据库实例名" noresetlogs //是否重做日志或重命名数据库 noarchivelog//归档状态
maxlogfiles   //最大日志文件大小
maxlogmembers //日志文件组的成员数
maxinstances  //最大实例的个数
maxloghistory //最大历史日志文件个数
logfile       //日志文件
group1        "日志文件的路径1"   size   日志文件的大小，
...
Groupn        "日志文件的路径n'   size   日志文件的大小
datafile      //数据文件
'路径1'
...
'路径n'
Character set we8dec
```
{% endcode %}

在创建控制文件时如果不需要重做日志文件和重命名数据库，就使用noresetogs，否则就使用resetlogs;在创建控制文件时也可以设置归档状态，如果设置为noarchivelog，则指非归档状态，若设置为archivelog，则是归档状态。

> 修改init.ora中controlfiles参数

使用前面讲过的spfile方法，修改init.ora中的参数controlfiles。

> 验证控制文件

重启数据库后，查询v$controlfile数据字典，检查控制文件是否全部正确加载。如果数据库启动不了，可以重启数据库服务。

至此，控制文件创建成功。

### 日志文件的管理

#### 新建日志文件组

创建日志文件组是每一个数据库管理员必须要掌握的技能，日志文件组能够帮助数据库管理员管理日志文件。

{% code lineNumbers="true" %}
```sql
ALTER DATABASE [database_name]
ADD LOGFILE GROUP n
filename SIZE m

【语法说明】
database_name:要修改的数据库名，省略该项表示当前的数据库。
n:创建重组日志组的组号，组号在重做日志组中都是唯一的。
filename:日志文件组存储的位置。
m:日志文件组的大小，默认的大小是50MB。
```
{% endcode %}

#### 添加日志文件到组中

日志文件组创建完成后，一个日志文件组中可以包含多个日志文件，至少含有一个日志文件。

{% code lineNumbers="true" %}
```sql
ALTER DATABASE [database_name]
ADD LOGFILE MEMBER
filename TO GROUP n

【语法说明】
database_name:数据库的名称，默认是当前的数据库。
filename:日志文件的路径。
n:日志文件填人的组号。
```
{% endcode %}

#### 删除日志文件或组

> 删除日志文件组

{% code lineNumbers="true" %}
```sql
ALTER DATABASE [database_name]
DROP LOGFILE
GROUP n

n代表的是日志文件组的组号。
```
{% endcode %}

> 删除日志文件

{% code lineNumbers="true" %}
```sql
ALTER DATABASE [database_name]
DROP LOGFILE MEMBER
filename

filename是指日志文件的名字，包括路径。
```
{% endcode %}

#### 查询日志文件或组

> 查询日志文件组

{% code lineNumbers="true" %}
```sql
select * from v$log
```
{% endcode %}

> 查询日志文件

{% code lineNumbers="true" %}
```sql
select * from v$logfile;
```
{% endcode %}

## 表空间管理

表空间是使用Oracle数据库必须具备的知识，在Oracle中创建数据库的同时就需要指定数据库建立的表空间。

### 表空间概述

#### 相关概念

在Oracle中表空间和数据文件的概念经常是成对出现的，每一个数据文件只对应一个表空间，一个表空间可以存放多个数据文件。在创建表空间的同时必须创建数据文件，同理，如果要创建数据文件必须要指定表空间。

&#x20;一个Oracle数据库是由一个或多个表空间组成的，在表空间中可以存储数据文件。这些数据文件也不是任意格式的，也要按照Oracle运行的操作系统的物理结构。数据文件中存放的就是要存放在数据库中的数据。在表空间中的逻辑存储单位是段(segment)。

例如，我们为表创建一个索引，那么就会在这个段中又创建一个区，这个区就叫做区段(extent)，也叫数据护展，每一个区段只能存在于一个数据文件中。区段再进一步划分还有区块(block)。但是，一个文件在磁盘上存储一般都是不连续的，所以，在表空间中的段要由不同数据文件中的区段组成。块是Oracle数据库中最小的空间分配单位。Oracle中常见的块大小是2、4、8、16KB。

#### 默认表空间

在Oracle 11g数据库存在6个默认表空间。查看默认表空间的方法主要有两种方式，一种是通过企业管理器直接查看，另一种是在数据字典中查看。下面分别用这两种方式查看认表空间。

在Oracle 11g中默认的表空间有6个，分别是EXAMPLE、SYSAUX、SYSTEM、TEMP、UNDOTBSI、USERS。

* EXAMPLE表空间:用于安装Oracle11g数据库使用示例数据库。&#x20;
* SYSAUX表空间:作为EXAMPLE的辅助表空间。&#x20;
* SYSTEM表空间:用来存储SYS用户的表、视图以及存储过程等数据库对象。&#x20;
* TEMP表空间:用于存储SOL语句处理的表和索引的信息。&#x20;
* UNDOTBS1表空间:用于存储撤销信息。&#x20;
* USERS表空间:存储数据库用户创建的数据库对象。

> 查看指定用户的默认表空间

```sql
SELECT TABLESPACE_NAME FROM DBA_TABLESPACES;
```

> 查看指定用户的默认表空间

查看某个用户的默认表空间,可以通过DBA\_USERS数据字典进行查询。

```sql
SELECT DEFAULT_TABLESPACE,USERNAME FROM DBA_USERS 
WHERE USERNAME LIKE 'SYS%';
```

{% hint style="info" %}
还可以使用数据字典dba\_free\_space查询;还可以从dba\_data\_files数据字典中查看表空间中数据文件的信息。
{% endhint %}

### 管理表空间

#### 创建表空间

{% code lineNumbers="true" %}
```sql
CREATE TABLESPACE tablespace_name
DATAFILE filename SIZE size
[AUTOEXTEND [ON/OFF]] NEXT size
[MAXSIZE size]
[PERMANENT|TEMPORARY]
[EXTENT MANAGEMENT
    [DICTIONARY | LOCAL
        [AUTOALLOCATE | UNIFORM. [SIZE integer[K|M]]]
    ]
]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
TABLESPACE:指定要创建的表空间的名称。
DATAFILE:指定在表空间中存放数据文件的文件名，这里还要指出文件存放的路径。
SIZE:指定数据文件的大小。
AUTOEXTEND:指定数据文件的扩展方式，ON代表自动扩展，OFF代表非自动扩展。另外，如果要把数据文件指定为自动扩展，应该在NEXT后面指定具体的大小。
MAXSIZE:指定数据文件为自动扩展方式时的最大值。
PERMANENT|TEMPORARY:指定表空间的类型，PERMANENT是指永久表空间;TEMPORARY是指临时表空间。在创建表空间时默认都是永久表空间。
EXTENT MANAGEMENT DICTIONARY|LOCAL:指定表空间的管理方式DICTIONARY是指字典管理方式;LOCAL是指本地的管理方式。在创建表空间时默认的管理方式是本地的管理方式。
```
{% endcode %}

{% hint style="info" %}
使用本地表空间管理的方式可以减少数据字典表的争用现象，并且也不需要对空间进行回收。因此，Oracle推荐使用本地表空间管理的方式创建表空间。
{% endhint %}

{% code title="创建表空间" lineNumbers="true" %}
```sql
CREATE TABLESPACE TESTONE
DATAFILE 'TESTOND.DBF' SIZE 10M;
```
{% endcode %}

{% code title="创建自动扩展表空间" lineNumbers="true" %}
```sql
CREATE TABLESPACE TESTTWO
DATAFILE 'TESTTWO.DBF' SIZE 10M
AUTOEXTEND ON NEXT 128M
MAXSIZE 2048M;
```
{% endcode %}

#### 重命名表空间

```sql
ALTER TABLESPACE oldname RENAME TO newname;
```

{% hint style="info" %}
不是所有的表空间都可以重命名，SYSTEM和SYSAUX表空间就不能重命名;除此之外，当表空间处于OFFLINE状态时也不可以重命名。
{% endhint %}

#### 设置表空间读写状态

```sql
ALTER TABLESPACE tablespace READ {ONLY|WRITE};
```

{% hint style="info" %}
在把表空间更改成只读状态时，要把表空间设置成联机状态。
{% endhint %}

#### 设置表空间可用状态

表空间的可用状态是指表空间的联机和脱机状态，如果把表空间设置成联机状态，那么表空间就可以被用户操作，反之设置成脱机状态，表空间就是不可用的。

```sql
ALTER TABLESPACE tablespace {ONLINE|OFFLINE [NORMARL|TEMPORARY|IMMEDIATE]}
```

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
ONLINE:设置表空间为联机状态，即可用状态。
OFFLINE:设置表空间为脱机状态，即不可用状态。这里还包括3种方式NORMAL指的是正常状态，TEMPORARY指的是临时状态,IMMEDIATE指的是立即状态。
```
{% endcode %}

#### 建立大文件表空间

建立大文件的表空间与前面讲过的创建表空间有些类似，大文件表空间是在正常表空间不够用的情况下才需要创建的。

{% code lineNumbers="true" %}
```sql
CREATE BIGFILE TABLESPACE tablespace
DATAFILE filename SIZE size;
```
{% endcode %}

{% hint style="info" %}
大文件的表空间最大可以是128TB，因此，可以说大文件表空间是存储大数据文件同时也是扩展表空间存储的好方式。
{% endhint %}

#### 删除表空间

表空间的管理可以使用本地管理的方式，也可以使用数据字典的方式，在删除表空间时由于管理的方式不同，那么删除的速度也会受到影响。通过大量的试验可以发现，使用本地方式管理的表空间在删除时速度更快一些。因此，要删除表空间时可以考虑把表空间的管理方式修改成本地方式管理后再删除。

{% code lineNumbers="true" %}
```sql
DROP TABLESPACE tablespace_name
[INCLUDING CONTENTS] [CASCADE CONSTRAINTS]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
[INCLUDING CONTENTS]:如果在删除表空间时要把表空间中的数据文件也删除，可以在删除的表空间语句后面加上该语句。
[CASCADECONSTRAINTS]:如果在删除表空间时要把表空间中的完整性也删除，可以在删除的表空间语句后面加上该语句。
```
{% endcode %}

### 管理临时表空间

在Oracle 11g数据库中除了上面讲述的表空间外，还有临时表空间，临时表空间主要是用来保存临时的数据信息的。

#### 建立临时表空间

临时表空间一般是指在数据库中存储数据，当内存不够时写入的空间，这个空间并不像一般的表空间，当执行完对数据库的操作后，该空间的内容自动清空。临时表空间经常会在使用一些操作时使用，如连接没有索引的两个表，查询数据时都会用到。

{% code lineNumbers="true" %}
```sql
CREATE TEMPORARY TABLESPACE tablespace_name
TEMPFILE 'filename.dbf' SIZE size;
```
{% endcode %}

> 设置临时表空间为默认表空间

```sql
ALTER DATABASE DEFAULT TEMPORARY TABLESPACE tablespace_name;
```

#### 查询临时表空间

```sql
SELECT TABLESPACE_NAME FROM DBA_TEMP_FILES;
```

{% hint style="info" %}
如果要查询临时表空间中用户的信息也可以在数据字典DBA\_USERS中查看。查看DBA USERS数据字典时可以先使用DESCDBAUSERS语句得到字典中存在的字段信息，然后再使用SELECT语句把需要的信息查询出来。
{% endhint %}

#### 创建临时表空间组

临时表空间组是由多个临时表空间组成的，每一个临时表空间组至少要有一个临时表空间存在，并且临时表空间组的名称也不能和其他表空间重名。

临时表空间组的创建实际上就是为表空间设置一个组，所以创建临时表空间组的语法和创建临时表空间的语法类似。在创建临时表空间时可以有两种方法，一种是在创建临时表空间组的同时也创建临时表空间，另一种是在创建临时表空间组时把已经存在的临时表空间移动到该临时表空间组中。

> 创建临时表空间存入表空间组中

{% code lineNumbers="true" %}
```sql
CREATE TEMPORARY TABLESPACE tablespace_name
TEMPFILE filename SIZE size TABLESPACE GROUP group_name;
```
{% endcode %}

> 把原临时表空间移到新创建的临时表空间组中

```sql
ALTER TABLESPACE tablespace_name TABLESPACE GROUP group_name;
```

#### 查询临时表空间组

```sql
SELECT * FROM DBA_TABLESPACE_GROUPS;
```

#### 删除临时表空间组

如果删除数据库中多余的临时表空间组，不需要先把临时表空间组中的临时表空间移除，只要删除临时表空间组中的所有临时表空间同时就会把临时表空间组删除掉。

{% hint style="info" %}
临时表空间组删除后不能恢复，所以在执行删除操作时一定要慎重。另外，在删除临时表空间组后，临时表空间的文件并没有删除，如果要删除临时表空间组中的临时表空间，那么就要先把临时表空间组中的临时表空间移除。
{% endhint %}

```sql
DROP TABLESPACE tablespace_name INCLUDING CONTENTS AND DATAFILES;
```

{% hint style="info" %}
在删除临时表空间时，不能删除默认的临时表空间
{% endhint %}

### 数据文件管理

#### 移动数据文件

在创建表空间时数据文件就已经创建好了，如果想把当前表空间中的数据文件移动到其他表空间中，在Oracle 11g的企业管理器中是无法完成的。移动数据文件的基本步骤如下:

> 把要存放数据文件所用的表空间设置成脱机状态

```sql
ALTER TABLESPACE tablespace_name OFFLINE
```

> 可以手动把要移动的文件移动到其他的表空间中

> 更改数据文件的名称

```sql
ALTER TABLESPACE tablespace_name 
RENAME DATAFILE oldname TO newname;
```

> 把表空间设置成联机状态

```sql
ALTER TABLESPACE tablespace_name ONLINE;
```

#### 删除数据文件

在使用数据文件时经常会去除一些没有用的数据文件，但是删除数据文件也有前提条件。当数据文件处于以下3种情况时它是不能够被删除的:&#x20;

* 数据文件中存在数据;&#x20;
* 数据文件是表空间中唯一或第一个的数据文件;
* 数据文件或数据文件所在的表空间处于只读状态

```sql
ALTER TABLESPACE tablespace_name DROP DATAFILE 'filename';
```

## 数据库安全管理

### 用户管理

#### 什么是用户

Oracle的用户管理应该说是每个数据库管理员都会遇到的一个问题，对用户管理涉及的主要问题就是用户所赋予的权限，根据每个用户访问Oracle数据库的需求不同，分配给用户的权限也就不同。如果数据库管理员对Oracle数据库用户的权限分配得合理，那么就能够提高数据库的安全性，相反，如果对Oracle数据库的用户权限分配得不合理，那么就会给数据库造成很大的隐患。

在Oracle中用户登录数据库的方式主要有三种:第一种是一般的密码验证方式。第二种是外部验证方式，这种方式并没有把验证密码存放在Oracle数据库中，其验证的密码通常与数据库所在的操作系统的密码一致。第三种就是全局验证方式，这种验证方式也不常用，它也不是把密码存放在Oracle数据库中的。这三种验证方式中最常用的就是密码验证的方式，同时这种方式的安全性也更高一些。

#### 创建用户

在Oracle中创建用户必须拥有数据库管理员的权限才能创建，在创建用户时还需要注意的是创建的用户的密码必须是以字母开头的。

{% code lineNumbers="true" %}
```sql
CREATE USER username IDENTIFIED BY password
OR EXTERNALLY AS certificate_DN
OR GLOBALLY AS directory_DN
[DEFAULT TABLESPACE tablespace]
[TEMPORARY TABLESPACE tablespace|tablespace_group_name]
[QUOTA size|UNLIMITED ON tablespace]
[PROFILE profile]
[PASSWORD EXPIRE]
[ACCOUNT LOCK|UNLOCK]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
在创建用户时有三种验证方式，以口令作为验证方式时选择【IDENTIFIED BY password】选项即可，以外部作为验证方式时选择【EXTERNALLY AS certificate_DN】选项:以全局作为验证方式时选择【GLOBALLYAS directory_DN】选项即可。
DEFAULT TABLESPACE:设置默认表空间，如果省略了该语句，那么这个新创建的用户就存放在数据库的默认表空间中，如果在数据库没有设置默认表空间，那么创建的用户就存放在SYSTEM表空间中。
TEMPORARY TABLESPACE:设置临时表空间或临时表空间组，可以把临时表空间存放在临时表空间组中，如果省略了该语句，那么就会把临时的文件存放到当前数据库中默认的临时表空间中;如果没有默认的临时表空间，那么会把临时文件存放到SYSTEM的临时表空间中。
QUOTA:设置当前用户使用表空间的最大值，在创建用户时可以有多个QUOTA来设置用户在不同表空间中能够使用的表空间大小。如果设置成UNLIMITED，表示对表空间的使用没有限制。
PROFILE:设置当前用户使用的概要文件的名称，如果省略了该子句，那么用户就使用当前数据库中默认的概要文件。
PASSWORD EXPIRE:设置当前用户密码立即处于过期状态，用户如果想再登录数据库必须要更改密码。
ACCOUNT:设置用户的锁定状态，如果设置成LOCK，那么该用户不能访问数据库，如果设置成UNLOCK，那么用户则可以访问数据库。在Oracle11g中默认的用户状态都为锁定的状态。
```
{% endcode %}

{% hint style="info" %}
创建用户时不能设置用户在临时表空间上使用的范围。
{% endhint %}

#### 修改用户

{% code lineNumbers="true" %}
```sql
ALTER USER user IDENTIFIED
{ BY password [REPLACE old_password]
    | EXTERNALLY [AS 'certificate_DN']
    | GLOBALLY [AS 'directory_DN']
}
[DEFAULT TABLESPACE tablespace]
[TEMPORARY TABLESPACE {tablespace | tablespace_group_name}]
[QUOTA {size_clause|UNLIMITED}ON tablespace]
[PROFILE profile]
[PASSWORD EXPIRE]
[ACCOUNT {LOCK|UNLOCK}]
```
{% endcode %}

#### 删除用户

数据库管理员经常会去除一些废弃不用的用户，而删除用户的同时也要把该用户所使用的数据库对象一并删除掉。

```sql
DROP USER user CASCADE
```

{% hint style="info" %}
如果要删除的用户中没有任何数据库对象，那么就可以省略CASCADE关键字。
{% endhint %}

### 授权管理

#### 什么是权限

在Oracle数据库中，权限有系统权限和对象权限两类。系统权限主要是指SESSION权限、USER权限等，也就是说对数据库的系统级的操作都可以称为系统权限。对象权限主要是指表对象、序列、触发器等操作的权限。

#### 授权权限

授予权限的对象就是用户或者角色，授予权限的操作包括授予系统权限和授予对象权限。

> 授予系统权限

{% code lineNumbers="true" %}
```sql
GRANT system_privilege
| ALL PRIVILEGES TO {user IDENTIFIED BY password | role}
[WITH ADMIN OPTION]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
system_privilege:创建的系统权限名称。
ALL PRIVILEGES:可以设置除SELECT ANY DICTIONARY权限之外的所有系统权限。
(user IDENTIFIED BY password | role}:设置权限的对象，user IDENTIFIED BY password子句代表的是设置指定用户的权限;role代表的是设置角色的权限。
WITH ADMIN OPTION:设置该子句后，表示当前给予授权的用户还可以给其他用户进行系统权限的授子。
```
{% endcode %}

> 授予用户对象权限

{% code lineNumbers="true" %}
```sql
GRANT object_privilege|ALL
ON schema.object
TO user|role
[WITH ADMIN OPTION]
[WITH THE GRANT ANY OBJECT]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
object_privilege:对象权限的名称。
ALL:如果选择ALL，则代表授予用户所有的对象权限，这个权限在使用的时候一定要慎重。
schema.object:为用户授子的对象权限使用的对象。
user|role:user代表的是用户;role代表的是角色。
WITH ADMIN OPTION:设置该子句后，表示当前给予授权的用户还可以给其他用户进行系统授权。
WITH THE GRANT ANY OBJECT:设置该子句后，表示当前给予授权的用户还可以给予其他用户对象权限。
```
{% endcode %}

#### 撤销权限

撤销权限也叫收回权限，也就是删除用户的系统权限或者对象权限。

> 撤销系统权限

{% code lineNumbers="true" %}
```sql
REVOKE system_privilege
FROM user|role;
```
{% endcode %}

> 撤销对象权限

{% code lineNumbers="true" %}
```sql
REVOKE object_privilege|ALL
ON schema.object
FROM user|role
[CASCADE CONTRAINTS]
```
{% endcode %}

这里需要说明的就是`CASCADE CONTRAINTS`选项，它表示该用户授予其他用户的权限一并撤销。

{% hint style="info" %}
在撤销用户权限时，撤销系统权限与撤销对象权限是不同的。如果撤销用户的系统权限，那么该用户授予其他用户的系统权限仍然存在;而撤销了用户的对象权限后，用户授子其他用户的对象权限也同时被撤销了。
{% endhint %}

#### 查询用户权限

Oracle11g中的用户权限存放在数据库的数据字典中，用户的系统权限存放在数据字典DBA\_SYS PRIVS中，用户的对象权限存放在数据字典DBA\_TAB\_PRIVS中。

```sql
SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE = 'DEMO'
```

```sql
SELECT * FROM DBA_TAB_PRIVS WHERE GRANTEE = 'DEMO'
```

{% hint style="info" %}
除了可以在DBA\_SYS\_PRIVS和DBA\_TAB\_PRIVS数据字典中查询权限之外，还可以直接在数据字典USER\_SYS\_PRIVS中查询当前登录用户的系统权限;在数据字典ALL\_TAB\_PRIVS中查询当前登录用户的对象权限。
{% endhint %}

### 角色管理

#### 什么是角色

角色就是由一组权限组成，用户也是可以被授予权限的。那么角色与用户有什么区别呢?用户是数据库的使用者而角色是权限的授予对象，给用户授予角色，也可以理解成给用户授予一组权限。数据库中的角色可以授予多个用户也可以不授予用户，并且一个用户可以被授予多个角色。角色是在数据库中由数据库管理员定义的权限集合,方便对不同用户的权限授予。例如，如果在数据库中设置一个拥有能够查询数据库中表的权限的角色，那么凡是用户需要拥有查询数据库中表的权限时，都可以直接授予该角色。

#### 创建角色

{% code lineNumbers="true" %}
```sql
CREATE ROLE role
[NOT IDENTIDIED 
    | IDENTIFIED BY [password]
    | IDENTIFIED BY EXETERNALLY
    | IDENTIFIED BY GLOBALLY
]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
在创建角色时要注意验证的方式，可以选择的验证方式有四种:
第一种是NOT IDENTIDIED，不需要验证;
第二种是IDENTIFIED BY [password],这是口令验证的方式;
第三种是IDENTIFIED BY EXETERNALLY,这是外部验证的方式;
第四种是IDENTIFIED BY GLOBALLY,这是全局验证方式。
```
{% endcode %}

> 授予角色权限

授予角色权限与授予用户权限所使用的语法一样，只是授予对象不是用户而是角色。

{% code lineNumbers="true" %}
```sql
GRANT system_privilege
| ALL PRIVILEGES TO role
[WITH ADMIN OPTION]
```
{% endcode %}

在给角色授予权限时，数据库管理员必须拥有GRANT\_ANY\_PRIVIEGES权限才可以给角色赋予任何权限。

#### 设置角色

角色创建完成后并不能直接使用它，而是要把角色授予用户才能使角色生效。

```sql
GRANT role TO user;
```

由于一个用户可以同时拥有多个角色，所以也可以设置哪些角色生效哪些角色不生效。设置的生效与失效方法如下:

{% code lineNumbers="true" %}
```sql
--指定角色生效
SET ROLE role

--设置用户所有角色生效
SET ROLE ALL

--设置在EXCEPT后的角色不生效
SET ROLE ALL EXCEPT role

--设置用户所有角色不生效
SET ROLE NONE
```
{% endcode %}

#### 修改角色

{% code lineNumbers="true" %}
```sql
ALTER ROLE role
[NOT IDENTIDIED
    |IDENTIDIED BY [password]
    |IDENTIDIED BY EXETERNALLY
    |IDENTIDIED BY GLOBALLY
]
```
{% endcode %}

上面的语法只是修改角色本身，如果修改已经授予角色的权限或者角色，则要使用 GRANT或者REVOKE来完成。

#### 删除角色

```sql
DROP ROLE rolename;
```

#### 查询角色

```sql
SELECT * FROM DBA_ROLE_PRIVS WHERE GRANTEE = 'DEMO';
```

### 概要文件Profile

PROFILE是Oracle中的概要文件，在PROFILE中主要存放的就是数据库中的系统资源或者数据库使用限制的一些内容。

#### 什么是Profile

PROFILE就是Oracle中的概要文件，在Oracle系统中如果不创建概要文件，默认会在系统中使用默认的概要文件DEFAULT。如果创建用户时没有为用户设置概要文件，那么默认都会使用数据库中的默认概要文件。概要文件会给数据库管理员带来很大的方便，数据库管理员可以先对数据库中的用户分组，按照每一组的权限不同，建立不同的概要文件。需要说明的一点是，虽然概要文件可以用于用户，但是概要文件是不能在角色中使用的。

#### 创建Profile

{% code lineNumbers="true" %}
```sql
CREATE PROFILE profile
LIMIT
{resource_parameters|password_parameters}
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
resource_parameters:资源参数，这些参数主要有CPU_PER_SESSION，代表允许一个会话占用CPU的总量;CPU_PER_CALL，代表允许一个调用占用CPU的最大值，CONNECT_TIME代表允许一个持续的会话的最大值。这些资源参数还有很多，在这里就不一一介绍了。
password_parameters:口令参数，这里也列举几个比较常用的参数，PASSWORDLIFE_TIME，指的是多少天后口令失效，PASSWORD_REUSE_TIME，是指密码保留的时间，PASSWORD_GRACE_TIME，是指设置密码失效后锁定。
```
{% endcode %}

#### 修改Profile

在修改概要文件时也可以同时修改多个配置，没有修改的配置还保持原样。

{% code lineNumbers="true" %}
```sql
ALTER PROFILE profile
LIMIT
{resource_parameters|password_parameters}
```
{% endcode %}

#### 删除Profile



```sql
DROP PROFILE profile [CASCADE]
```

{% code title="语法说明" overflow="wrap" %}
```
CASCADE:如果要删除的概要文件已经被用户使用过，那么在删除概要文件时要加上该关键字，把用户所使用的概要文件也撤销，如果要删除的概要文件没有被用户使用过，那么就可以省略该关键字。
```
{% endcode %}

{% hint style="info" %}
在Oracle中默认的概要文件是不能删除的。
{% endhint %}

#### 查询Profile

```sql
SELECT * FROM DBA_PROFILES;
```

## 备份恢复

### 数据库备份与恢复

#### 什么是数据库备份

数据库备份就是将数据库的内容全部复制出来保存到计算机的另一个位置或者其他存储设备上。数据库备份也分很多种，主要有物理备份和逻辑备份。 物理备份是指通常所说的归档模式备份(又叫热备份)和非归档模式备份(冷备份)，归档模式备份是当数据库的模式设置成归档模式时对数据库进行的备份，而非归档模式备份是当数据库的模式设置成非归档模式时对数据库的备份。逻辑备份主要是指对数据库的导入和导出操作，在Oracle 10g之前使用IMP/EMP的方式进行导人和导出操作，从Oracle10g开始引人了数据泵技术，使用EXPDP/IMPDP的方式对数据进行导入和导出的操作。

#### 什么是数据库恢复

数据库恢复就是把从数据库中备份出来的数据重新还原给原来的数据库，数据库的恢复技术分为完全恢复和不完全恢复两种。完全恢复是指把数据库恢复到数据库失败时的数据库状态不完全恢复是指将数据库恢复到数据库失败前的某一时刻的数据库状态。数据库备份分物理备份和逻辑备份，数据库恢复也分物理恢复和逻辑恢复，物理恢复就是把从数据库中备份的文件重新复制到原来的数据库中;逻辑恢复就是把从数据库中导出的数据再导入原来的数据库。

### 物理备份和恢复数据库

#### 对数据库进行脱机备份

脱机备份称为冷备份。首先，管理员身份的用户使用shutdown命令关闭数据库的服务，之后复制需要的文件，包括把数据文件和控制文件等相关的内容复制到其他磁盘的路径上。如果数据库出现问题，那么就可以把从数据库中复制出来的相关内容再复制回原来的数据库目录中。

#### 对数据库进行联机备份

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

### 逻辑备份和恢复数据库

#### 逻辑导出数据

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

#### 逻辑导入数据

逻辑导人数据是逻辑导出数据的逆过程，导入数据可以使用与EMP对应的IMP工具，也可以使用与EMPDP对应的IMPDP工具。

> 使用IMP导入数据

```bash
imp username/password file=filename.dmp tables=table_name;
```

> 使用IMPDP导入数据

{% code overflow="wrap" lineNumbers="true" %}
```bash
impdp username/password directory=dir dumpfile=file.dmp tables=table_name
```
{% endcode %}

## RMAN工具

RMAN是Oracle数据库提供的一种恢复和备份数据库的工具，也是数据库管理员管理数据库常用的工具之一。

### RMAN概述

RMAN是Recovery Manager的缩写，为Oracle的恢复管理器，主要用于备份和恢复数据库这也是Oracle推荐使用的一款恢复备份工具。

#### RMAN的特点

RMAN既然是Oracle的恢复管理器，在Oracle官方网站上给出的RMAN的特点主要有以下4个:&#x20;

* 它可以备份数据库、表空间、数据文件、控制文件以及日志文件。&#x20;
* 压缩备份可以只备份发生变化的内容。&#x20;
* 集成了第三方的磁带媒介软件。&#x20;
* 可以在Oracle数据库的目录中存放备份信息。

#### 与RMAN有关的概念

使用RMAN进行备份和恢复操作时，不可避免地要提到下面的一些常用概念，主要有目标数据库、闪回区、介质管理等。

1. 目标数据库\
   当在使用RMAN进行备份时，就会提到目标数据库这个概念。目标数据库就是使用RMAN工具进行备份和还原的数据库。&#x20;
2. RMAN客户端\
   当使用RMAN工具进行数据备份时，使用的前提就是计算机要拥有RMAN客户端。一般情况下，这个客户端是不用单独安装的，只要安装Oracle系统就自动安装了RMAN客户端。通常安装的目录与Oracle数据库的目录一致。
3. 闪回区\
   闪回区(FlashRecoveryArea)是在磁盘上的一个区域，在这个区域中存放与数据库的备份和恢复相关的一些文件，使用闪回区能够方便用户备份和还原数据库。闪回区这个概念是在 Oracle 10g时引出的。
4. 介质管理\
   介质管理设备通常被称为SBT(SystemBackuptoTape)设备，也就是把数据库备份到磁带中。RMAN通过介质管理器将数据备份到磁带上，介质管理器通常由第三方软件商提供。它将数据块中的数据流从RMAN通道进程传递到对应的磁带上。
5. 恢复目录\
   恢复目录(RecoveryCatalog)是一个独立的数据库，用于存放目标数据库的备份，这个目标数据库可以是一个，也可以是多个。

### 使用恢复目录

恢复目录是使用RMAN工具进行备份时要使用的存储备份信息的数据库，这也是Oracle推荐使用的一种方式，这样存储要比直接把目标数据库中内容存放到控制文件中更节省空间。

#### 创建恢复目录

为了确保数据的安全，一般情况下都是把恢复目录数据库创建到另外一个Oracle服务器中。在创建恢复目录时还要考虑数据库的容量，这个容量的大小当然要取决于目标数据库容量的大小。一般情况下，在恢复目录数据库设置的恢复目录表空间可以设置为50MB左右。创建恢复目录分为连接恢复目录的数据库、创建恢复目录的用户、给恢复目录用户赋角色以及创建恢复目录4个步骤。

> 连接恢复目录的数据库

恢复目录的用户就是指在备份目标数据库时使用的用户，这个用户与其他用户不同，必须要赋予RECOVERYCATALOG OWNER的角色才可以

```sql
conn username/password@servicename
```

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```sql
username:登录恢复目录数据库的用户名。
password:登录恢复目录数据库的密码。
servicename:恢复目录数据库的服务名。
```
{% endcode %}

> 创建恢复目录的用户

由于使用恢复目录时需要一些权限，所以说最好单独为恢复目录创建一个用户。创建用户的语句如下:&#x20;

{% code lineNumbers="true" %}
```sql
CREATE USER username IDENTIFIED BY password
[DEFAULT TABLESPACE tablespace_name]
```
{% endcode %}

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```
username:新创建的恢复数据库的用户名。 
password:新创建的恢复数据的密码。 
[DEFAULT TABLESPACE tablespace_name]:可选项，是给当前用户设置一个默认表空间，如果不设置默认表空间，则默认的表空间是SYSTEM表空间。最好给恢复目录设置一个默认的表空间，这样便于管理备份的数据。
```
{% endcode %}

> 给恢复目录用户赋角色

只创建一个数据库的用户是不能实现RMAN备份与恢复工作的，还需要给该用户赋予权限和角色。恢复目录用户应该拥有数据库管理员的权限，并在此基础上还要拥有RECOVERY\_CATALOG\_OWNER的权限。在此假设恢复目录用户已经拥有了数据库管理员的权限，这里只给恢复目录用户赋予RECOVERY\_CATALOG\_OWNER的角色。具体的语句如下:

```sql
GRANT RECOVERY_CATALOG_OWNER TO RMANUSER;
```

> 创建恢复目录

使用恢复目录用户登录数据库后就可以创建恢复目录

```sql
connect catalog RMANUSER/RMAN@RM
```

连接到RM恢复目录数据库之后，可以使用下面的语句创建恢复目录:

```sql
create catalog
```

为了方便自行测试恢复目录的使用，可以直接在本机的数据库中创建一个新的表空间，并在数据库上新创建一个用户，赋予RECOVERY\_CATALOG\_OWNER的角色，创建好后直接使用该用户登录本机数据库，并在本机数据库上创建一个恢复目录。

{% hint style="info" %}
使用恢复目录可以在一处同时存放多个目标数据库的文件，方便数据库管理员的管理这就要求尽量把恢复目录数据库放置到与目标数据库不同的服务器上。另外，一个最重要的优点就是恢复目录可以长时间保存RMAN备份的源数据
{% endhint %}

#### 使用RMAN连接

> 使用目标数据库中的控制文件备份数据

当备份数据库时不使用恢复目录，而是使用目标数据库中的控制文件来存放数据，那么使用目标数据库中的控制文件代替恢复目录。具体登录语法如下:&#x20;

```bash
RMAN TARGET username/password nocatalog;

#TARGET:代表要连接的是目标数据库 
#nocatalog:代表不使用恢复目录。
```

{% hint style="info" %}
如果目标数据库不在本地，需要远程连接，那么可以在用户名和密码之后输入@目标数据库的服务名的方式连接目标数据库。例如，如果orcl是远程数据库的服务名，那么在本例中写成RMANTARGETsys/abc123@orcl即可
{% endhint %}

> 连接到恢复目录数据库

{% code overflow="wrap" lineNumbers="true" %}
```bash
rman target username/password@servicename catalog username/password

#catalog:指恢复目录，在catalog后面是恢复目录数据库的用户名和密码。
```
{% endcode %}

除了使用上面的这种方法连接数据库外，还可以使用CONNECT命令在RMAN>下连接恢复目录数据库。具体的命令如下:

```bash
#连接目标数据库
CONNECT TARGET username/password@servicename

#连接恢复目录数据库
CONNECT CATALG username/password@servicename
```

#### 在恢复目录中注册数据库

目录数据库中注册数据库。其语法如下: 在恢复目录中注册数据库是以备份目标数据库为前提的，可以使用REGISTER命令在恢复&#x20;

```bash
REGISTER database
```

### 通道分配

通道分配是在使用RMAN备份时必须要提到的概念，通道分配分为手动和自动两种方式。

#### 什么是通道分配

通道就是指数据库与某一个设备关联，这个设备就是指存储的介质--磁带或磁盘。目前存储介质使用最多的还是磁带。通道分配就是确定连接数据库备份的设置个数，每设置一个设备就代表RMAN会自动启动一个服务器会话，由此来完成数据库的备份与恢复的操作。通道分配有手动分配和自动分配两种，这两种方式使用的命令不同:在手动分配通道时要使用RUN命令实现，而自动分配则只需要使用CONFIGURE命令即可完成。

#### 手动通道分配

```
RUN
{
    ALLOCATE CHANNEL channel_name1 DEVICE TYPE type_name1
    ALLOCATE CHANNEL channel_name2 DEVICE TYPE type_name2
    ...
    BACKUP ...
}
```

{% code title="语法说明" overflow="wrap" lineNumbers="true" %}
```bash
channel_name:分配的通道名称，type_name是分配的设备类型，这个设备类型是磁带(sbt)和磁盘(disk)，并且可以手动分配多个通道，即可以含有多个ALLOCATECHANNEL语句。
BACKUP:备份数据库的关键字，可以在BACKUP后面写上要备份的表空间等信息。
```
{% endcode %}

数据库的状态是正在运行的，不能够备份数据库中的文件。如果要备份数据库中的文件，可以先关闭数据库，然后把数据库启动到MOUNT的状态。这样就可以完成数据库的备份操作。

{% hint style="info" %}
在使用RUN命令前一定要在RMAN命令状态下，并且确保已经连接到目标数据库
{% endhint %}

#### 自动通道分配

<pre class="language-bash" data-line-numbers><code class="lang-bash"><strong>#指定设备的类型以及通道的个数，type_name是类型的名称，n代表通道的个数。
</strong><strong>CONFIGURE DEVICE TYPE type_name PARALLELISM n
</strong>#指定默认设备的类型，如果使用的设备基本都是磁盘，那么可以把默认设备设置成磁盘。
CONFIGURE DEFAULT DEVICE TYPE TO type_name
</code></pre>

### 备份集

#### 什么是备份集

备份集是在备份数据库时的选项，一个备份集可以存储一个或多个文件的备份信息，所以说备份集经常用在需要同时备份多个数据文件的情况。每一个备份集是由多个备份片组成的，备份片是一个单独的文件，并且备份片的大小也是有限制的。如果没有限制备份片的大小，那么在备份集中只能存在一个备份片。

#### BACKUP的使用

{% code overflow="wrap" lineNumbers="true" %}
```bash
BACKUP [level] [backup type][option]

#level;备份的增量，可以是1、2、3、4或者FULL，FULL代表的是全备份。
#backup type:备份数据库中的对象类型，这里可以是database(数据库)、datafile(数据文件)、tablespace(表空间)、controlfilecopy(备份使用copy命令备份的数据文件)archivelog all(备份归档日志文件)等对象。
#option:一个可选项，包括channel(用于指定备份所使用的通道)、maxsetsize(定义备份集的最大值)等信息。
```
{% endcode %}

### 从备份中恢复

#### 使用RESTORE还原

{% code overflow="wrap" lineNumbers="true" %}
```bash
RESTORE database_object

#database_object:数据库对象，可以是DATABASE(数据库)、TABLESPACE(表空间)、DATAFILE(数据文件)、CONTROLFILE(控制文件)、ARCHIVELOG(归档日志)、SPFILE(参数文件)。其中，只能在MOUNT状态下使用的对象有DATABASECONTROLFILE、SPFILE，只能在OPEN状态下使用的对象是TABLESPACE。
```
{% endcode %}

{% hint style="info" %}
在还原数据库时，要保证数据库的状态是MOUNT
{% endhint %}

#### 使用RECOVER恢复

{% code overflow="wrap" lineNumbers="true" %}
```bash
RECOVER database_type

#database_object:可以是DATABASE(数据库)、TABLESPACE(表空间)、DATAFILE(数据文件)。其中，DATABASE只能在MOUNT状态下使用，TABLESPACE只能在OPEN状态下使用。
```
{% endcode %}
