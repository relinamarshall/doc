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

# 表空间管理

表空间是使用Oracle数据库必须具备的知识，在Oracle中创建数据库的同时就需要指定数据库建立的表空间。

## 表空间概述

### 相关概念

在Oracle中表空间和数据文件的概念经常是成对出现的，每一个数据文件只对应一个表空间，一个表空间可以存放多个数据文件。在创建表空间的同时必须创建数据文件，同理，如果要创建数据文件必须要指定表空间。

&#x20;一个Oracle数据库是由一个或多个表空间组成的，在表空间中可以存储数据文件。这些数据文件也不是任意格式的，也要按照Oracle运行的操作系统的物理结构。数据文件中存放的就是要存放在数据库中的数据。在表空间中的逻辑存储单位是段(segment)。

例如，我们为表创建一个索引，那么就会在这个段中又创建一个区，这个区就叫做区段(extent)，也叫数据护展，每一个区段只能存在于一个数据文件中。区段再进一步划分还有区块(block)。但是，一个文件在磁盘上存储一般都是不连续的，所以，在表空间中的段要由不同数据文件中的区段组成。块是Oracle数据库中最小的空间分配单位。Oracle中常见的块大小是2、4、8、16KB。

### 默认表空间

在Oracle 11g数据库存在6个默认表空间。查看默认表空间的方法主要有两种方式，一种是通过企业管理器直接查看，另一种是在数据字典中查看。下面分别用这两种方式查看认表空间。

在Oracle 11g中默认的表空间有6个，分别是EXAMPLE、SYSAUX、SYSTEM、TEMP、UNDOTBSI、USERS。

* **EXAMPLE表空间**:用于安装Oracle11g数据库使用示例数据库。&#x20;
* **SYSAUX表空间**:作为EXAMPLE的辅助表空间。&#x20;
* **SYSTEM表空间**:用来存储SYS用户的表、视图以及存储过程等数据库对象。&#x20;
* **TEMP表空间**:用于存储SOL语句处理的表和索引的信息。&#x20;
* **UNDOTBS1表空间**:用于存储撤销信息。&#x20;
* **USERS表空间**:存储数据库用户创建的数据库对象。

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

## 管理表空间

### 创建表空间

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

### 重命名表空间

```sql
ALTER TABLESPACE oldname RENAME TO newname;
```

{% hint style="info" %}
不是所有的表空间都可以重命名，SYSTEM和SYSAUX表空间就不能重命名;除此之外，当表空间处于OFFLINE状态时也不可以重命名。
{% endhint %}

### 设置表空间读写状态

```sql
ALTER TABLESPACE tablespace READ {ONLY|WRITE};
```

{% hint style="info" %}
在把表空间更改成只读状态时，要把表空间设置成联机状态。
{% endhint %}

### 设置表空间可用状态

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

### 建立大文件表空间

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

### 删除表空间

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
[CASCADE CONSTRAINTS]:如果在删除表空间时要把表空间中的完整性也删除，可以在删除的表空间语句后面加上该语句。
```
{% endcode %}

## 管理临时表空间

在Oracle 11g数据库中除了上面讲述的表空间外，还有临时表空间，临时表空间主要是用来保存临时的数据信息的。

### 建立临时表空间

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

### 查询临时表空间

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

### 查询临时表空间组

```sql
SELECT * FROM DBA_TABLESPACE_GROUPS;
```

### 删除临时表空间组

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

## 数据文件管理

### 移动数据文件

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

### 删除数据文件

在使用数据文件时经常会去除一些没有用的数据文件，但是删除数据文件也有前提条件。当数据文件处于以下3种情况时它是不能够被删除的:&#x20;

* 数据文件中存在数据;&#x20;
* 数据文件是表空间中唯一或第一个的数据文件;
* 数据文件或数据文件所在的表空间处于只读状态

```sql
ALTER TABLESPACE tablespace_name DROP DATAFILE 'filename';
```
