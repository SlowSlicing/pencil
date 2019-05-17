* 首先停掉 MySQL 服务

```
$ systemctl stop mysqld
```

* 在 MySQL 配置文件中增加 `skip-grant-tables` 配置

```
$ vim /etc/my.cnf
......
[mysqld]
skip-grant-tables
......
```

* 重启 MySQL

```
$ systemctl restart mysqld
```

* 登录 MySQL 修改密码

>　　这次登录无需密码

```
$ mysql -uroot

# MySQL 5.6
mysql> update mysql.user set authentication_string=password('123456') where and user='root';
# MySQL 5.7
mysql> update mysql.user set password=password('123456') where and user='root';

# 刷新
mysql> flush privileges;
```

* OK，用新密码重试

