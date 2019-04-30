MySQL
MySQL,主主复制,负载均衡,HAProxy,Keepalived
[toc]

> 搭建环境：
>> 两台 CentOS 7.4（192.168.117.139、192.168.117.140）
>> MySQL 5.7
>> HAProxy 1.5.18
>> Keepalived 1.3.5

# MySQL 安装

&emsp;&emsp;安装笔记参考[MySQL Yum 存储库安装](https://blog.csdn.net/wo18237095579/article/details/80585486)，此处不再浪费“墨水”。两台机器按照相同步骤安装完成即可。

# 主主复制

&emsp;&emsp;什么叫主主复制？就是两个 MySQL 都能读能写，数据记录通过二进制传达给对方从而保持数据的一致性。

> 192.168.117.139 主从复制 + 192.168.117.140 主从复制 == 192.168.117.139、192.168.117.140 主主复制）

&emsp;&emsp;因此主主复制中必须要解决的事情就是自增主键的问题。如果 `MySQL1` 主键 id 增加到 999 了，此时二进制数据还没到达 MySQL2，那么 MySQL2 恰好要插入数据，那么新数据主键 id 也是 999，那不就是乱套了么！解决这一问题我们可以直接更改MySQL中的配置文件即可。

### 修改配置文件

&emsp;&emsp;如果按照上方笔记安装的话，MySQL 配置文件在：`/etc/my.cnf`

```
# 192.168.117.139 服务器

[mysqld]

log-bin=mysql-bin # 开启二进制日志
binlog_format=mixed
server-id       = 1 # 任意自然数n，只要保证两台MySQL主机不重复就可以了。

auto_increment_increment=2 # 步进值auto_imcrement。一般有n台主 MySQL 就填n
auto_increment_offset=1 # 起始值。一般填第n台主MySQL。此时为第一台主MySQL
binlog-ignore-db=mysql # 忽略mysql库，一般不需要写
binlog-ignore-db=information_schema # 忽略information_schema库，一般不需要写
replicate-do-db=aa # 要同步的数据库，默认所有库

# 192.168.117.140 服务器

[mysqld]

server-id=2
log-bin=mysql-bin

auto_increment_increment=2
auto_increment_offset=2
replicate-do-db=aa
```

&emsp;&emsp;配置好后重启 MySQL

```
systemctl restart mysqld
```

> &emsp;&emsp;**注意：**如果使用的是虚拟机，进行的克隆，因为是整个系统的 "Copy" ，所以 MySQL 的 auto.cnf 文件中的 `server-uuid` 会相同，在后面主主复制操作时会出错，记得修改，两个不一样即可。auto.cnf 地址：`/var/lib/mysql/auto.cnf`

### 配置 192.168.117.139 的主从复制

#### 创建 MySQL 用户

&emsp;&emsp;在 `192.168.117.139` 的 MySQL 上创建 `192.168.117.140` MySQL 登录使用 MySQL 用户并授予`从复制权限`（反之同样来一遍）。

```
# 创建 MySQL 用户
mysql> CREATE USER 'mysql140'@'192.168.117.140' IDENTIFIED BY 'Mysql*140';
Query OK, 0 rows affected (0.00 sec)

# 授权
mysql> GRANT REPLICATION SLAVE ON *.* TO 'mysql140'@'192.168.117.140' IDENTIFIED BY 'Mysql*140';
Query OK, 0 rows affected, 1 warning (0.00 sec)

# 刷新权限
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

#### 查看二进制文件

```
mysql> show master status;
+------------------+----------+--------------+--------------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         | Executed_Gtid_Set |
+------------------+----------+--------------+--------------------------+-------------------+
| mysql-bin.000001 |      885 |              | mysql,information_schema |                   |
+------------------+----------+--------------+--------------------------+-------------------+
1 row in set (0.00 sec)
```

#### 告知 192.168.117.140 主 MySQL 二进制文件名与位置

&emsp;&emsp;在 `192.168.117.140` 上的 MySQL 中执行

* 
```
mysql> change master to master_host='192.168.117.139', master_user='mysql140', master_password='Mysql*140', master_log_file='mysql-bin.000001', master_log_pos=885;
```

#### 查看结果

```
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.117.139
                  Master_User: mysql140
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 885
               Relay_Log_File: localhost-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: aa
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 885
              Relay_Log_Space: 531
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 8a6cacf5-8029-11e8-ae8e-000c291537e4
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

&emsp;&emsp;**上面几个关键点注意：**

* **Master_Host:** 主库地址
* **Master_User:** 连接主库使用的 用户
* **Master_Log_File:** 主库二进制文件
* **Read_Master_Log_Pos:** 主库二进制文件坐标
* **Slave_IO_Running:** 与主库 IO 通信状态，需要为 `Yes` 方可
* **Slave_SQL_Running:** 负责自己的 Slave MySQL 进行状态，需要为 `Yes` 方可
* **Replicate_Do_DB:** 同步的 DB

> &emsp;&emsp;至此，192.168.117.139 的主从复制配置成功。这些操作在 192.168.117.140 上再做一遍即可完成**主主复制**

### 查看主主复制效果

&emsp;&emsp;这里使用 Navicat 视图工具随便添加一点数据

* 192.168.117.139 上新增 demo 表

![139新增 demo 表](http://img.lynchj.com/8b7f4288eee6477b9257b963d1cb1d07.png)

* 刷新 192.168.117.140 发现已经同步成功

![已同步 demo](http://img.lynchj.com/1f91b360eba84c4b8900b0e39937a9b0.png)

* 192.168.117.140 上新增 test 表

![140新增 test 表](http://img.lynchj.com/49bc8b30bc384d6fb91d9db95d7cd532.png)

* 刷新 192.168.117.139 发现已经同步成功

![已同步 test](http://img.lynchj.com/c2dad2f245954e80b61b4caf536718e2.png)

# 中间件说明

### HAProxy

&emsp;&emsp;HAProxy 是一个开源的高性能的反向代理或者说是负载均衡服务软件之一，它支持双机热备、虚拟主机、基于TCP和HTTP应用代理等功能。其配置简单，而且拥有很好的对服务器节点的健康检查功能（相当于 keepalived 健康检查），当其代理的后端服务器出现故障时，HAProxy 会自动的将该故障服务器摘除，当服务器的故障恢复后 HAProxy 还会自动将 RS 服务器。

&emsp;&emsp;HAProxy 特别适用于那些负载特大的 web 站点，这些站点通常又需要会话保持或七层处理。HAProxy 运行在当前的硬件上，完全可以支持数以万计的并发连接。并且它的运行模式使得它可以很简单安全的整合进您当前的架构中， 同时可以保护你的 web 服务器不被暴露到网络上。

&emsp;&emsp;HAProxy 软件引入了 frontend，backend 的功能，frontend（acl规则匹配）可以根据任意HTTP请求头做规则匹配，然后把请求定向到相关的 backend（server pools等待前端把请求转过来的服务器组）。通过 frontend 和 backup，我们可以很容易的实现 HAProxy 的`7层代理功能`，HAProxy 是一款不可多得的优秀代理服务软件。

### Keepalived

&emsp;&emsp;Keepalived 是以 `VRRP协议` 为实现基础的，VRRP 全称 Virtual Router Redundancy Protocol，即虚拟路由冗余协议。

&emsp;&emsp;虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个 Master 和多个 Backup，Master 上面有一个对外提供服务的 `vip`（该路由器所在局域网内其他机器的默认路由为该 `vip`），Master 会发组播，当 Backup 收不到 `vrrp` 包时就认为 Master 宕掉了，这时就需要根据 `VRRP` 的优先级来选举一个 Backup 当 Master。这样的话就可以保证路由器的高可用了。

&emsp;&emsp;Keepalived 主要有三个模块，分别是 `core`、`check` 和 `vrrp`。`core模块`为 Keepalived 的核心，负责主进程的启动、维护以及全局配置文件的加载和解析。`check`负责健康检查，包括常见的各种检查方式。`vrrp`模块是来实现`VRRP协议`的。

# 中间件的安装与配置（HAProxy、Keepalived）

&emsp;&emsp;我这里全部都采用 yum 安装方式，源使用的是阿里云。

### HAProxy 安装

#### 安装

```
[root@localhost ~]# yum -y install haproxy
```

#### 配置

&emsp;&emsp;配置文件路径：`/etc/haproxy/haproxy.cfg`

&emsp;&emsp;添加关于 MySQL 的配置信息

```
[root@localhost ~]# vim /etc/haproxy/haproxy.cfg

# 配置查看负载信息管理页面
listen stats
    mode http
    bind :8888
    stats enable
    stats hide-version
    stats uri     /haproxyadmin?stats
    stats realm   Haproxy\ Statistics
    stats auth    admin:admin

# MySQL 负载配置
listen proxy-mysql
    bind 0.0.0.0:13306
    mode tcp
    balance roundrobin
    option tcplog
    #option mysql-check user haproxy
    server MySQL1 192.168.117.139:3306 check weight 1 maxconn 2000
    server MySQL2 192.168.117.140:3306 check weight 1 maxconn 2000
    option tcpka
```

* **配置说明：**

```
global

    log         127.0.0.1 local2         //日志定义级别
    chroot      /var/lib/haproxy         //当前工作目录
    pidfile     /var/run/haproxy.pid     //进程id
    maxconn     4000                     //最大连接数
    user        haproxy                  //运行改程序的用户
    group       haproxy
    daemon                               //后台形式运行
    stats socket /var/lib/haproxy/stats

defaults
    mode                    tcp            //haproxy运行模式（http | tcp | health）
    log                     global
    option                  dontlognull
    option                  redispatch     //serverId对应的服务器挂掉后,强制定向到其他健康的服务器
    retries                 3              //三次连接失败则服务器不用
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s            //连接超时
    timeout client          1m             //客户端超时
    timeout server          1m             //服务器超时
    timeout http-keep-alive 10s
    timeout check           10s            //心跳检测
    maxconn                 600            //最大连接数

listen stats                               //配置haproxy状态页（用来查看的页面）
    mode http
    bind :8888
    stats enable
    stats hide-version                    //隐藏haproxy版本号
	stats uri     /haproxyadmin?stats     //一会用于打开状态页的uri
    stats realm   Haproxy\ Statistics     //输入账户密码时的提示文字
    stats auth    admin:admin             //用户名:密码

# MySQL 负载配置
listen proxy-mysql 
    bind 0.0.0.0:13306 # 监听端口
    mode tcp # 模式
    balance roundrobin # 负载均衡的方式，轮询（平均）方式
    option tcplog # 允许记录tcp 连接的状态和时间
    #option mysql-check user haproxy
    server MySQL1 192.168.117.139:3306 check weight 1 maxconn 2000
    server MySQL2 192.168.117.140:3306 check weight 1 maxconn 2000
    option tcpka # 是否允许向server和client发送keepalive
```

#### 启动 HAProxy

&emsp;&emsp;重启 HAProxy 

```
[root@localhost ~]# systemctl restart haproxy
```

&emsp;&emsp;**启动失败，查看错误信息如下：**

```
[root@localhost ~]# tail -f /var/log/messages
Jul  7 05:36:20 localhost systemd: Started HAProxy Load Balancer.
Jul  7 05:36:20 localhost systemd: Starting HAProxy Load Balancer...
Jul  7 05:36:20 localhost haproxy-systemd-wrapper: [WARNING] 187/053620 (5022) : config : 'option forwardfor' ignored for proxy 'proxy-mysql' as it requires HTTP mode.
Jul  7 05:36:20 localhost haproxy-systemd-wrapper: [ALERT] 187/053620 (5022) : Starting proxy stats: cannot bind socket [0.0.0.0:8888]
Jul  7 05:36:20 localhost haproxy-systemd-wrapper: [ALERT] 187/053620 (5022) : Starting proxy proxy-mysql: cannot bind socket [0.0.0.0:13306]
Jul  7 05:36:20 localhost haproxy-systemd-wrapper: haproxy-systemd-wrapper: exit, haproxy RC=1
Jul  7 05:36:20 localhost systemd: haproxy.service: main process exited, code=exited, status=1/FAILURE
Jul  7 05:36:20 localhost systemd: Unit haproxy.service entered failed state.
Jul  7 05:36:20 localhost systemd: haproxy.service failed.
```

* 关闭 `防火墙`、`SELinux` 再次尝试重启，成功。

```
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# setenforce 0
[root@localhost ~]# systemctl restart haproxy
[root@localhost ~]# ps -ef | grep haproxy
root       5044      1  0 05:37 ?        00:00:00 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
haproxy    5045   5044  0 05:37 ?        00:00:00 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
haproxy    5046   5045  0 05:37 ?        00:00:00 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
root       5048   1146  0 05:37 pts/0    00:00:00 grep --color=auto haproxy
```

#### 查看负载情况

* 访问地址：http://192.168.117.139:8888/haproxyadmin?stats

![负载状态](http://img.lynchj.com/60f0cd0ba38647ae9e84e8441b4bbe8b.png)

> &emsp;&emsp;安装 HAProxy 的步骤在 `192.168.117.140` 上做相同步骤即可。

### Keepalived 安装

#### 安装

```
[root@localhost ~]# yum -y install keepalived
```

#### 配置

&emsp;&emsp;配置文件路径：`/etc/keepalived/keepalived.conf`

&emsp;&emsp;配置 Keepalived

```
! Configuration File for keepalived
#简单的头部，这里主要可以做邮件通知报警等的设置，此处就暂不配置了；
global_defs {
        notificationd LVS_DEVEL
}
#预先定义一个脚本，方便后面调用，也可以定义多个，方便选择；
vrrp_script chk_haproxy {
    script "/etc/keepalived/chk.sh"  #具体脚本路径
    interval 2  #脚本循环运行间隔
}
#VRRP虚拟路由冗余协议配置
vrrp_instance VI_1 {   #VI_1 是自定义的名称；
    state MASTER #MASTER表示是一台主设备，BACKUP表示为备用设备【我们这里因为设置为开启不抢占，所以都设置为备用】
    nopreempt      #开启不抢占
    interface ens33   #指定VIP需要绑定的物理网卡
    virtual_router_id 11   #VRID虚拟路由标识，也叫做分组名称，该组内的设备需要相同
    priority 120   #定义这台设备的优先级 1-254；开启了不抢占，所以此处优先级必须高于另一台

    advert_int 1   #生存检测时的组播信息发送间隔，组内一致
    authentication {    #设置验证信息，组内一致
        auth_type PASS   #有PASS 和 AH 两种，常用 PASS
        auth_pass asd    #密码
    }
    virtual_ipaddress {
        192.168.117.111    #指定VIP地址，组内一致，可以设置多个IP
    }
    track_script {    #使用在这个域中使用预先定义的脚本，上面定义的
        chk_haproxy   
    }

    notify_backup "systemctl restart haproxy"   #表示当切换到backup状态时,要执行的脚本
    notify_fault "systemctl stop haproxy"     #故障时执行的脚本
}

```

&emsp;&emsp;几个点，说明下：

* **state：**如果是主机的话，值要写成 `MASTER`，备机写成 `BACKUP`
* **interface：**物理机网卡名称，我的这个虚拟机上的现有网卡名称是 `ens33`
* **priority：** 定义这台设备的优先级 1-254；备机小于主机
* **virtual_ipaddress：**指定的虚拟 IP，可以设置多个，但是一定要是在当前网段，否则设置了也无法访问。

#### 编写脚本文件

&emsp;&emsp;脚本文件是为了检测 HAProxy 程序是否还在正常运行，两秒钟检测一次，如果程序已经挂掉，则把当前主 Keepalived 一同挂掉，这样就可以让备机接管 `VIP`（虚拟IP），达到高可用的目的。

```
[root@localhost ~]# vim /etc/keepalived/chk.sh
#!/bin/bash
#
if [ $(ps -C haproxy --no-header | wc -l) -eq 0 ]; then
       systemctl stop keepalived
fi

[root@localhost ~]# chmod 777 /etc/keepalived/chk.sh
```

> &emsp;&emsp;此处检测程序较为简单，比较保险的做法可以参考：多次检测负载的服务是否正常可接受访问，如多次访问失败则停止主 Keepalive。


#### 启动 Keepalived

```
[root@localhost ~]# systemctl restart keepalived
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:15:37:e3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.117.139/24 brd 192.168.117.255 scope global noprefixroute dynamic ens33
       valid_lft 1632sec preferred_lft 1632sec
    inet 192.168.117.111/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::fdf1:237c:2a96:81c6/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

&emsp;&emsp;可以看到，上方`ens33`网卡下有一个新增的 ip，**inet 192.168.117.111/32 scope global ens33**

> &emsp;&emsp;安装 Keepalived 的步骤在 `192.168.117.140` 上做相同步骤即可。

# 测试

### 测试环境

* Spring Boot
* spring-boot-starter-data-jpa

### 配置项

```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.117.111:13306/aa?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8
    username: 你的用户名
    password: 你的密码
```

### 测试伪代码

```
################# Controller
import com.lynchj.mysqlzhuzhucopy.component.MyConstants;
import com.lynchj.mysqlzhuzhucopy.entity.User;
import com.lynchj.mysqlzhuzhucopy.service.UserServiceInter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.math.BigDecimal;
import java.util.Date;

@RestController
@RequestMapping("/user")
@Slf4j
public class UserController {

    @Resource
    private UserServiceInter userServiceInter;

    private static Integer count = 0;

    @RequestMapping("/saveUserTwo")
    public User saveUserTwo() {

        System.err.println("当前第 [" + ++count + "] 条数据" );
        
        User user = new User();
        user.setId(MyConstants.idWorkerUtil.nextId() + "");
        user.setCreateTime(new Date());
        user.setName("吼吼吼");
        user.setBirthday(new Date());
        user.setHeight(new BigDecimal("1.74"));

        return userServiceInter.saveUser(user);

    }

}

################# Service
import com.lynchj.mysqlzhuzhucopy.dao.UserDaoInter;
import com.lynchj.mysqlzhuzhucopy.entity.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
@Slf4j
public class UserService implements UserServiceInter {

    @Resource
    private UserDaoInter userDaoInter;

    @Override
    public User saveUser(User user) {

        return userDaoInter.save(user);

    }

}

################# Dao
public interface UserDaoInter extends JpaRepository<User, String> {

    @Override
    User save(User user);

}

################# entity
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Id;
import java.math.BigDecimal;
import java.util.Date;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class User {

    @Id
    private String id;

    private Date createTime;

    private String name;

    private Integer age;

    private Date birthday;

    private BigDecimal height;

}
```

### 测试

&emsp;&emsp;我这里是用 JMeter 进行测试，以 `每秒两条线程请求` 的频率进行存储数据，中间会手动杀死主设备上的 HAProxy，观察 Keepalived 的 VIP 是否已经`飘到`备用设备上并且程序是否会自动切换。

![JMeter 测试](http://img.lynchj.com/9df8e3ffea504bfbb4521154ef35534e.png)

* 观察控制台打印：

![控制台输出结果](http://img.lynchj.com/400be3c6048f4882abd9f09418173985.png)

&emsp;&emsp;中间手动 kill 掉了主机上的 HAProxy，出现了错误信息呢，在之后又继续新增操作。

* 此时查看备机的 IP 信息如下：

```
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:83:3b:07 brd ff:ff:ff:ff:ff:ff
    inet 192.168.117.140/24 brd 192.168.117.255 scope global noprefixroute dynamic ens33
       valid_lft 1505sec preferred_lft 1505sec
    inet 192.168.117.111/32 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::f6e7:5569:d37:4950/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::fdf1:237c:2a96:81c6/64 scope link tentative noprefixroute dadfailed
       valid_lft forever preferred_lft forever
```

> Over
