# PostgreSQL

## 常用指令

反斜杠指令

```
\l          查询所有数据库
\c DBNAME   进入数据库
\d          显示所有关系表
```

创建用户和数据库

```
create user USERNAME with password 'PASSWD';
create database DBNAME owner USERNAME;
```

将数据库的权限全部赋予某个用户

```
grant all on database DBNAME to USERNAME;
```

导入导出

```
pg_dump DBNAME > FILE.sql
psql -U USERNAME DBNAME < FILE.sql
```
