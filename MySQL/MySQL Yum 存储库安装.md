# 全新安装MySQL的步骤

> 　　请按照以下步骤使用MySQL Yum存储库安装最新的GA版本的MySQL。

### 添加MySQL Yum存储库

　　首先，将MySQL Yum存储库添加到系统的存储库列表中。这是一次性操作，可以通过安装MySQL提供的RPM来执行。按着这些次序：

1.转至MySQL开发人员专区中的`Download MySQL Yum Repository`页面（http://dev.mysql.com/downloads/repo/yum/）。
2.选择并下载适用于您的平台的发行包。
3.使用以下命令安装下载的发行包，替换 `platform-and-version-specific-package-name` 为下载的RPM包的名称：

```
shell> sudo yum localinstall platform-and-version-specific-package-name.rpm
```

　　对于基于EL6的系统，该命令的形式为：

```
sudo yum localinstall mysql80013-community-release-el6-{version-number}.noarch.rpm
```

　　对于基于EL7的系统：

```
sudo yum localinstall mysql80013-community-release-el7-{version-number}.noarch.rpm
```

　　安装命令将MySQL Yum存储库添加到系统的存储库列表中，并下载GnuPG密钥以检查软件包的完整性。有关GnuPG密钥检查的详细信息，请参见 [第2.1.3.2节](https://dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature.html)`使用GnuPG进行`[签名检查](https://dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature.html)。

　　您可以通过以下命令检查MySQL Yum存储库是否已成功添加（对于启用了`dnf`的系统，请使用`dnf`替换命令中的yum）：

```
yum repolist enabled | grep "mysql.*-community.*"
```

　　**注意：**

> 　　一旦您的系统上启用了MySQL Yum存储库，`yum update` 命令进行的任何系统范围的更新（或者`dnf-enabled`系统的dnf升级）都将升级系统上的MySQL软件包，请参见 第[2.10.1.3节](https://dev.mysql.com/doc/refman/8.0/en/updating-yum-repo.html)`使用MySQL Yum资源库升级MySQL`以及有关可能对您的系统产生的影响的讨论。

### 选择一个发布系列

　　使用MySQL Yum存储库时，默认情况下会选择最新的GA系列（当前为MySQL 8.0，2018年6月5日18:22:58）进行安装。如果这是你想要的，你可以跳到下一步， 安装MySQL。

　　在MySQL Yum存储库中，不同版本的MySQL社区服务器系列托管在不同的子库中。最新GA系列的子库（当前为MySQL 8.0）默认情况下处于启用状态，其他系列的子库（例如MySQL 5.7系列）默认处于禁用状态。使用此命令查看MySQL Yum存储库中的所有子存储库，并查看其中哪些被启用或禁用（对于启用了dnf的系统，用dnf替换该命令中的 yum）：

```
yum repolist all | grep mysql
```

　　要安装最新GA系列的最新版本，不需要配置。要从最新GA系列以外的特定系列安装最新版本，请在运行安装命令之前禁用最新GA系列的子库，并为特定系列启用子库。如果你的平台支持 `yum-config-manager`，你可以通过发布这些命令来实现这一点，这些命令禁用了5.7系列的子库并启用了8.0系列的子库：

```
sudo yum-config-manager --disable mysql57-community
sudo yum-config-manager --enable mysql80-community
```

　　对于支持dnf的平台：

```
sudo dnf config-manager --disable mysql57-community
sudo dnf config-manager --enable mysql80-community
```

　　除了使用`yum-config-manager`或 `dnf config-manager`命令之外，您还可以通过手动编辑`/etc/yum.repos.d/mysql-community.repo` 文件来选择发行版系列 。这是该文件中的 MySQL 5.7 子版本库的典型配置：

```
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

　　找到要配置的子存储库的条目，然后编辑该`enabled`选项。指定 `enabled=0`禁用子库，或 `enabled=1`启用子库。例如，要安装MySQL 8.0，请确保您的 MySQL 5.7 子库中的 `enabled = 0`，并且您的 MySQL 8.0 子库中的 `enabled = 1`：

```
# Enable to use MySQL 8.0
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

　　您应该只能在任何时候为一个发行版系列启用子库。当启用多个版本系列的子库时，Yum会使用最新系列。

　　通过运行以下命令并检查其输出（对于启用了dnf的系统，用dnf替换命令中的yum）， 验证是否已启用和禁用正确的子库 ：

```
yum repolist enabled | grep mysql
```

### 安装MySQL

　　通过以下命令安装MySQL（对于启用了dnf的系统，请使用dnf替换命令中的 yum）：

```
sudo yum install mysql-community-server
```

　　这将安装MySQL服务器（`mysql-community-server`）的软件包以及运行服务器所需组件的软件包，包括客户端软件包（`mysql-community-client`），客户端和服务器（`mysql-community-common`）的常见错误消息和字符集以及共享客户端库（mysql-community-libs） 。

### 启动MySQL服务器

　　使用以下命令启动MySQL服务器：

```
systemctl start mysqld
```

　　您可以使用以下命令检查MySQL服务器的状态：

```
systemctl status mysqld
```

　　在服务器初始启动时，如果服务器的数据目录为空，则会发生以下情况：

* 服务器已初始化。
* 数据目录中会生成SSL证书和密钥文件。
* [validate_password](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html) 已安装并启用。
* 超级用户帐户`'root'@'localhost`已创建。超级用户的密码被设置并存储在日志文件中。要显示它，请使用以下命令：

```
sudo grep 'temporary password' /var/log/mysqld.log
```

　　**通过使用生成的临时密码登录并尽快更改root密码并为超级用户帐户设置自定义密码**：

```
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码';
```

　　**注意：**

> 　　[validate_password](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html) 是默认安装的。实施的默认密码策略validate_password要求密码至少包含一个大写字母，一个小写字母，一个数字和一个特殊字符，并且总密码长度至少为8个字符。

### 允许外网连接

```
grant all privileges on *.* to 'root'@'%' identified by '你的密码' with grant option
flush privileges;
```

# 卸载

### 卸载rpm包

```
rpm -qa | grep -i mysql
yum remove 上条命令检索出来的各个包名
```

### 删除mysql文件目录

```
find / -name mysql
find / -name mysql -exec rm -rf {} \;
```

* 至此MySQL已成功卸载

> 参考：https://dev.mysql.com/doc/refman/8.0/en/linux-installation-yum-repo.html
