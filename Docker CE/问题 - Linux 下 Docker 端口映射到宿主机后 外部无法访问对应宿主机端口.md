[TOC]

### 问题描述

&emsp;&emsp;前段时间使用 Docker 装了 GitLab，SSH 配置都已经配置完毕，容器端口和宿主机端口也映射完毕。Firewall 和 SELinux 也已经关闭。

&emsp;&emsp;1、在宿主机上访问对应的端口使用 SSH 拉取 GitLab 上的代码**正常**
&emsp;&emsp;2、在容器中访问容器中对应 SSH 的端口**正常**
&emsp;&emsp;3、在外部网络访问 Docker 宿主机的对应端口使用 SSH 拉取代码`异常`

### 原因

&emsp;&emsp;**这是因为我的宿主机没有开启 `ip 转发`功能，导致外部网络访问宿主机对应端口是没能转发到 Docker Container 所对应的端口上。**

### 解决问题

&emsp;&emsp;这里记录一下：Linux 发行版默认情况下是不开启 `ip 转发`功能的。这是一个好的做法，因为大多数人是用不到 ip 转发的，但是如果我们架设一个 Linux 路由或者 VPN 服务我们就需要开启该服务了。

&emsp;&emsp;在 Linux 中开启 ip 转发的内核参数为：`net.ipv4.ip_forward`，查看是否开启 ip转发：

```
cat /proc/sys/net/ipv4/ip_forward	# 0：未开启，1：已开启
```

* 修改 `net.ipv4.ip_forward` 的值：

```
sysctl -w net.ipv4.ip_forward=1
# 或者
echo 1 > /proc/sys/net/ipv4/ip_forward
```

> &emsp;&emsp;上面的这种方式无需重启，不过也只能算是`临时生效`。它的效果会随着计算机的重启而失效。

* 永久生效的 ip 转发

```
vim /etc/sysctl.conf # 在此文件中新增 "net.ipv4.ip_forward = 1"，保存退出
# 立即生效
sysctl -p /etc/sysctl.conf
# 在红帽发行版的 Linux 系统中也可以通过重启网卡来立即生效
service network restart # CentOS 6
systemctl restart network # CentOS 7
# 而在 debian/ubuntu 系列的发行版则用
/etc/init.d/procps.sh restart
```
