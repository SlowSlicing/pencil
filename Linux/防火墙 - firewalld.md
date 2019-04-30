Linux
防火墙,firewalld,RHEL,CentOS
&emsp;&emsp;RHEL 7 系统中集成了多款防火墙管理工具，其中`firewalld`（Dynamic Firewall Manager of Linuxsystems，Linux 系统的动态防火墙管理器）服务是`默认的`防火墙配置管理工具，它拥有基于`CLI`（命令行界面）和基于`GUI`（图形用户界面）的两种管理方式。

&emsp;&emsp;相较于传统的防火墙管理配置工具，`firewalld` 支持动态更新技术并加入了区域（zone）的概念。简单来说，区域就是firewalld 预先准备了几套防火墙策略集合（`策略模板`），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。例如，我们有一台笔记本电脑，每天都要在办公室、咖啡厅和家里使用。按常理来讲，这三者的安全性按照由高到低的顺序来排列，应该是家庭、公司办公室、咖啡厅。当前，我们希望为这台笔记本电脑指定如下防火墙策略规则：在家中允许访问所有服务；在办公室内仅允许访问文件共享服务；在咖啡厅仅允许上网浏览。在以往，我们需要频繁地手动设置防火墙策略规则，而现在只需要预设好区域集合，然后只需轻点鼠标就可以自动切换了，从而极大地提升了防火墙策略的应用效率。firewalld 中常见的区域名称（`默认为public`）以及相应的策略规则如下表所示：

| 区域 | 默认策略规则 |
| --- | --- |
| trusted | 允许所有的数据包 |
| home | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client 与dhcpv6-client 服务相关，则允许流量 |
| internal | 等同于home 区域 |
| work | 拒绝流入的流量，除非与流出的流量数相关；而如果流量与ssh、ipp-client 与dhcpv6-client 服务相关，则允许流量 |
| public | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client 服务相关，则允许流量 |
| external | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh 服务相关，则允许流量 |
| dmz | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh 服务相关，则允许流量 |
| block | 拒绝流入的流量，除非与流出的流量相关 |
| drop | 拒绝流入的流量，除非与流出的流量相关 |

#### 终端管理工具

&emsp;&emsp;命令行终端是一种极富效率的工作方式，`firewall-cmd`是firewalld 防火墙配置管理工具的`CLI`（命令行界面）版本。它的参数一般都是以`长格式`来提供的，不要一听到长格式就头大，因为RHEL 7 系统支持部分命令的参数补齐，其中就包含这条命令（很酷吧）。也就是说，现在除了能用Tab 键自动补齐命令或文件名等内容之外，还可以用Tab 键来补齐下表中所示的长格式参数了（这太棒了）。

| 参数 | 作用 |
| --- | --- |
| `--get-default-zone`  | 查询默认的区域名称 |
| `--set-default-zone=<区域名称>`  | 设置默认的区域，使其永久生效 |
| `--get-zones`  | 显示可用的区域 |
| `--get-services`  | 显示预先定义的服务 |
| `--get-active-zones`  | 显示当前正在使用的区域与网卡名称 |
| `--add-source=` |  将源自此IP 或子网的流量导向指定的区域 |
| `--remove-source=`  | 不再将源自此IP 或子网的流量导向某个指定区域 |
| `--add-interface=<网卡名称>`  | 将源自该网卡的所有流量都导向某个指定区域 |
| `--change-interface=<网卡名称>`  | 将某个网卡与区域进行关联 |
| `--list-all`  | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
| `--list-all-zones`  | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
| `--add-service=<服务名>`  | 设置默认区域允许该服务的流量 |
| `--add-port=<端口号/协议>`  | 设置默认区域允许该端口的流量 |
| `--remove-service=<服务名>`  | 设置默认区域不再允许该服务的流量 |
| `--remove-port=<端口号/协议>`  | 设置默认区域不再允许该端口的流量 |
| `--reload`  | 让“永久生效”的配置规则立即生效，并覆盖当前的配置规则 |
| `--panic-on`  | 开启应急状况模式 |
| `--panic-off`  | 关闭应急状况模式 |

&emsp;&emsp;与Linux 系统中其他的防火墙策略配置工具一样，使用`firewalld` 配置的防火墙策略默认为运行时（`Runtime`）模式，又称为`当前生效模式`，而且随着系统的重启会失效。如果想让配置策略一直存在，就需要使用永久（`Permanent`）模式了，方法就是在用`firewall-cmd 命令`正常设置防火墙策略时添加`--permanent 参数`，这样配置的防火墙策略就可以永久生效了。但是，永久生效模式有一个`不近人情`的特点，就是使用它设置的策略只有在系统重启之后才能自动生效。如果想让配置的策略立即生效，需要手动执行`firewall-cmd --reload` 命令。

&emsp;&emsp;接下来的实验都很简单，注意使用的是`Runtime 模式`还是`Permanent 模式`。如果不关注这个细节，就算是正确配置了防火墙策略，也可能无法达到预期的效果。

&emsp;&emsp;**查看firewalld 服务当前所使用的区域：**

```
[root@lynchj ~]# firewall-cmd --get-default-zone
public
```

&emsp;&emsp;**查询eno16777728 网卡在firewalld 服务中的区域：**

```
[root@lynchj ~]# firewall-cmd --get-zone-of-interface=eno16777728
public
```

&emsp;&emsp;**把firewalld 服务中eno16777728 网卡的默认区域修改为external，并在系统重启后生效。分别查看当前与永久模式下的区域名称：**

```
[root@lynchj ~]# firewall-cmd --permanent --zone=external --change-interface=
eno16777728
success
[root@lynchj ~]# firewall-cmd --get-zone-of-interface=eno16777728
public
[root@lynchj ~]# firewall-cmd --permanent --get-zone-of-interface=eno16777728
external
```

&emsp;&emsp;**把firewalld 服务的当前默认区域设置为public：**

```
[root@lynchj ~]# firewall-cmd --set-default-zone=public
success
[root@lynchj ~]# firewall-cmd --get-default-zone
public
```

&emsp;&emsp;**启动/关闭firewalld 防火墙服务的应急状况模式，阻断一切网络连接（当远程控制服务器时请慎用）：**

```
[root@lynchj ~]# firewall-cmd --panic-on
success
[root@lynchj ~]# firewall-cmd --panic-off
success
```

&emsp;&emsp;**查询public 区域是否允许请求SSH 和HTTPS 协议的流量：**

```
[root@lynchj ~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@lynchj ~]# firewall-cmd --zone=public --query-service=https
no
```

&emsp;&emsp;**把firewalld 服务中请求HTTPS 协议的流量设置为永久允许，并立即生效：**

```
[root@lynchj ~]# firewall-cmd --zone=public --add-service=https
success
[root@lynchj ~]# firewall-cmd --permanent --zone=public --add-service=https
success
[root@lynchj ~]# firewall-cmd --reload
success
```

&emsp;&emsp;**把firewalld 服务中请求HTTP 协议的流量设置为永久拒绝，并立即生效：**

```
[root@lynchj ~]# firewall-cmd --permanent --zone=public --remove-service=http
success
[root@lynchj ~]# firewall-cmd --reload
success
```

&emsp;&emsp;**把在firewalld 服务中访问8080 和8081 端口的流量策略设置为允许，但仅限当前生效：**

```
[root@lynchj ~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
success
[root@lynchj ~]# firewall-cmd --zone=public --list-ports
8080-8081/tcp
```

&emsp;&emsp;**把原本访问本机888 端口的流量转发到22 端口，要且求当前和长期均有效：**

> &emsp;&emsp;流量转发命令格式为：`firewall-cmd --permanent -zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>`

```
[root@lynchj ~]# firewall-cmd --permanent --zone=public --add-forward-port=
port=888:proto=tcp:toport=22:toaddr=192.168.10.10
success
[root@lynchj ~]# firewall-cmd --reload
success
```

&emsp;&emsp;**在客户端使用ssh 命令尝试访问192.168.10.10 主机的888 端口：**

```
[root@client A ~]# ssh -p 888 192.168.10.10
The authenticity of host '[192.168.10.10]:888 ([192.168.10.10]:888)' can't
be established.
ECDSA key fingerprint is b8:25:88:89:5c:05:b6:dd:ef:76:63:ff:1a:54:02:1a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.10.10]:888' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: root
Last login: Sun Jul 19 21:43:48 2017 from 192.168.10.10
```

&emsp;&emsp;**firewalld 中的`富规则`表示更细致、更详细的防火墙策略配置，它可以针对`系统服务`、`端口号`、`源地址`和`目标地址`等诸多信息进行更有`对性的策略配置`。它的优先级在所有的防火墙策略中也是`最高的`。比如，我们可以在firewalld 服务中配置一条富规则，使其拒绝192.168.10.0/24 网段的所有用户访问本机的ssh 服务（22 端口）：**

```
[root@lynchj ~]# firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.10.0/24" service name="ssh" reject"
success
[root@lynchj ~]# firewall-cmd --reload
success
```

&emsp;&emsp;**在客户端使用ssh 命令尝试访问192.168.10.10 主机的ssh 服务（22 端口）：**

```
[root@client A ~]# ssh 192.168.10.10
Connecting to 192.168.10.10:22...
Could not connect to '192.168.10.10' (port 22): Connection failed.
```

#### 常用命令组合

* 对指定的 IP 开放指定的端口段

```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="30000-31000" accept"
```
