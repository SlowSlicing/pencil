MySQL
CentOS,MySQL
#### 卸载自带MariaDB
```
[root@root tmp]# rpm -qa | grep mariadb
mariadb-libs-5.5.41-2.el7_0.x86_64
[root@root tmp]# rpm -e --nodeps mariadb-libs-5.5.41-2.el7_0.x86_64
```

#### 下载安装包
![下载MySQL RPM包](https://upload-images.jianshu.io/upload_images/2860514-6be22e09fa1ad8c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 安装
```
// 下载
[root@root tmp]# wget https://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar
--2018-05-03 09:53:46--  https://dev.mysql.com/get/Downloads/MySQL-5.6/MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar
正在解析主机 dev.mysql.com (dev.mysql.com)... 137.254.60.11
正在连接 dev.mysql.com (dev.mysql.com)|137.254.60.11|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 302 Found
位置：https://cdn.mysql.com//Downloads/MySQL-5.6/MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar [跟随至新的 URL]
--2018-05-03 09:53:47--  https://cdn.mysql.com//Downloads/MySQL-5.6/MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar
正在解析主机 cdn.mysql.com (cdn.mysql.com)... 23.11.255.146
正在连接 cdn.mysql.com (cdn.mysql.com)|23.11.255.146|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：243456000 (232M) [application/x-tar]
正在保存至: “MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar”

100%[===========================================================================================================================================================>] 243,456,000 6.10MB/s 用时 41s

2018-05-03 09:54:29 (5.68 MB/s) - 已保存 “MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar” [243456000/243456000])

// 解压
[root@root tmp]# tar -xvf MySQL-5.6.40-1.el7.x86_64.rpm-bundle.tar
MySQL-shared-5.6.40-1.el7.x86_64.rpm
MySQL-devel-5.6.40-1.el7.x86_64.rpm
MySQL-server-5.6.40-1.el7.x86_64.rpm
MySQL-shared-compat-5.6.40-1.el7.x86_64.rpm
MySQL-test-5.6.40-1.el7.x86_64.rpm
MySQL-client-5.6.40-1.el7.x86_64.rpm
MySQL-embedded-5.6.40-1.el7.x86_64.rpm
// 安装，跳过此步操作，请先进行下一步操作，会出错
[root@root tmp]# rpm -ivh MySQL-server-5.6.40-1.el7.x86_64.rpm
```
**出现错误：**
```
警告：MySQL-server-5.6.40-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:MySQL-server-5.6.40-1.el7        ################################# [100%]
警告：用户mysql 不存在 - 使用root
警告：群组mysql 不存在 - 使用root
警告：用户mysql 不存在 - 使用root
警告：群组mysql 不存在 - 使用root
FATAL ERROR: please install the following Perl modules before executing /usr/bin/mysql_install_db:
Data::Dumper
```
**解决方案：**
```
// 安装组件
[root@root tmp]# yum install -y perl-Module-Install.noarch
// 卸载原有的，重新安装
[root@root tmp]# rpm -qa | grep -i mysql
MySQL-server-5.6.40-1.el7.x86_64
[root@root tmp]# rpm -e MySQL-server-5.6.40-1.el7.x86_64
警告：MySQL-server-5.6.40-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:MySQL-server-5.6.40-1.el7        ################################# [100%]
警告：用户mysql 不存在 - 使用root
警告：群组mysql 不存在 - 使用root
警告：用户mysql 不存在 - 使用root
警告：群组mysql 不存在 - 使用root
2018-05-03 10:26:04 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-05-03 10:26:04 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2018-05-03 10:26:04 0 [Note] /usr/sbin/mysqld (mysqld 5.6.40) starting as process 2589 ...
2018-05-03 10:26:04 2589 [Note] InnoDB: Using atomics to ref count buffer pool pages
2018-05-03 10:26:04 2589 [Note] InnoDB: The InnoDB memory heap is disabled
2018-05-03 10:26:04 2589 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2018-05-03 10:26:04 2589 [Note] InnoDB: Memory barrier is not used
2018-05-03 10:26:04 2589 [Note] InnoDB: Compressed tables use zlib 1.2.3
2018-05-03 10:26:04 2589 [Note] InnoDB: Using Linux native AIO
2018-05-03 10:26:04 2589 [Note] InnoDB: Using CPU crc32 instructions
2018-05-03 10:26:04 2589 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2018-05-03 10:26:04 2589 [Note] InnoDB: Completed initialization of buffer pool
2018-05-03 10:26:04 2589 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2018-05-03 10:26:04 2589 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2018-05-03 10:26:04 2589 [Note] InnoDB: Database physically writes the file full: wait...
2018-05-03 10:26:04 2589 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2018-05-03 10:26:05 2589 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2018-05-03 10:26:06 2589 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2018-05-03 10:26:06 2589 [Warning] InnoDB: New log files created, LSN=45781
2018-05-03 10:26:06 2589 [Note] InnoDB: Doublewrite buffer not found: creating new
2018-05-03 10:26:06 2589 [Note] InnoDB: Doublewrite buffer created
2018-05-03 10:26:06 2589 [Note] InnoDB: 128 rollback segment(s) are active.
2018-05-03 10:26:06 2589 [Warning] InnoDB: Creating foreign key constraint system tables.
2018-05-03 10:26:06 2589 [Note] InnoDB: Foreign key constraint system tables created
2018-05-03 10:26:06 2589 [Note] InnoDB: Creating tablespace and datafile system tables.
2018-05-03 10:26:06 2589 [Note] InnoDB: Tablespace and datafile system tables created.
2018-05-03 10:26:06 2589 [Note] InnoDB: Waiting for purge to start
2018-05-03 10:26:06 2589 [Note] InnoDB: 5.6.40 started; log sequence number 0
A random root password has been set. You will find it in '/root/.mysql_secret'.
2018-05-03 10:26:06 2589 [Note] Binlog end
2018-05-03 10:26:06 2589 [Note] InnoDB: FTS optimize thread exiting.
2018-05-03 10:26:06 2589 [Note] InnoDB: Starting shutdown...
2018-05-03 10:26:08 2589 [Note] InnoDB: Shutdown completed; log sequence number 1625977


2018-05-03 10:26:08 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2018-05-03 10:26:08 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2018-05-03 10:26:08 0 [Note] /usr/sbin/mysqld (mysqld 5.6.40) starting as process 2611 ...
2018-05-03 10:26:08 2611 [Note] InnoDB: Using atomics to ref count buffer pool pages
2018-05-03 10:26:08 2611 [Note] InnoDB: The InnoDB memory heap is disabled
2018-05-03 10:26:08 2611 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2018-05-03 10:26:08 2611 [Note] InnoDB: Memory barrier is not used
2018-05-03 10:26:08 2611 [Note] InnoDB: Compressed tables use zlib 1.2.3
2018-05-03 10:26:08 2611 [Note] InnoDB: Using Linux native AIO
2018-05-03 10:26:08 2611 [Note] InnoDB: Using CPU crc32 instructions
2018-05-03 10:26:08 2611 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2018-05-03 10:26:08 2611 [Note] InnoDB: Completed initialization of buffer pool
2018-05-03 10:26:08 2611 [Note] InnoDB: Highest supported file format is Barracuda.
2018-05-03 10:26:08 2611 [Note] InnoDB: 128 rollback segment(s) are active.
2018-05-03 10:26:08 2611 [Note] InnoDB: Waiting for purge to start
2018-05-03 10:26:08 2611 [Note] InnoDB: 5.6.40 started; log sequence number 1625977
2018-05-03 10:26:08 2611 [Note] Binlog end
2018-05-03 10:26:08 2611 [Note] InnoDB: FTS optimize thread exiting.
2018-05-03 10:26:08 2611 [Note] InnoDB: Starting shutdown...
2018-05-03 10:26:10 2611 [Note] InnoDB: Shutdown completed; log sequence number 1625987




A RANDOM PASSWORD HAS BEEN SET FOR THE MySQL root USER !
// 密码存放处
You will find that password in '/root/.mysql_secret'.

You must change that password on your first connect,
no other statement but 'SET PASSWORD' will be accepted.
See the manual for the semantics of the 'password expired' flag.

Also, the account for the anonymous user has been removed.

In addition, you can run:

  /usr/bin/mysql_secure_installation

which will also give you the option of removing the test database.
This is strongly recommended for production servers.

See the manual for more instructions.

Please report any problems at http://bugs.mysql.com/

The latest information about MySQL is available on the web at

  http://www.mysql.com

Support MySQL by buying support/licenses at http://shop.mysql.com

New default config file was created as /usr/my.cnf and
will be used by default by the server when you start it.
You may edit this file to change server settings
[root@root tmp]# rpm -ivh MySQL-server-5.6.40-1.el7.x86_64.rpm
// 安装客户端
[root@root tmp]# rpm -ivh MySQL-client-5.6.40-1.el7.x86_64.rpm
// 安装开发环境
[root@root tmp]# rpm -ivh MySQL-devel-5.6.40-1.el7.x86_64.rpm
```

#### 修改密码
> 使用此种RPM安装方式，直接查看 `/root/.mysql_secret` 获取初始密码，再进行修改
```
[root@root tmp]#  vim /root/.mysql_secret
# The random password set for the root user at Thu May  3 10:26:06 2018 (local time): PP5AHQi1LgirSWnu
[root@root tmp]# mysql -uroot -pPP5AHQi1LgirSWnu
Warning: Using a password on the command line interface can be insecure.
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
[root@rabbit3 tmp]# systemctl restart mysql
[root@rabbit3 tmp]# mysql -uroot -pPP5AHQi1LgirSWnu
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.40

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```
```
// 修改密码
mysql> SET PASSWORD = PASSWORD('123456');
```

#### 配置远程可登录
* 授权法
```
mysql> grant all privileges  on *.* to root@'%' identified by "123456";
// 推送设置到内存或重启服务器也行
mysql> FLUSH PRIVILEGES;
```
