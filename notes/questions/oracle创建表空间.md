# oracle创建表空间

## 创建表空间

>  create tablespace 表空间名 datafile '文件路径.dbf' size 200m autoextend on;

**示例**

```sql
create tablespace ssyh datafile 'E:\oracleDB\SSYH.dbf' size 200m autoextend on;
```



## 创建用户

>  create user 用户名 identified by "密码" default tablespace 表名;

**示例**

```sql
create user SSYH_XTGLY identified by "a123456" default tablespace SSYH;
```

### 修改用户密码

> alter user 用户名 identified by 密码;

**示例**

```sql
alter user jgywz identified by jgywz123;
```

## 删除用户

>  drop user 用户名 cascade

示例

```sql
drop user SSYH_XTGLY cascade
```



## 授权

> grant dba, connect, resource to 用户名

**示例**

```sql
grant dba, connect, resource to SSYH_XTGLY;
```



## 导入 

> imp '用户名/密码' file=文件地址 fromuser=原用户名 touser=新用户名 ignore=y commit=yes

示例

```sh
imp "SSYH_XTGLY/a123456" file=E:\db\SSYH.dmp fromuser=SSYH_XTGLY touser=SSYH_XTGLY ignore=y commit=yes;
```



## 导出

> exp 用户名/密码@远程ip:端口/实例名 file=导出的文件地址.dmp log=日志文件.log

示例

```sh
exp SSYH_XTGLY/a123456@127.0.0.1:1521/ORCL file=E:\db\SSYH.dmp log=E:\db\SSYH.log
```