MySQL
MySQL,用户,授权
[TOC]

# 用户管理

## 创建用户

&emsp;&emsp;**命令：**

```
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

&emsp;&emsp;**说明：**

* **username：**你将创建的用户名
* **host：**指定该用户在哪个主机上可以登陆，如果是本地用户可用 localhost，**如果想让该用户可以从任意远程主机登陆，可以使用通配符`%`**
* **password：**该用户的登陆密码，密码可以为空（视`密码策略如何设置`而定，我这里不允许为空），如果为空则该用户可以不需要密码登陆服务器

&emsp;&emsp;**例：**

```
CREATE USER 'test1'@'localhost' IDENTIFIED BY 'Aa123456.';
CREATE USER 'test2'@'192.168.1.111' IDENTIFIED BY 'Aa123456.';
CREATE USER 'test3'@'%' IDENTIFIED BY 'Aa123456.';
CREATE USER 'test4'@'%' IDENTIFIED BY '';
CREATE USER 'test5'@'%';
```

## 删除用户

### 删除用户

```
DELETE FROM user WHERE User='test1' and Host='localhost';
# 刷新权限
flush privileges;
```

### 删除用户及权限

```
drop user test2@192.168.1.111;
# 刷新权限
flush privileges;
```

## 修改用户密码

### MySQL 5.7.X

```
update mysql.user set authentication_string=password('Aa784165456.') where User="test3" and Host="%";
flush privileges;
```

### MySQL 5.6.X

```
UPDATE user SET Password=PASSWORD('Aa784165456.') where USER='test3';
flush privileges;
```

# 授权

## 设置权限

&emsp;&emsp;**命令：**

```
GRANT privileges ON databasename.tablename TO 'username'@'host'
```

&emsp;&emsp;**说明：**

* **privileges：**用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL
* **databasename：**数据库名
* **tablename：**表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*

&emsp;&emsp;**例子：**

```
GRANT SELECT, INSERT ON test.user TO 'test1'@'localhost';
GRANT ALL ON *.* TO 'test2'@'192.168.1.111';
GRANT ALL ON maindataplus.* TO 'test3'@'%';
```

&emsp;&emsp;**注意：**

&emsp;&emsp;用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令：

```
GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

> 具体信息可以用命令`SHOW GRANTS FOR 'test3'@'%';` 查看。

## 撤销权限

&emsp;&emsp;**命令：**

```
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```

&emsp;&emsp;**说明：**

&emsp;&emsp;privilege, databasename, tablename：同授权部分

&emsp;&emsp;**例：**

```
REVOKE INSERT ON mysql.user FROM 'test3'@'%';
```

&emsp;&emsp;**注意：**

* GRANT，REVOKE 用户权限后，该用户只有重新连接 MySQL 数据库，权限才能生效。

## 常用授权

| 命令 | 说明 |
| --- | --- |
| `grant all PRIVILEGES on *.* to 'root'@'%' IDENTIFIED by '你的密码'` | 给 root 用户 所有权限 |
| `grant select on testdb.* to root@'%'` | 给 root 用户 testdb 库的查询权限 |
| `grant insert on testdb.* to root@'%'` | 给 root 用户 testdb 库的新增权限 |
| `grant update on testdb.* to root@'%'` | 给 root 用户 testdb 库的更新权限 |
| `grant delete on testdb.* to root@'%'` | 给 root 用户 testdb 库的删除权限 |
| `grant select, insert, update, delete on testdb.* to root@'%'` | 给 root 用户 testdb 库的查询、新增、更新、删除权限 |
| `grant create,alter,drop on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库创建、修改、删除 MySQL 数据表结构权限 |
| `grant references on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库外键权限 |
| `grant create temporary tables on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库临时表权限 |
| `grant index on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库索引权限 |
| `grant create view on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库创建视图权限 |
| `grant show view on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库查看视图原代码权限 |
| `grant create routine on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库创建存储过程、函数权限 |
| `grant alter routine on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库修改、删除存储过程、函数权限 |
| `grant execute on testdb.* to root@'192.168.1.%';` | 给 ip 为 `192.168.0.任意` 的root用户 testdb 库执行存储过程、函数权限 |


