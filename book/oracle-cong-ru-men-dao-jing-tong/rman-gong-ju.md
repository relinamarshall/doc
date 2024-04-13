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

# RMAN工具

RMAN是Oracle数据库提供的一种恢复和备份数据库的工具，也是数据库管理员管理数据库常用的工具之一。

## RMAN概述

RMAN是Recovery Manager的缩写，为Oracle的恢复管理器，主要用于备份和恢复数据库这也是Oracle推荐使用的一款恢复备份工具。

### RMAN的特点

RMAN既然是Oracle的恢复管理器，在Oracle官方网站上给出的RMAN的特点主要有以下4个:&#x20;

* 它可以备份数据库、表空间、数据文件、控制文件以及日志文件。&#x20;
* 压缩备份可以只备份发生变化的内容。&#x20;
* 集成了第三方的磁带媒介软件。&#x20;
* 可以在Oracle数据库的目录中存放备份信息。

### RMAN有关的概念

使用RMAN进行备份和恢复操作时，不可避免地要提到下面的一些常用概念，主要有目标数据库、闪回区、介质管理等。

1. **目标数据库**\
   当在使用RMAN进行备份时，就会提到目标数据库这个概念。目标数据库就是使用RMAN工具进行备份和还原的数据库。&#x20;
2. **RMAN客户端**\
   当使用RMAN工具进行数据备份时，使用的前提就是计算机要拥有RMAN客户端。一般情况下，这个客户端是不用单独安装的，只要安装Oracle系统就自动安装了RMAN客户端。通常安装的目录与Oracle数据库的目录一致。
3. **闪回区**\
   闪回区(FlashRecoveryArea)是在磁盘上的一个区域，在这个区域中存放与数据库的备份和恢复相关的一些文件，使用闪回区能够方便用户备份和还原数据库。闪回区这个概念是在 Oracle 10g时引出的。
4. **介质管理**\
   介质管理设备通常被称为SBT(SystemBackuptoTape)设备，也就是把数据库备份到磁带中。RMAN通过介质管理器将数据备份到磁带上，介质管理器通常由第三方软件商提供。它将数据块中的数据流从RMAN通道进程传递到对应的磁带上。
5. **恢复目录**\
   恢复目录(RecoveryCatalog)是一个独立的数据库，用于存放目标数据库的备份，这个目标数据库可以是一个，也可以是多个。

## 使用恢复目录

恢复目录是使用RMAN工具进行备份时要使用的存储备份信息的数据库，这也是Oracle推荐使用的一种方式，这样存储要比直接把目标数据库中内容存放到控制文件中更节省空间。

### 创建恢复目录

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

### 使用RMAN连接

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

### 在恢复目录中注册数据库

目录数据库中注册数据库。其语法如下: 在恢复目录中注册数据库是以备份目标数据库为前提的，可以使用REGISTER命令在恢复&#x20;

```bash
REGISTER database
```

## 通道分配

通道分配是在使用RMAN备份时必须要提到的概念，通道分配分为手动和自动两种方式。

### 什么是通道分配

通道就是指数据库与某一个设备关联，这个设备就是指存储的介质--磁带或磁盘。目前存储介质使用最多的还是磁带。通道分配就是确定连接数据库备份的设置个数，每设置一个设备就代表RMAN会自动启动一个服务器会话，由此来完成数据库的备份与恢复的操作。通道分配有手动分配和自动分配两种，这两种方式使用的命令不同:在手动分配通道时要使用RUN命令实现，而自动分配则只需要使用CONFIGURE命令即可完成。

### 手动通道分配

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

### 自动通道分配

<pre class="language-bash" data-line-numbers><code class="lang-bash"><strong>#指定设备的类型以及通道的个数，type_name是类型的名称，n代表通道的个数。
</strong><strong>CONFIGURE DEVICE TYPE type_name PARALLELISM n
</strong>#指定默认设备的类型，如果使用的设备基本都是磁盘，那么可以把默认设备设置成磁盘。
CONFIGURE DEFAULT DEVICE TYPE TO type_name
</code></pre>

## 备份集

### 什么是备份集

备份集是在备份数据库时的选项，一个备份集可以存储一个或多个文件的备份信息，所以说备份集经常用在需要同时备份多个数据文件的情况。每一个备份集是由多个备份片组成的，备份片是一个单独的文件，并且备份片的大小也是有限制的。如果没有限制备份片的大小，那么在备份集中只能存在一个备份片。

### BACKUP的使用

{% code overflow="wrap" lineNumbers="true" %}
```bash
BACKUP [level] [backup type][option]

#level;备份的增量，可以是1、2、3、4或者FULL，FULL代表的是全备份。
#backup type:备份数据库中的对象类型，这里可以是database(数据库)、datafile(数据文件)、tablespace(表空间)、controlfilecopy(备份使用copy命令备份的数据文件)archivelog all(备份归档日志文件)等对象。
#option:一个可选项，包括channel(用于指定备份所使用的通道)、maxsetsize(定义备份集的最大值)等信息。
```
{% endcode %}

## 从备份中恢复

### 使用RESTORE还原

{% code overflow="wrap" lineNumbers="true" %}
```bash
RESTORE database_object

#database_object:数据库对象，可以是DATABASE(数据库)、TABLESPACE(表空间)、DATAFILE(数据文件)、CONTROLFILE(控制文件)、ARCHIVELOG(归档日志)、SPFILE(参数文件)。其中，只能在MOUNT状态下使用的对象有DATABASECONTROLFILE、SPFILE，只能在OPEN状态下使用的对象是TABLESPACE。
```
{% endcode %}

{% hint style="info" %}
在还原数据库时，要保证数据库的状态是MOUNT
{% endhint %}

### 使用RECOVER恢复

{% code overflow="wrap" lineNumbers="true" %}
```bash
RECOVER database_type

#database_object:可以是DATABASE(数据库)、TABLESPACE(表空间)、DATAFILE(数据文件)。其中，DATABASE只能在MOUNT状态下使用，TABLESPACE只能在OPEN状态下使用。
```
{% endcode %}
