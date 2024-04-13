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

# 安全管理

## 用户管理

### 什么是用户

Oracle的用户管理应该说是每个数据库管理员都会遇到的一个问题，对用户管理涉及的主要问题就是用户所赋予的权限，根据每个用户访问Oracle数据库的需求不同，分配给用户的权限也就不同。如果数据库管理员对Oracle数据库用户的权限分配得合理，那么就能够提高数据库的安全性，相反，如果对Oracle数据库的用户权限分配得不合理，那么就会给数据库造成很大的隐患。

在Oracle中用户登录数据库的方式主要有三种:第一种是一般的密码验证方式。第二种是外部验证方式，这种方式并没有把验证密码存放在Oracle数据库中，其验证的密码通常与数据库所在的操作系统的密码一致。第三种就是全局验证方式，这种验证方式也不常用，它也不是把密码存放在Oracle数据库中的。这三种验证方式中最常用的就是密码验证的方式，同时这种方式的安全性也更高一些。

### 创建用户

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

### 修改用户

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

### 删除用户

数据库管理员经常会去除一些废弃不用的用户，而删除用户的同时也要把该用户所使用的数据库对象一并删除掉。

```sql
DROP USER user CASCADE
```

{% hint style="info" %}
如果要删除的用户中没有任何数据库对象，那么就可以省略CASCADE关键字。
{% endhint %}

## 授权管理

### 什么是权限

在Oracle数据库中，权限有系统权限和对象权限两类。系统权限主要是指SESSION权限、USER权限等，也就是说对数据库的系统级的操作都可以称为系统权限。对象权限主要是指表对象、序列、触发器等操作的权限。

### 授权权限

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

### 撤销权限

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

### 查询用户权限

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

## 角色管理

### 什么是角色

角色就是由一组权限组成，用户也是可以被授予权限的。那么角色与用户有什么区别呢?用户是数据库的使用者而角色是权限的授予对象，给用户授予角色，也可以理解成给用户授予一组权限。数据库中的角色可以授予多个用户也可以不授予用户，并且一个用户可以被授予多个角色。角色是在数据库中由数据库管理员定义的权限集合,方便对不同用户的权限授予。例如，如果在数据库中设置一个拥有能够查询数据库中表的权限的角色，那么凡是用户需要拥有查询数据库中表的权限时，都可以直接授予该角色。

### 创建角色

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

### 设置角色

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

### 修改角色

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

### 删除角色

```sql
DROP ROLE rolename;
```

### 查询角色

```sql
SELECT * FROM DBA_ROLE_PRIVS WHERE GRANTEE = 'DEMO';
```

## 概要文件Profile

PROFILE是Oracle中的概要文件，在PROFILE中主要存放的就是数据库中的系统资源或者数据库使用限制的一些内容。

### 什么是Profile

PROFILE就是Oracle中的概要文件，在Oracle系统中如果不创建概要文件，默认会在系统中使用默认的概要文件DEFAULT。如果创建用户时没有为用户设置概要文件，那么默认都会使用数据库中的默认概要文件。概要文件会给数据库管理员带来很大的方便，数据库管理员可以先对数据库中的用户分组，按照每一组的权限不同，建立不同的概要文件。需要说明的一点是，虽然概要文件可以用于用户，但是概要文件是不能在角色中使用的。

### 创建Profile

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

### 修改Profile

在修改概要文件时也可以同时修改多个配置，没有修改的配置还保持原样。

{% code lineNumbers="true" %}
```sql
ALTER PROFILE profile
LIMIT
{resource_parameters|password_parameters}
```
{% endcode %}

### 删除Profile

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

### 查询Profile

```sql
SELECT * FROM DBA_PROFILES;
```
