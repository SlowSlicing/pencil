[toc]

# Firewalld

　　Linux 上新的防火墙软件，和原来的 Iptables 作用差不多。
　　`firewall-cmd` 是 `firewalld` 的命令行管理工具，firewalld 是 CentOS 7 的一大特性。
　　firewalld 自身并不具备防火墙的功能，而是和 iptables 一样需要通过内核的 `netfilter` 来实现，也就是说 firewalld 和 iptables 一样，他们的作用都是用于维护规则，而真正使用规则干活的是内核的 netfilter，只不过 firewalld 和 iptables 的结构以及使用方法不一样罢了。

## 好处

* 支持动态更新，不用重启服务；
* 加入了防火墙的 `zone` 概念；

>　　`firewalld` 可以动态修改单条规则，而不需要像 `iptables` 那样，在修改了规则后必须得全部刷新才可以生效。`firewalld` 在使用上要比 `iptables` 人性化很多，即使不明白**五张表五条链**而且对 `TCP/IP` 协议也不理解也可以实现大部分功能。

# 详解

## zone

* 网络区域简介

　　通过将网络划分成不同的区域（通常情况下称为 `zones`），制定出不同区域之间的访问控制策略来控制不同任程度区域间传送的数据流。例如互联网是不可信任的区域，而内部网络是高度信任的区域。以避免安全策略中禁止的一些通信。它有控制信息基本的任务在不同信任的区域。典型信任的区域包括互联网 ( 一个没有信任的区域 ) 和一个内部网络 ( 一个高信任的区域 )。最终目标是提供受控连通性在不同水平的信任区域通过安全政策的运行和连通性模型之间根据最少特权原则。例如：公共 WIFI 网络连接应该不信任，而家庭有线网络连接就应该完全信任。网络安全模型可以在安装、初次启动和首次建立网络连接时选择初始化。该模型描述了主机所联的整个网络环境的可信级别，并定义了新连接的处理方式。在 `/etc/firewalld/` 的区域设定是一系列可以被快速执行到网络接口的预设定。有几种不同的初始化区域：

1. **drop（丢弃）：**任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。
2. **block（限制）：**任何接收的网络连接都被 IPv4 的 icmp-host-prohibited 信息和 IPv6 的 icmp6-adm-prohibited 信息所拒绝。
3. **public（公共）：**在公共区域内使用，不能相信网络内的其他计算机不会对您的计算机造成危害，只能接收经过选取的连接。
4. **external（外部）：**特别是为路由器启用了伪装功能的外部网。您不能信任来自网络的其他计算，不能相信它们不会对您的计算机造成危害，只能接收经过选择的连接。
5. **dmz（非军事区）：**用于您的非军事区内的电脑，此区域内可公开访问，可以有限地进入您的内部网络，仅仅接收经过选择的连接。
6. **work（工作）：**用于工作区。您可以基本相信网络内的其他电脑不会危害您的电脑。仅仅接收经过选择的连接。
7. **home（家庭）：**用于家庭网络。您可以基本信任网络内的其他计算机不会危害您的计算机。仅仅接收经过选择的连接。
8. **internal（内部）：**用于内部网络。您可以基本上信任网络内的其他计算机不会威胁您的计算机。仅仅接受经过选择的连接。
9. **trusted（信任）：**可接受所有的网络连接。

>　　说明：firewalld 的默认区域是 public。

　　显示所有支持的分区：

```
$ firewall-cmd --get-zones 
block drop work internal external home dmz public trusted
```

### 网络请求如何判断使用哪个 zone 的

　　firewalld 是如何规定一个网络请求使用的时哪个 zone 上的规则呢？首先会根据 zone 上绑定的 source 进行判断，然后再根据 interface 进行判断。比如：我现在有两台机器，一台我本机（192.168.1.188），另一个是虚拟机（192.168.1.61）。以下操作未说明的情况下都是在虚拟机上：

* 查看两个 zone 的信息

```
$ firewall-cmd --list-all --zone=work
work (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth1
  sources:
  services: ssh dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

$ firewall-cmd --list-all --zone=home
home
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh mdns samba-client dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

　　可以看到网卡绑定在 `zone work` 上面，而 `zone home` 基本没什么东西

* 在 `zone home` 绑定 `source 192.168.1.188`

```
$ firewall-cmd --add-source=192.168.1.188 --zone=home
success

$ firewall-cmd --list-all --zone=home
home (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.1.188
  services: ssh mdns samba-client dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

* 这里想做的是：在我本机上（192.168.1.188）上面启动一个 tomcat 服务，监听 8080 端口，从本机上向虚拟机（192.168.1.61）的 8093 端口发起请求，在虚拟机上配置端口转发到 192.168.1.188 的 8080 端口上面，访问看效果

```
# 先开启伪装 ip，支持转发
$ firewall-cmd --add-masquerade --zone=home
success
$ firewall-cmd --add-masquerade --zone=work
success
# 在 zone home 中添加转发
[root@localhost ~]# firewall-cmd --add-forward-port=port=8093:proto=tcp:toport=8080:toaddr=192.168.1.188 --zone=home
success
[root@localhost ~]# firewall-cmd --list-all --zone=home
home (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.1.188
  services: ssh mdns samba-client dhcpv6-client
  ports:
  protocols:
  masquerade: yes
  forward-ports: port=8093:proto=tcp:toport=8080:toaddr=192.168.1.188
  source-ports:
  icmp-blocks:
  rich rules:
```

　　从本机访问 `http://192.168.1.61:8093` 看结果：

![访问结果](https://img.lynchj.com/20190306114701.png)

* 继续在 `zone work` 中添加转发规则

```
$ firewall-cmd --add-forward-port=port=8093:proto=tcp:toport=8081:toaddr=192.168.1.188 --zone=work
success
$ firewall-cmd --list-all --zone=work
work (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth1
  sources:
  services: ssh dhcpv6-client
  ports:
  protocols:
  masquerade: yes
  forward-ports: port=8093:proto=tcp:toport=8081:toaddr=192.168.1.188
  source-ports:
  icmp-blocks:
  rich rules:
```

　　这里转发到的是我本地的一个并没有提供服务的端口 `8081`，再次从本机访问 `http://192.168.1.61:8093` 看结果：

![访问结果](https://img.lynchj.com/20190306114933.png)

　　已经凉了。。。。

>　　这里得出的结论是，会根据 source 和 interface 选择对应的 zone，经过顺序为 source、interface，但是后经过的规则会覆盖掉前面的规则。不对之处还请指出。

## service

　　服务就是把一组规则包装成模块。FirewallD 已经预先定义了一些服务，让客户更容易被允许或者被禁止进入服务。与对开放端口截然不同，使用预先定义服务或者用户自定服务，能够让管理更容易。

　　获取所有支持的服务：

```
$ firewall-cmd --get-services
RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync elasticsearch freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kibana klogin kpasswd kshell ldap ldaps libvirt libvirt-tls managesieve mdns mosh mountd ms-wbt mssql mysql nfs nfs3 nrpe ntp openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster quassel radius rpc-bind rsh rsyncd samba samba-client sane sip sips smtp smtp-submission smtps snmp snmptrap spideroak-lansync squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
```

　　查看服务相信信息：

```
$ firewall-cmd --info-service=ssh
ssh
  ports: 22/tcp
  protocols:
  source-ports:
  modules:
  destination:
```

## rich langulage rule

　　通过 `rich language` 语法，可以用比使用 `直接接口` （iptables 的那种配置方式）方式更易理解的方法建立复杂防火墙规则。此外，还能永久保留设置。这种语言使用关键字，是 iptables 工具的抽象表示。这种语言可以用来配置分区，也仍然支持现行的配置方式。

* 语法

```
$ firewall-cmd --add-rich-rule="rule [family='<rule family>'] [ source address='<address>' [invert='True'] ] [ destination address='<address>' [invert='True'] ] [ <element> ] [ log [prefix='<prefix text>'] [level='<log level>'] [limit value='rate/duration'] ] [ audit ] [ accept|reject|drop ]"
```

### 理解多规则命令

* Rule

```
rule [family="ipv4|ipv6"]
```

　　一个规则是关联某个特定分区的，一个分区可以有几个规则。如果几个规则互相影响或者冲突，则执行和数据包相匹配的第一个规则。如果提供了规则系列，它可以是 ipv4 或者 ipv6 。规则系列把规则限定在 IPv4 或 IPv6 。如果没有提供规则系列， 将为 IPv4 和 IPv6 增加规则。如果源地址或者目标地址在一个规则中被使用，那么必须提供规则系列。端口转发也存在这种情况。

* Source

```
source [not] address="address[/mask]"|mac="mac-address"|ipset="ipset"
```

　　通过指定源地址，一个尝试连接的源头可以被限制在源地址中。一个源地址或者地址范围是一个为 IPv4 或者 IPv6 地址。不支持使用主机名。可以通过增加 `invert="true"` 或 `invert="yes"` 来颠倒源地址命令的意思。

* Destination

```
destination [not] address="address[/mask]"
```

　　指定目的地址，与 source 相似，在服务器为多网卡的前提下。

* Service

```
service name="service name"
```

　　针对服务名的一种配置方式。服务名称是 `firewalld` 提供的其中一种服务。要获得被支持的服务的列表，输入以下命令： `firewall-cmd --get-services`。

* Port

```
port port="port value" protocol="tcp|udp"
```

　　端口既可以是一个独立端口数字，又或者端口范围，例如，5060-5062。协议可以指定为 tcp 或 udp 。

* Protocol

```
protocol value="protocol value"
```

　　协议值可以是一个协议 ID 数字，或者一个协议名。预设的一些可用协议，请查阅 `/etc/protocols`。

* Masquerade

```
masquerade
```

　　开启规则伪装，如果在一个配置规则里开启转发服务的话。

* Forward-Port

```
forward-port port="port value" protocol="tcp|udp" to-port="port value" to-addr="address"
```

　　从一个带有指定为 tcp 或 udp 协议的本地端口转发数据包到另一个本地端口，或另一台机器，或另一台机器上的另一个端口。 port 和 to-port 可以是一个单独的端口数字，或一个端口范围。而目的地址是一个简单的 IP 地址。forward-port 命令使用内部动作 accept（允许的）。

* Action

```
accept|reject|drop|mark
```

　　特殊使用方式：

```
accept [limit value="rate/duration"]
reject [type="reject type"] [limit value="rate/duration"]
drop [limit value="rate/duration"]
mark set="mark[/mask]" [limit value="rate/duration"]　
```
1. **accept：**允许指定规则。
2. **reject：**拒绝指定规则，给与拒绝响应。
3. **drop：**丢弃指定规则，不给任何响应。
4. **mark：**所有数据都将在 mangle 表的 PREROUTING 链中标记，并且带哟偶标记和掩码组合。可以使用 `limit` 参数限流。

# 常用命令收集

## 关于 zone

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --get-default-zone | 获取默认分区 |
| firewall-cmd --set-default-zone=`<zone>` | 设置默认分区。`<zone>`：分区名称，如：public、home |
| firewall-cmd --get-zones | 获取所有的分区名称 |
| firewall-cmd --get-active-zones | 获取活动的分区信息 |
| firewall-cmd --new-zone=`<zone>` | 创建一个分区 |
| firewall-cmd --delete-zone=`<zone>` | 删除一个分区 |
| firewall-cmd --zone=`<zone>` | 选择这个分区，同行命令后边指定端口、ip 等规则都会作用在这个分区上 |
| firewall-cmd --list-all | 列出指定分区的所有已添加或已启用的信息 |
| firewall-cmd --info-zone=`<zone>` | 查看指定分区的信息 |
| firewall-cmd --path-zone=`<zone>` --permanent | 查看分区对应的配置文件路径 |

### 配置文件

　　可以通过以下目录更改分区信息：

```
$ ll /usr/lib/firewalld/zones/
总用量 36
-rw-r--r--. 1 root root 299 4月  11 2018 block.xml
-rw-r--r--. 1 root root 293 4月  11 2018 dmz.xml
-rw-r--r--. 1 root root 291 4月  11 2018 drop.xml
-rw-r--r--. 1 root root 304 4月  11 2018 external.xml
-rw-r--r--. 1 root root 369 4月  11 2018 home.xml
-rw-r--r--. 1 root root 384 4月  11 2018 internal.xml
-rw-r--r--. 1 root root 315 4月  11 2018 public.xml
-rw-r--r--. 1 root root 162 4月  11 2018 trusted.xml
-rw-r--r--. 1 root root 311 4月  11 2018 work.xml
```

>  　　如果目录 `/etc/firewalld/zones` 下有同名文件，则 `/etc/firewalld/zones` 下文件生效。

## 关于 source

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --list-sources | 列出默认分区所有绑定的源 ip |
| firewall-cmd --add-source=`<source>` | 绑定源 ip 到默认分区 |
| firewall-cmd --remove-source=`<source>` | 从默认分区删除绑定的源 ip |
| firewall-cmd --query-source=`<source>` | 查询默认分区绑定的源 ip |

## 关于 interface（网卡）

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --list-interfaces | 获取当前区域所有接口 |
| firewall-cmd --get-zone-of-interface=`<interface>` | 根据接口名称获取其所在区域 |
| firewall-cmd --zone=`<zone>` --add-interface=`<interface>` | 把指定接口移动（添加）到指定分区。`--zone=<zone>`，不指定此参数将使用默认分区。`<interface>`：指定的接口（网卡）|
| firewall-cmd --change-interface=`<interface>` | 更改接口到当前区域 |
| firewall-cmd --query-interface=`<interface>` | 查询指定接口是否属当前区域 |
| firewall-cmd --remove-interface=`<interface>` | 从当前区域删除指定接口 |

## 关于 port

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --zone=public --list-ports | 获取指定分区暴露的端口信息 |
| firewall-cmd --zone=public --add-port=`<port>`/`<protocol>` | 添加暴露的端口规则。`<port>`：端口，如：8080、22、3306。`<protocol>`：协议，如：tcp、udp |
| firewall-cmd --zone=public --remove-port=`<port>`/`<protocol>` | 删除暴露的端口规则 |
| firewall-cmd --query-port=`<port>`/`<protocol>` | 查询指定端口/协议是否已经开启 |

## 关于服务

　　类似于端口的可视化服务，如果你添加 ssh 服务，那么就是开放了 22 端口。就是 firewalld 默认的把一些系统服务与端口对应了起来，开启服务，即为开启端口。

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --get-services | 获取支持的服务列表 |
| firewall-cmd --list-services | 列出当前 zone 提供的所有服务 |
| firewall-cmd --zone=home --add-service=ssh | 添加一个服务 |
| firewall-cmd --zone=home --remove-service=ssh | 删除一个服务 |
| firewall-cmd --query-service=ssh | 查询一个服务是否开启 |
| firewall-cmd --info-service=ssh | 查询一个服务对应的信息，如：端口号 |

### 配置文件

　　可以通过以下目录更改服务信息：

```
$ ll /usr/lib/firewalld/services/
总用量 432
-rw-r--r--. 1 root root 412 4月  11 2018 amanda-client.xml
-rw-r--r--. 1 root root 447 4月  11 2018 amanda-k5-client.xml
-rw-r--r--. 1 root root 320 4月  11 2018 bacula-client.xml
-rw-r--r--. 1 root root 346 4月  11 2018 bacula.xml
-rw-r--r--. 1 root root 275 4月  11 2018 bitcoin-rpc.xml
-rw-r--r--. 1 root root 307 4月  11 2018 bitcoin-testnet-rpc.xml
-rw-r--r--. 1 root root 281 4月  11 2018 bitcoin-testnet.xml
-rw-r--r--. 1 root root 244 4月  11 2018 bitcoin.xml
......
```

## 关于伪装 ip

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --zone=public --query-masquerade | 查询是否已经开启了伪装 ip。开启伪装 ip 才具把原有请求转发到其他服务器上的能力 |
| firewall-cmd --zone=external --add-masquerade | 开启伪装 ip |
| firewall-cmd --zone=external --remove-masquerade | 禁用伪装 ip |

>  　　如果目录 `/etc/firewalld/services` 下有同名文件，则 `/etc/firewalld/services` 下文件生效。

## 关于转发

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --list-forward-ports | 转发的规则列表 |
| firewall-cmd --add-forward-port=`port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]` | 添加转发规则，注意：`仅支持 IPV4`，需要开启 firewalld 伪装 ip 功能 |
| firewall-cmd --remove-forward-port=`port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]` | 移除转发规则，`仅支持 IPV4` |
| firewall-cmd --query-forward-port=`port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]` | 查询转发规则，`仅支持 IPV4` |

## 关于 rich language rule

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --add-rich-rule="rule family='ipv4' port protocol='`<protocol>`' port='`<port>`' accept" | 允许任意 ip 访问指定的端口，反之把 `accept` 换成 `reject` 即可 |
| firewall-cmd --add-rich-rule="rule family='ipv4' service name='`<service name>`' accept" | 允许任意 ip 访问指定的服务对应的端口（获取所有预设支持的服务信息：`firewall-cmd --get-services`），反之把 `accept` 换成 `reject` 即可 |
| firewall-cmd --add-rich-rule="rule family='ipv4' source address='`<ip>`' port protocol='`<protocol>`' port='`<port>`' accept" | 允许指定的源 ip 访问指定的端口，反之把 `accept` 换成 `reject` 即可 |
| firewall-cmd --add-rich-rule="rule family='ipv4' source address='`<ip>`' destination address='`<ip>`' port protocol='`<protocol>`' port='`<port>`' accept" | 允许指定的源 ip 访问指定 ip 的指定端口，反之把 `accept` 换成 `reject` 即可 |
| firewall-cmd --add-rich-rule="rule family='ipv4' forward-port port='`<port>`' protocol='`<protocol>`' to-port='`<port>`' to-addr='`<ip>`" | 转发请求你流量到另一台服务器。`记得开启伪装` |
| firewall-cmd --permanent --add-rich-rule='rule protocol value='`<protocol>`' reject' | 拒绝某种协议服务，如：icmp |

## 其他配置

| 命令 | 作用 | 
| --- | --- |
| firewall-cmd --reload | 重新加载配置，不会中断用户连接，不会丢失状态 |
| firewall-cmd --complete-reload | 重新加载配置，会中断用户连接，会丢失状态（只是在重启的时候丢失状态、中断用户连接，之后恢复正常）。通常在防火墙出现严重问题时，这个命令才会被使用。比如，防火墙规则是正确的，但却出现状态信息问题和无法建立连接。|
| firewall-cmd --panic-on | 终止所有输入和输出的数据包 |
| firewall-cmd --panic-off | 开启你所有输入和输出的数据包 |
| firewall-cmd --query-panic | 查询 panic 是否开启。`yes`：开启，`no`：未开启 |

# 特殊说明

* **--permanent**
　　为了让一个规则永久生效，需要向所有命令添加 `--permanent` 选项。注意，这不只是意味着永久更改，而且**更改将仅仅在防火墙重新加载、服务器重启或者系统重启之后生效**。用 `firewall-cmd` 更改规则如果没有加 `--permanent` 选项的设定能立即生效，但是它**仅仅在下次防火墙重新加载、系统启动或者 `firewalld` 服务重启之前可用**。防火墙不会在断开连接时重新加载，而会提示您通过重新加载，放弃临时更改。

* **--timeout=`<timeval>`**
　　在原有设置规则的基础上，最后还可以在加上这个参数，意指：这特规则在指定时间之后过期。
　　timeval 是一个数字（默认：秒）或在数字后跟一个字符 `s`（秒）、`m`（分钟）、`h`（小时）的数字，例如 `20m` 或 `1h`。
