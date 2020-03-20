# 查看Oracle数据库所有的用户及表空间等

查看所有用户：

```sql
select * from all_users;
```

查看表空间：

```sql
select tablespace_name from dba_tablespaces;
```

查看用户具有怎样的角色：

```sql
select * from dba_role_privs where grantee='用户名'；
```

查看某个角色包括哪些系统权限：

```sql
select * from dba_sys_privs where grantee='DBA'
```

查看oracle中所有的角色：

```sql
select * from dba_roles;
```

查看所有用户占用的表空间：

```sql
SELECT OWNER as "用户名", sum(BYTES) / 1024 / 1024 / 1024 as "所有表的大小(GB)"
  FROM DBA_SEGMENTS
 WHERE SEGMENT_NAME in (select t2.OBJECT_NAME
                          from dba_objects t2
                         where t2.OBJECT_TYPE = 'TABLE')
 group by OWNER order by 2 desc
```

查看当前用户的表：

```sql
select table_name from user_tables;
```







