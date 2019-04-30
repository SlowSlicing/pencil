> 　　此处只是做个笔记，正常工作中都在使用`firewalld`

　　在早期的Linux 系统中，默认使用的是`iptables` 防火墙管理服务来配置防火墙。尽管新型的`firewalld` 防火墙管理服务已经被投入使用多年，但是大量的企业在生产环境中依然出于各种原因而继续使用iptables。考虑到iptables 在当前生产环境中还具有顽强的生命力，所以在此做下复习，其实各个防火墙工具的配置思路是一致的，在掌握了iptables 后再学习其他防火墙管理工具时，也有借鉴意义。

#### 策略与规则链

　　防火墙会从上至下的顺序来读取配置的策略规则，在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果在读取完所有的策略规则之后没有匹配项，就去执行默认的策略。一般而言，防火墙策略规则的设置有两种：一种是`通`（即放行），一种是`堵`（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许时，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。

　　iptables 服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

　　**在进行路由选择前处理数据包（PREROUTING）；**
　　**处理流入的数据包（INPUT）；**
　　**处理流出的数据包（OUTPUT）；**
　　**处理转发的数据包（FORWARD）；**
　　**在进行路由选择后处理数据包（POSTROUTING）。**

　　一般来说，从内网向外网发送的流量一般都是可控且良性的，因此我们使用最多的就是`INPUT 规则链`，该规则链可以增大黑客人员从外网入侵内网的难度。

　　比如在您居住的社区内，物业管理公司有两条规定：禁止小商小贩进入社区；各种车辆在进入社区时都要登记。显而易见，这两条规定应该是用于社区的正门的（流量必须经过的地方），而不是每家每户的防盗门上。根据前面提到的防火墙策略的匹配顺序，可能会存在多种情况。比如，来访人员是小商小贩，则直接会被物业公司的保安拒之门外，也就无需再对车辆进行登记。如果来访人员乘坐一辆汽车进入社区正门，则`禁止小商小贩进入社区`的第一条规则就没有被匹配到，因此按照顺序匹配第二条策略，即需要对车辆进行登记。如果是社区居民要进入正门，则这两条规定都不会匹配到，因此会执行默认的放行策略。

　　但是，仅有策略规则还不能保证社区的安全，保安还应该知道采用什么样的动作来处理这些匹配的流量，比如`允许`、`拒绝`、`登记`、`不理它`。这些动作对应到`iptables 服务`的术语中分别是`ACCEPT`（允许流量通过）、`REJECT`（拒绝流量通过）、`LOG`（记录日志信息）、`DROP`（拒绝流量通过）。`允许流量通过`和`记录日志信息`都比较好理解，这里需要着重说说的是`REJECT` 和`DROP` 的不同点。就DROP 来说，它是直接`将流量丢弃而且不响应`；REJECT 则会在`拒绝流量后再回复一条“您的信息已经收到，但是被扔掉了”信息`，从而让流量发送方清晰地看到数据被拒绝的响应信息。

　　我们来举一个例子。比如有一天您正在家里看电视，突然听到有人敲门，您透过防盗门的猫眼一看是推销商品的，便会在不需要的情况下开门并拒绝他们（REJECT）。但如果您看到的是债主带了十几个小弟来讨债，此时不仅要拒绝开门，还要默不作声，伪装成自己不在家的样子（DROP）。

　　当把Linux 系统中的防火墙策略设置为`REJECT` 拒绝动作后，流量发送方会看到端口不可达的响应：

```
[root@lynchj ~]# ping -c 4 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
From 192.168.10.10 icmp_seq=1 Destination Port Unreachable
From 192.168.10.10 icmp_seq=2 Destination Port Unreachable
From 192.168.10.10 icmp_seq=3 Destination Port Unreachable
From 192.168.10.10 icmp_seq=4 Destination Port Unreachable
--- 192.168.10.10 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3002ms
```

　　而把Linux 系统中的防火墙策略修改成`DROP` 拒绝动作后，流量发送方会看到响应超时的提醒。但是流量发送方无法判断流量是被拒绝，还是接收方主机当前不在线：

```
[root@linuxprobe ~]# ping -c 4 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
--- 192.168.10.10 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3000ms
```

#### iptables 中基本的命令参数

　　`iptables` 是一款`基于命令行的防火墙策略管理工具`，具有`大量参数`，`学习难度较大`。好在对于日常的防火墙策略配置来讲，大家无需深入了解诸如`四表五链`的理论概念，只需要掌握常用的参数并做到灵活搭配即可，这就足以应对日常工作了。

　　`iptables 命令`可以根据流量的`源地址`、`目的地址`、`传输协议`、`服务`类型等信息进行匹配，一旦匹配成功，`iptables` 就会根据策略规则所预设的动作来处理这些流量。另外，再次提醒一下，防火墙策略规则的匹配顺序是从上至下的，因此要把较为严格、优先级较高的策略规则放到前面，以免发生错误。下表总结归纳了常用的`iptables 命令参数`。再次强调，无需死记硬背这些参数，只需借助下面的实验来理解掌握即可。

| 参数 | 作用 |
| --- | --- | 
| -P | 设置默认策略 |
| -F | 清空规则链 |
| -L | 查看规则链 |
| -A | 在规则链的末尾加入新规则 |
| -I | num 在规则链的头部加入新规则 |
| -D | num 删除某一条规则 |
| -s | 匹配来源地址IP/MASK，加叹号“!”表示除这个IP 外 |
| -d | 匹配目标地址 |
| -i | 网卡名称 匹配从这块网卡流入的数据 |
| -o | 网卡名称 匹配从这块网卡流出的数据 |
| -p | 匹配协议，如TCP、UDP、ICMP |
| `--dport num` | 匹配目标端口号 |
| `--sport num` | 匹配来源端口号 |


　　**在`iptables命令`后添加`-L参数`查看已有的防火墙规则链：**

```
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
ACCEPT all -- anywhere anywhere ctstate RELATED,ESTABLISHED
ACCEPT all -- anywhere anywhere
INPUT_direct all -- anywhere anywhere
INPUT_ZONES_SOURCE all -- anywhere anywhere
INPUT_ZONES all -- anywhere anywhere
ACCEPT icmp -- anywhere anywhere
REJECT all -- anywhere anywhere reject-with icmp-host-prohibited
………………省略部分输出信息………………
```

　　**在`iptables命令`后添加`-F参数`清空已有的防火墙规则链：**

```
[root@lynchj ~]# iptables -F
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
Chain FORWARD (policy ACCEPT)
target prot opt source destination
Chain OUTPUT (policy ACCEPT)
target prot opt source destination
………………省略部分输出信息………………
```

　　**把`INPUT规则链`的默认策略设置为拒绝：**

```
[root@lynchj ~]# iptables -P INPUT DROP
[root@lynchj ~]# iptables -L
Chain INPUT (policy DROP)
target prot opt source destination
…………省略部分输出信息………………
```


　　前文提到，防火墙策略规则的设置有两种：`通和堵`。当把`INPUT 链`设置为默认拒绝后，就要在防火墙策略中写入允许策略了，否则所有到来的流量都会被拒绝掉。另外，需要注意的是，`规则链的默认拒绝动作只能是DROP，而不能是REJECT`。

　　**向INPUT链中添加允许ICMP流量进入的策略规则：**

　　在日常运维工作中，经常会使用`ping 命令`来检查对方主机是否在线，而向防火墙的INPUT 规则链中添加一条允许`ICMP 流量`进入的策略规则就默认允许了这种ping 命令检测行为。

```
[root@lynchj ~]# iptables -I INPUT -p icmp -j ACCEPT
[root@lynchj ~]# ping -c 4 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.156 ms
64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.117 ms
64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.099 ms
64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.090 ms
--- 192.168.10.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.090/0.115/0.156/0.027 ms
```

　　**删除INPUT规则链中刚刚加入的那条策略（允许ICMP流量），并把默认策略设置为允许：**

```
[root@lynchj ~]# iptables -D INPUT 1
[root@lynchj ~]# iptables -P INPUT ACCEPT
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
………………省略部分输出信息………………
```

　　**将INPUT规则链设置为只允许指定网段的主机访问本机的22端口，拒绝来自其他所有主机的流量：**

```
[root@lynchj ~]# iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j
ACCEPT
[root@lynchj ~]# iptables -A INPUT -p tcp --dport 22 -j REJECT
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

　　再次重申，防火墙策略规则是按照从上到下的顺序匹配的，因此一定要把允许动作放到拒绝动作前面，否则所有的流量就将被拒绝掉，从而导致任何主机都无法访问我们的服务。另外，这里提到的22 号端口是ssh 服务使用的。

　　在设置完上述`INPUT 规则链`之后，我们使用IP 地址在192.168.10.0/24 网段内的主机访问服务器（即前面提到的设置了INPUT 规则链的主机）的22 端口，效果如下：

```
[root@Client A ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is 70:3b:5d:37:96:7b:2e:a5:28:0d:7e:dc:47:6a:fe:5c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 此处输入对方主机的root 管理员密码
Last login: Sun Feb 12 01:50:25 2017
[root@Client A ~]#
```

　　然后，我们再使用IP 地址在192.168.20.0/24 网段内的主机访问服务器的22 端口（虽网段不同，但已确认可以相互通信），效果如下，就会提示连接请求被拒绝了（Connection failed）：

```
[root@Client B ~]# ssh 192.168.10.10
Connecting to 192.168.10.10:22...
Could not connect to '192.168.10.10' (port 22): Connection failed.
```

　　**向`INPUT规则链`中添加拒绝所有人访问本机12345端口的策略规则：**

```
[root@lynchj ~]# iptables -I INPUT -p tcp --dport 12345 -j REJECT
[root@lynchj ~]# iptables -I INPUT -p udp --dport 12345 -j REJECT
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable
REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable
ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

　　**向`INPUT规则链`中添加拒绝192.168.10.5主机访问本机80端口（ Web服务）的策略规则：**

```
[root@lynchj ~]# iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
REJECT tcp -- 192.168.10.5 anywhere tcp dpt:http reject-with icmp-port-unreachable
REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable
REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable
ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

　　**向INPUT规则链中添加拒绝所有主机访问本机1000〜1024端口的策略规则：**

```
[root@lynchj ~]# iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT
[root@lynchj ~]# iptables -A INPUT -p udp --dport 1000:1024 -j REJECT
[root@lynchj ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
REJECT tcp -- 192.168.10.5 anywhere tcp dpt:http reject-with icmp-port-unreachable
REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable
REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable
ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
REJECT tcp -- anywhere anywhere tcp dpts:cadlock2:1024 reject-with icmp-portunreachable
REJECT udp -- anywhere anywhere udp dpts:cadlock2:1024 reject-with icmp-portunreachable
………………省略部分输出信息………………
```

　　有关`iptables 命令`的知识记录到此就结束了。

> 　　请特别注意，使用`iptables 命令`配置的防火墙规则默认会在系统下一次重启时失效，如果想让配置的防火墙策略永久生效，还要执行保存命令：

```
[root@lynchj ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [ OK ]
```
