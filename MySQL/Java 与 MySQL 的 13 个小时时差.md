[toc]

# 起因

　　事情是这样的，在 Java 中执行代码是这样的：

![](https://pencil.file.lynchj.com/blog/20190517100520.png)

　　上面是插入前的数据截图，注意`时间`。数据库的时区查询、`now()` 函数查询、与标准时间的时差查询是这样的：

![时区](https://pencil.file.lynchj.com/blog/20190517100957.png)
![时间](https://pencil.file.lynchj.com/blog/20190517101023.png)
![时差](https://pencil.file.lynchj.com/blog/20190517102306.png)

　　最终插入到数据库中的数据是这样的：

![](https://pencil.file.lynchj.com/blog/20190517101220.png)

　　What？？？什么情况，一下还做了两种测试情况：

1. 直接把 Java 代码和 MySQL 放在同一个服务器上执行，`时差一样存在`；
2. 把 Java 中的 MyBatis 执行的 sql copy 到 MySQL 客户端（mysql/Navicat）中执行，`没有时差问题`；

　　这里的几个点：

1. MySQL 客户端（mysql/Navicat）执行没问题；
2. Java 代码中获取当前时间没问题；
3. Java 代码所在服务器的时区和当前时间没问题；
4. MySQL 服务的时区和当前时间没问题；

　　这里根据以上几点推断是因为 JDBC 连接到 MySQL 把 CST 时区识别成了 `Central Standard Tim UT-6:00` **美国中部标准时间**，如果是夏令时，就是 `Central Standard Tim UT-5:00`，中国所在时区是 `+8:00`，这里美国是 `-5:00`，正好相差 13 小时。

　　这里的 CST？CST 并不仅仅只是代表了中国的东八区，而是代表了四个时区：

* **Central Standard Time (USA) UT-6:00**
* **Central Standard Time (Australia) UT+9:30**
* **China Standard Time UT+8:00**
* **Cuba Standard Time UT-4:00**

# 解决之道

　　解决办法有两种。

* 是在 Java 中 JDBC 连接配置出加入 `serverTimezone=GMT%2B8` 或者 `serverTimezone=Hongkong`

![](https://pencil.file.lynchj.com/blog/20190517102941.png)

* 修改 MySQL 的配置参数，在 MySQL 配置文件 `/etc/my.cnf` 中加入 `default_time_zone=+8:00` 配置项

```
......
[mysqld]
default_time_zone=+8:00
......
```
