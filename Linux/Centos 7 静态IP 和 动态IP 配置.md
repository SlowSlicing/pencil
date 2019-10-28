[toc]

# 服务器网口信息

```
$ ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: em1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 20:04:0f:e7:e4:1c brd ff:ff:ff:ff:ff:ff
3: em2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 20:04:0f:e7:e4:1d brd ff:ff:ff:ff:ff:ff
4: em3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 20:04:0f:e7:e4:1e brd ff:ff:ff:ff:ff:ff
5: em4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 20:04:0f:e7:e4:1f brd ff:ff:ff:ff:ff:ff
```

> 　　上面那些 `link/ether` 后面的 `20:04:0f:e7:e4:1*` 是网口的 Mac 地址

　　我这台服务器有四个网口，**我把网线插在第一个网口上**（这里已经保证上级网络没有问题）。同时要保证服务器的网络服务是启动状态。

```
$ systemctl status network
● network.service - LSB: Bring up/down networking
   Loaded: loaded (/etc/rc.d/init.d/network; bad; vendor preset: disabled)
   Active: active (exited) since 三 2019-07-31 10:03:13 CST; 2 months 18 days ago
     Docs: man:systemd-sysv-generator(8)
  Process: 1037 ExecStart=/etc/rc.d/init.d/network start (code=exited, status=0/SUCCESS)

7月 31 10:03:13 demo systemd[1]: Starting LSB: Bring up/down networking...
7月 31 10:03:13 demo network[1037]: 正在打开环回接口： [  确定  ]
7月 31 10:03:13 demo network[1037]: 正在打开接口 em1： [  确定  ]
7月 31 10:03:13 demo network[1037]: 正在打开接口 house： [  确定  ]
7月 31 10:03:13 demo systemd[1]: Started LSB: Bring up/down networking.
```

　　如果不是 `Active: active` 最好先启动网络服务：`systemctl start network`。

# 动态 IP

```
# 进入网络配置文件目录
$ cd /etc/sysconfig/network-scripts/
# 编辑配置文件，添加修改以下内容
$ vim ifcfg-em1

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
# DHCP 动态获取
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=em1
UUID=eaeb21e1-b380-4060-983a-06b333e0e6b7
DEVICE=em1
# 开机自动启用网络连接
ONBOOT=yes

# 保存并退出
$ :wq
```

> 　　网卡的 UUID 获取方法为：`uuidgen em1`，后面的参数为网口名。

　　重启之后生效。

# 静态 IP

```
# 进入网络配置文件目录
$ cd /etc/sysconfig/network-scripts/
# 编辑配置文件，添加修改以下内容
$ vim ifcfg-em1

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
# 静态获取
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=em1
UUID=eaeb21e1-b380-4060-983a-06b333e0e6b7
DEVICE=em1
ONBOOT=yes
# 静态 IP
IPADDR=192.168.2.217
# 网关
GATEWAY=192.168.2.1
# 子网掩码
NETMASK=255.225.255.0
ZONE=public

# 保存并退出
$ :wq
```

　　重启之后生效。