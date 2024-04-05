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

# 一、SQL语句

## 多表插入

{% code lineNumbers="true" %}
```sql
INSERT ALL
WHEN LOGIN_NAME = 'DEMO' THEN
INTO LOG_USER_DEMO
WHEN LOGIN_NAME = 'SYS' THEN
INTO LOG_USER_SYS
ELSE
INTO LOG_USER_OTHER
SELECT * FROM LOG_USER LU;

SELECT * FROM LOG_USER_DEMO;
SELECT * FROM LOG_USER_SYS;
SELECT * FROM LOG_USER_OTHER;
```
{% endcode %}

## 更新语句

{% code lineNumbers="true" %}
```sql
UPDATE (
    SELECT PRODUCTNAME,'3' NEWVALUE 
    FROM PRODUCTINFO 
    WHERE PRODUCTID=6
) SET PRODUCTNAME = NEWVALUE
```
{% endcode %}

{% code lineNumbers="true" %}
```sql
UPDATE PRODUCTINFO
SET (PRODUCTNAME,PRODUCTPRICE) = (SELECT '4' ,3000  FROM DUAL)
WHERE PRODUCTID = 6
```
{% endcode %}

## 删除语句

{% code lineNumbers="true" %}
```sql
DELETE FROM (SELECT 1 FROM PRODUCTINFO WHERE PRODUCTID = 6)
```
{% endcode %}

## MERGE语句

{% code lineNumbers="true" %}
```sql
MERGE INTO LOG_USER_OTHER LUO
USING LOG_USER_DEMO LUD
ON (LUO.LOGIN_NAME = LUD.LOGIN_NAME)
WHEN MATCHED THEN 
    UPDATE SET LOGIN_TIME = SYSDATE 
    WHERE LOGIN_NAME = LUO.LOGIN_NAME
WHEN NOT MATCHED THEN 
    INSERT (LOGIN_ID,LOGIN_NAME,LOGIN_TIME)
    VALUES(LUD.LOGIN_ID,LUD.LOGIN_NAME,LUD.LOGIN_TIME);
```
{% endcode %}

{% hint style="info" %}
只记录了一些少见写法，详细信息请查阅PDF版
{% endhint %}
