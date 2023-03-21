mysql数据库新建用户，并赋予权限



新建用户：

```
CREATE USER 'testuser' IDENTIFIED BY '123456';
```

赋予权限：

```
grant select on *.* to 'testuser' identified by '123456';
```