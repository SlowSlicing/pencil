[toc]

# bridge 网络

## 容器之间的互通

　　这里首先启动两个容器，一个 name 为：`test1`，另一个 name 为：`test2`。如下：

```
[root@Docker1 ~]# docker run --name test1 -d busybox /bin/sh -c "while true; do sleep 60; done"
a7e7ecb9de2a8ef47d29630455fd6587602f9fa25dc7db25c70867aa15ebb5eb
[root@Docker1 ~]# docker run --name test2 -d busybox /bin/sh -c "while true; do sleep 60; done"
4da18e7bebd304c78237e1a38a07fca84b6c229344f98792c6c0a2629a68b1a8
[root@Docker1 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
4da18e7bebd3        busybox             "/bin/sh -c 'while t…"   5 seconds ago       Up 4 seconds                            test2
a7e7ecb9de2a        busybox             "/bin/sh -c 'while t…"   9 seconds ago       Up 8 seconds                            test1
```

　　在容器 `test1` 中执行命令，查看网卡信息：

```
[root@Docker1 ~]# docker exec test1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
38: eth0@if39: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

　　在容器 `test2` 中执行命令，查看网卡信息：

```
[root@Docker1 ~]# docker exec test2 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
40: eth0@if41: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

　　查看宿主机（装有 Docker 服务的机器）查看宿主机的网卡信息：

```
[root@Docker1 ~]# docker exec test1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
38: eth0@if39: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@Docker1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
    inet 10.211.55.9/24 brd 10.211.55.255 scope global noprefixroute dynamic eth0
       valid_lft 1149sec preferred_lft 1149sec
    inet6 fdb2:2c26:f4e4:0:2c63:a924:4f67:12b/64 scope global noprefixroute dynamic
       valid_lft 2588295sec preferred_lft 601095sec
    inet6 fe80::139f:3030:9bc7:ec26/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:80ff:fea4:a80e/64 scope link
       valid_lft forever preferred_lft forever
39: veth0075e8a@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 52:f3:a1:46:4d:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::50f3:a1ff:fe46:4da6/64 scope link
       valid_lft forever preferred_lft forever
41: vethd027e2f@if40: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 2e:be:16:55:ea:61 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::2cbe:16ff:fe55:ea61/64 scope link
       valid_lft forever preferred_lft forever
```

　　在没有启动两个容器之前，宿主机的网卡信息中是不存在最后边两个 `veth0075e8a` 和 `vethd027e2f` 的信息的。也可以看到每个容器的网卡信息：`eth0`，各自的ip `172.17.0.2/16` 和 `172.17.0.3/16`，它们的 Network Namesapce 是分开的。也是可以相互通信的，如下：

```
[root@Docker1 ~]# docker exec test1 ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.065 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.101 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.211 ms
64 bytes from 172.17.0.3: seq=3 ttl=64 time=0.166 ms

[root@Docker1 ~]# docker exec test2 ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.068 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.162 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.166 ms
```

　　结论显得很简单，下面就从 Liunx 的 Network Namespace 说说是如何做到通信的。

## Linux 的 Network Namespace

　　先查看下本机拥有的 Network Namespace（以下简称 `netns`）：

```
[root@Docker1 ~]# ip netns list
```

　　我当前的宿主机是没有 netns 的，这里先创建一个 `test1` 和 `test2`：

```
# 删除命令：ip netns delete test1
[root@Docker1 ~]# ip netns add test1
[root@Docker1 ~]# ip netns add test2
[root@Docker1 ~]# ip netns list
test2
test1
```

　　上一小节里面利用 Docker 创建了两个容器，并能查看其中的 netns 的信息，那么这里手动创建的 netns 如何查看？如下：

```
[root@Docker1 ~]# ip netns exec test1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

　　上面的命令意思：在 netns 名称为 test1 的上面执行命令 `ip a`。可以看到，只有一个本地会换地址。没有任何地址，状态为 `DOWN`。当然，你也可以执行其他的命令，如：`ip link`。

```
[root@Docker1 ~]# ip netns exec test1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

　　这里的 lo 网卡状态是 `DOWN`，这里把它给 `UP` 起来。

```
[root@Docker1 ~]# ip netns exec test1 ip link set dev lo up
[root@Docker1 ~]# ip netns exec test1 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

　　这里执行了 `UP` 操作之后它的状态依旧不是 UP，而是 `UNKNOWN`，是因为网络连接是多方面的，单单只是这个 netns 的 Veth UP 是没有用的，没人跟它连接。也就是说，单个网卡没办法 UP 起来。

　　现在的情况是这样的：

![独立 netns](http://img.lynchj.com/03742369565644d8a337bf3daf8ead04.png)

　　这里呢，想要做的是让这俩个独立的 netns 连接起来，能够通信。如果想让两台计算机连接起来，我们知道，需要网线，让它们可以进行通信。那么在 Linux 的 netns 中，有一个概念叫做 `Veth`，我们可以创建两个 Veth，并分别分给 test1 和 test2，这样他们就可以连接起来，如下图：

![连接](http://img.lynchj.com/44612de16b78492eb7ee727d4a555433.png)

　　如下操作，首先我查看当前宿主机的只存在五个 link，然后添加了两个类型为 `veth` 的 `link`，再次查看，发现存在两个个 link，`veth-test1` AND `veth-test2`。他们有个特点，没有 ip，状态也都是 DOWN。

```
[root@Docker1 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
39: veth0075e8a@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 52:f3:a1:46:4d:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
41: vethd027e2f@if40: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 2e:be:16:55:ea:61 brd ff:ff:ff:ff:ff:ff link-netnsid 1
[root@Docker1 ~]# ip link add veth-test1 type veth peer name veth-test2
[root@Docker1 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
39: veth0075e8a@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 52:f3:a1:46:4d:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
41: vethd027e2f@if40: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 2e:be:16:55:ea:61 brd ff:ff:ff:ff:ff:ff link-netnsid 1
42: veth-test2@veth-test1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3a:4a:97:ec:ee:5a brd ff:ff:ff:ff:ff:ff
43: veth-test1@veth-test2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 32:62:53:b3:f1:46 brd ff:ff:ff:ff:ff:ff
```

　　现在把新增的 Veth `veth-test1` 添加到 netns `test1` 中去（netns `test2` 同样操作），这时候在宿主机中就只剩下五个，并且，查看 netns test1 和 test2 的 link 可以发现，在宿主机"消失"的 Veth 都已经跑到它们里面去了，如下：

```
[root@Docker1 ~]# ip link set veth-test1 netns test1
[root@Docker1 ~]# ip link set veth-test2 netns test2
[root@Docker1 ~]# ip netns exec test1 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
43: veth-test1@if42: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 32:62:53:b3:f1:46 brd ff:ff:ff:ff:ff:ff link-netnsid 1
[root@Docker1 ~]# ip netns exec test2 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
42: veth-test2@if43: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3a:4a:97:ec:ee:5a brd ff:ff:ff:ff:ff:ff link-netnsid 0
[root@Docker1 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
39: veth0075e8a@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 52:f3:a1:46:4d:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
41: vethd027e2f@if40: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether 2e:be:16:55:ea:61 brd ff:ff:ff:ff:ff:ff link-netnsid 1
```

　　这样我们就实现了上面图中的，通过 Veth 进行连接通信的第一步了。可以看到，已经分配到 netns 中的 Veth 仅仅只有 Mac 地址，并不存在 IP 地址，而且状态也是 DOWN 的。

　　下面我们手动添加上 IP 地址：

```
[root@Docker1 ~]# ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
[root@Docker1 ~]# ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
[root@Docker1 ~]# ip netns exec test1 ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
43: veth-test1@if42: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 32:62:53:b3:f1:46 brd ff:ff:ff:ff:ff:ff link-netnsid 1
[root@Docker1 ~]# ip netns exec test2 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
42: veth-test2@if43: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3a:4a:97:ec:ee:5a brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

　　添加上 IP 地址还不算完，还要把对应的 Veth 给 UP 了。如下：

```
[root@Docker1 ~]# ip netns exec test1 ip link set dev veth-test1 up
[root@Docker1 ~]# ip netns exec test2 ip link set dev veth-test2 up
[root@Docker1 ~]# ip netns exec test1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
43: veth-test1@if42: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 32:62:53:b3:f1:46 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 192.168.1.1/24 scope global veth-test1
       valid_lft forever preferred_lft forever
    inet6 fe80::3062:53ff:feb3:f146/64 scope link
       valid_lft forever preferred_lft forever
[root@Docker1 ~]# ip netns exec test2 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
42: veth-test2@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 3a:4a:97:ec:ee:5a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.2/24 scope global veth-test2
       valid_lft forever preferred_lft forever
    inet6 fe80::384a:97ff:feec:ee5a/64 scope link
       valid_lft forever preferred_lft forever
```

　　此时发现已经存在 IP 地址，并且状态也已经是 UP 了，那么我们从 netns 中 ping 一下试试。

```
[root@Docker1 ~]# ip netns exec test1 ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.066 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.070 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.038 ms
^C
--- 192.168.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.038/0.058/0.070/0.014 ms
[root@Docker1 ~]# ip netns exec test2 ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.033 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.072 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.045 ms
^C
--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.033/0.050/0.072/0.016 ms
```

　　到这里，已经把下图的这种通信方式给实现了。这里是使用 Linux 的 netns 做的一个实验，最终达到通信的目的。和文章一开始通过 `docker run` 运行起来的两个 `busybox` 容器可以相互通信的道理是基本一样的。

![连接](http://img.lynchj.com/0ec0bfc602ff452a906da5c4f637a443.png)

## bridge

　　前面两小节是为了铺垫，主要普及一下 netns 的一些基本原理。这里再开说之前，先抛出两个疑问。

* 两个容器之间是怎么通信的？
* 容器又是怎么访问互联网的？

　　先看一下 Docker 具有哪些网络：

```
[root@Docker1 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
664f65044a2f        bridge              bridge              local
2596b17410cc        host                host                local
3accb13c9e4c        none                null                local
```

　　这里要说的是 bridge，它也是 Docker 中最重要的网络了。一开始启动了两个 busybox 的容器，这里为了方便，先停掉一个：

```
[root@Docker1 ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
4da18e7bebd3        busybox             "/bin/sh -c 'while t…"   2 hours ago         Up 2 hours                              test2
a7e7ecb9de2a        busybox             "/bin/sh -c 'while t…"   2 hours ago         Up 2 hours                              test1
[root@Docker1 ~]# docker stop test2
```

　　从上文已经知道，每一个容器都是新增一个 netns，也知道单个的 Veth 是没办法正常使用的，那么容器里的 Veth 是连接到哪里了呢？连接到 Docker 的 `bridge` 上了，**注意：这里说的是 `NAME = bridge` 的的网络上，因为自己也可以定义类型为 `bridge` 的网络，名称你可以叫 demo1、demo2 等。**可以通过下面的命令查看到：

```
[root@Docker1 ~]# docker network inspect 664f65044a2f
[
    {
        "Name": "bridge",
        "Id": "664f65044a2ff1f59413d02db2ec032cf6c0e1d759a0ca497d02c4c8dd4ef619",
        "Created": "2018-11-08T14:58:57.971424978+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "a7e7ecb9de2a8ef47d29630455fd6587602f9fa25dc7db25c70867aa15ebb5eb": {
                "Name": "test1",
                "EndpointID": "c1c530262d07673bbe69b7780e23d2e83b13bea7882da1a2e65d8076e63692f2",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

　　仔细看 `Containers` 项，会发现有 test1 的信息。说明 test1 这个 Container 是连接到了 `NETWORK ID` 为 `664f65044a2f` 的网络上面，而 `NETWORK ID` 为 `664f65044a2f` 的网络就是 `NAME = bridge` 的网络了。这里就有个问题，这个 bridge 是什么？在哪里？

```
[root@Docker1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
    inet 10.211.55.9/24 brd 10.211.55.255 scope global noprefixroute dynamic eth0
       valid_lft 1506sec preferred_lft 1506sec
    inet6 fdb2:2c26:f4e4:0:2c63:a924:4f67:12b/64 scope global noprefixroute dynamic
       valid_lft 2581512sec preferred_lft 594312sec
    inet6 fe80::139f:3030:9bc7:ec26/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:80ff:fea4:a80e/64 scope link
       valid_lft forever preferred_lft forever
39: veth0075e8a@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 52:f3:a1:46:4d:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::50f3:a1ff:fe46:4da6/64 scope link
       valid_lft forever preferred_lft forever
```

　　上面的命令可以看到，在宿主机中有一个接口 `docker0`，这就是刚才所说的 bridge 了，那我们 Container 中的 Veth 有是怎么连接上去的呢？注意看 `docker0` 下面的 `veth0075e8a@if38`，它也是一个 Veth，一个属于我们宿主机所在 netns 的 Veth，上面已经知道，一对 Veth 可以构成通信环境，查看一下 Container 中的 Veth 信息：

```
[root@Docker1 ~]# docker exec test1 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
38: eth0@if39: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

　　那么这里构成通信环境的两个 Veth 就是宿主机的 `veth0075e8a` 接口和 Container 的 `eth0` 接口，最终连接到宿主机上面，具体的就是连接到 `docker0` 上面。这里可以通过一个工具来帮我们验证一下为什么说 `veth0075e8a` 最终是连接到 `docker0` 上面，`brctl`，没有的话先通过 yum 进行安装，`yum -y install bridge-utils`。

```
[root@Docker1 ~]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024280a4a80e	no		veth0075e8a
```

　　这里可以看到，`veth0075e8a` 确实是连接到了 `docker0` 上。我们可以把原来那个停掉的 Container test2 给启动起来：

```
[root@Docker1 ~]# docker start test2
test2

[root@Docker1 ~]# docker network inspect 664f65044a2f
[
    {
        "Name": "bridge",
        "Id": "664f65044a2ff1f59413d02db2ec032cf6c0e1d759a0ca497d02c4c8dd4ef619",
        "Created": "2018-11-08T14:58:57.971424978+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4da18e7bebd304c78237e1a38a07fca84b6c229344f98792c6c0a2629a68b1a8": {
                "Name": "test2",
                "EndpointID": "500c6ef02b4c8cd61ee00461513bbb80fe6ea3985db239f93cde09a2a85a2447",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "a7e7ecb9de2a8ef47d29630455fd6587602f9fa25dc7db25c70867aa15ebb5eb": {
                "Name": "test1",
                "EndpointID": "c1c530262d07673bbe69b7780e23d2e83b13bea7882da1a2e65d8076e63692f2",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

　　这里起来以后首先查看了一下 Docker bridge 网络的信息，可以看到 Containers 项的网络信息。在宿主机查看网络信息发现：

```
[root@Docker1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
    inet 10.211.55.9/24 brd 10.211.55.255 scope global noprefixroute dynamic eth0
       valid_lft 1403sec preferred_lft 1403sec
    inet6 fdb2:2c26:f4e4:0:2c63:a924:4f67:12b/64 scope global noprefixroute dynamic
       valid_lft 2580649sec preferred_lft 593449sec
    inet6 fe80::139f:3030:9bc7:ec26/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:80ff:fea4:a80e/64 scope link
       valid_lft forever preferred_lft forever
39: veth0075e8a@if38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 52:f3:a1:46:4d:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::50f3:a1ff:fe46:4da6/64 scope link
       valid_lft forever preferred_lft forever
45: veth93f79d1@if44: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 2e:19:19:09:d2:29 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::2c19:19ff:fe09:d229/64 scope link
       valid_lft forever preferred_lft forever
```

　　发现多了一个 Veth `veth93f79d1`，这个就是和 Container test2 进行通信的 Veth 了。接下来使用工具 `brctl` 查看：

```
[root@Docker1 ~]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024280a4a80e	no		veth0075e8a
							veth93f79d1
```

　　到这里，可以用一张图来表示一下：

![容器之间通信](http://img.lynchj.com/8e52b2ef9ba54bf380e53e3b7a98838c.png)

　　这张图就很类似于我们现实中的网络，两台电脑通信，他们可以都连接到一个交换机/路由器上面，然后就可以进行通信了。到这里，已经解决了这小节提出的第一个问题**两个容器之间是怎么通信的。**

　　再解释第二个问题之前，先来看一张图：

![容器访问外网](http://img.lynchj.com/66e6870e2eb247648f797bb83123b688.png)

　　这里先抛开容器，作为宿主机的 Liunx 它本身是可以访问外网的，通过 eth0 接口进行访问。容器可以访问到 Docker0，这里使用的全部都是在 netns 中间进行通信的 Veth，而内部 IP 想要访问到外网，并且外网能够对内网的 IP 进行响应就需要进行 `NAT 网络地址转换`，包装一层信息，这个工作是由 iptables 完成的。这样，就可以达到访问外网的目的了。到这里第二个问题**容器又是怎么访问互联网的**也搞定了。

# none 网络

　　`none` 网络比较简单，这里已经把上节中的 Container test1 和 test2 删除掉了，重新启动一个 test1 Container 并指定网络为 none。

```
[root@Docker1 ~]# docker run --name test1 -d --network none busybox /bin/sh -c "while true; do sleep 60; done"
8df85a4e16a6cee04faf2cb480dd3ea044d94332fdc5fe25d19d986e8c790810
```

　　进入容器查看网络信息，发现只有一个本地回环地址，那么就说明这个属于 none 网络的 Container 是独立的，无法与其他网络进行通信的。

```
[root@Docker1 ~]# docker exec -it 8df85a4e16a6cee04faf2cb480dd3ea044d94332fdc5fe25d19d986e8c790810 /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

　　也可以查看一下 none 的网络信息：

```
[root@Docker1 ~]# docker network inspect none
[
    {
        "Name": "none",
        "Id": "3accb13c9e4c2c957c67fe6bc53065a2f32117c2f00949834c51f35e6f5fb22c",
        "Created": "2018-09-01T16:26:21.207367782+08:00",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "8df85a4e16a6cee04faf2cb480dd3ea044d94332fdc5fe25d19d986e8c790810": {
                "Name": "test1",
                "EndpointID": "6665b31036f8068b17956093045d303de19ad78c58c2b55f4bb2d4a1776a7819",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

# host 网络

　　同样的，删除掉上一届的 test1 Container。重新启动一个连接到 host 网络的 Container。

```
[root@Docker1 ~]# docker run --name test1 -d --network host busybox /bin/sh -c "while true; do sleep 60; done"
361b2f79a22a5ad1696cb2e54110c191ae88b5e8caf9fb0b164c1de99df62ca5
```

　　查看 host 网络：

```
[root@Docker1 ~]# docker network inspect 2596b17410cc
[
    {
        "Name": "host",
        "Id": "2596b17410ccaa4191ed459e3d4f8f8fc06919cfa0433e50ce8d79ed6e079fdf",
        "Created": "2018-09-01T16:26:21.213958845+08:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "361b2f79a22a5ad1696cb2e54110c191ae88b5e8caf9fb0b164c1de99df62ca5": {
                "Name": "test1",
                "EndpointID": "fa49dd211aa42b6b9a7aa39a326617ccfe259dddd287abae7b70d78eabc6634b",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

　　这里看到的 Container 网络信息也是空的，不要着急，先进入 Container 查看下网络信息：

```
[root@Docker1 ~]# docker exec -it test1 /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
    inet 10.211.55.9/24 brd 10.211.55.255 scope global dynamic eth0
       valid_lft 1249sec preferred_lft 1249sec
    inet6 fdb2:2c26:f4e4:0:2c63:a924:4f67:12b/64 scope global dynamic
       valid_lft 2575090sec preferred_lft 587890sec
    inet6 fe80::139f:3030:9bc7:ec26/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:80ff:fea4:a80e/64 scope link
       valid_lft forever preferred_lft forever
55: veth2aa69e1@if54: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue master docker0
    link/ether 2a:ec:a8:72:1c:49 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::28ec:a8ff:fe72:1c49/64 scope link
       valid_lft forever preferred_lft forever
```

　　退出容器，查看下宿主机的网络信息：


```
/ # exit
[root@Docker1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1c:42:9c:52:cd brd ff:ff:ff:ff:ff:ff
    inet 10.211.55.9/24 brd 10.211.55.255 scope global noprefixroute dynamic eth0
       valid_lft 1221sec preferred_lft 1221sec
    inet6 fdb2:2c26:f4e4:0:2c63:a924:4f67:12b/64 scope global noprefixroute dynamic
       valid_lft 2575063sec preferred_lft 587863sec
    inet6 fe80::139f:3030:9bc7:ec26/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:80:a4:a8:0e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:80ff:fea4:a80e/64 scope link
       valid_lft forever preferred_lft forever
55: veth2aa69e1@if54: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
    link/ether 2a:ec:a8:72:1c:49 brd ff:ff:ff:ff:ff:ff link-netnsid 4
    inet6 fe80::28ec:a8ff:fe72:1c49/64 scope link
       valid_lft forever preferred_lft forever
```

　　对的，没错，是一样的。共同用统一网络信息，这种情况下有一个坏处，那就是你不能启动两个同样的容易，端口号会冲突，这是因为现在使用的都是宿主机的网络环境。比如 Nginx Image，启动第一个的时候没事，正常运行，启动第二个的时候就会报错了。如下：

```
# 前台运行启动
[root@Docker1 ~]# docker run --name web1 --network host nginx

# 切换个窗口，再启动一次一个 name = web2 的
[root@Docker1 ~]# docker run --name web2 --network host nginx
2018/11/08 20:14:09 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/11/08 20:14:09 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/11/08 20:14:09 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/11/08 20:14:09 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/11/08 20:14:09 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
2018/11/08 20:14:09 [emerg] 1#1: still could not bind()
nginx: [emerg] still could not bind()
```
