Linux
RHEL 7,命令
[toc]


## 常用系统工作命令

---

### 1.echo 命令


&emsp;&emsp;`echo` 命令用于在终端输出字符串或变量提取后的值，格式为`echo [字符串 | $变量]`。例如，把指定字符串“测试”输出到终端屏幕的命令为：
```
[root@lynchj tmp]# echo "测试"
测试
```
| 命令 | 说明 | 样例 |
| --- | --- | --- |
| echo 内容 | 打印输入的内容 | echo "测试" |
| echo 变量 | 打印变量信息 | echo $SHELL |
| echo 内容 > text.txt | 新建文件，并输入内容。如果文件已存在，则覆盖文件内容 | echo "新建文件" > text.txt |
| echo 内容 >> text.txt | 新启一行，追加内容 | echo "追加内容" >> text.txt |

### 2.date 命令

&emsp;&emsp;`date` 命令用于显示及设置系统的时间或日期，格式为`date [选项] [+指定的格式]`。
&emsp;&emsp;只需在强大的 `date` 命令中输入以`+`号开头的参数，即可按照指定格式来输出系统的时间或日期，这样在日常工作时便可以把备份数据的命令与指定格式输出的时间信息结合到一起。例如，把打包后的文件自动按照`年-月-日`的格式打包成`backup-2017-9-1.tar.gz`，用户只需要看一眼文件名称就能大概了解到每个文件的备份时间了。`date` 命令常见的参数格式及作用如下所示：
| 参数 | 作用 |
| --- | --- |
| %t | 跳格[Tab 键] |
| %Y | 年（2017、2018） |
| %m | 月（1～12）|
| %d | 日（1～31）|
| %H | 小时（00～23）|
| %M | 分钟（00～59）|
| %S | 秒（00～59）|
| %I | 小时（00～12）|
| %j | 今年中的第几天 |

&emsp;&emsp;按照默认格式查看当前系统时间的date 命令如下所示：
```
[root@lynchj tmp]# date
Mon May 14 13:35:26 CST 2018
```
&emsp;&emsp;按照“年-月-日 小时:分钟:秒”的格式查看当前系统时间的date 命令如下所示：
```
[root@lynchj tmp]# date "+%Y-%m-%d %H:%M:%S"
2018-05-14 13:35:56
```
&emsp;&emsp;将系统的当前时间设置为2017 年9 月1 日8 点30 分的date 命令如下所示：
```
[root@lynchj tmp]# date -s "20180514 13:37:00"
Mon May 14 13:37:00 CST 2018
```

### 3.reboot 命令

&emsp;&emsp;`reboot` 命令用于重启系统，其格式为`reboot`。
&emsp;&emsp;由于重启计算机这种操作会涉及硬件资源的管理权限，因此默认只能使用 `root 管理员` 来重启，其命令如下：
```
[root@lynchj tmp]# reboot
```
### 4.poweroff 命令

&emsp;&emsp;`poweroff `命令用于关闭系统，其格式为 `poweroff`。
&emsp;&emsp;该命令与 `reboot` 命令相同，都会涉及硬件资源的管理权限，因此默认只有 `root 管理员` 才可以关闭电脑，其命令如下：
```
[root@lynchj tmp]# poweroff
```

### 5.wget 命令

&emsp;&emsp;`wget` 命令用于在终端中下载网络文件，格式为`wget [参数] 下载地址`。
&emsp;&emsp;如果您没有Linux 系统的管理经验，当前只需了解一下`wget`命令的`参数`以及作用，然后看一下下面的演示实验即可，切记不要急于求成。以下为简单命令参数说明：

| 参数 | 作用 |
| --- | --- |
| -b | 后台下载模式 |
| -P | 下载到指定目录 |
| -t | 最大尝试次数 |
| -c | 断点续传 |
| -p | 下载页面内所有资源，包括图片、视频等 |
| -r | 递归下载 |

&emsp;&emsp;下面以下载Notepad++为例：
```
[root@lynchj tmp]# wget -c https://notepad-plus-plus.org/repository/7.x/7.5.6/npp.7.5.6.Installer.exe
--2018-05-14 13:53:29--  https://notepad-plus-plus.org/repository/7.x/7.5.6/npp.7.5.6.Installer.exe
Resolving notepad-plus-plus.org (notepad-plus-plus.org)... 37.59.28.236
Connecting to notepad-plus-plus.org (notepad-plus-plus.org)|37.59.28.236|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4299968 (4.1M) [application/x-msdos-program]
Saving to: ‘npp.7.5.6.Installer.exe’

100%[===========================================================================================================================================================>] 4,299,968    541KB/s   in 12s

2018-05-14 13:53:42 (363 KB/s) - ‘npp.7.5.6.Installer.exe’ saved [4299968/4299968]
```

### 6.ps 命令

&emsp;&emsp;`ps` 命令用于查看系统中的进程状态，格式为`ps [参数]`。
&emsp;&emsp;估计在第一次执行这个命令时都要惊呆一下—怎么会有这么多输出值，这可怎么看得过来？其实，我们通常会将`ps` 命令与`管道符技术`搭配使用，用来抓取与某个指定服务进程相对应的`PID 号码`。`ps` 命令的常见参数以及作用如下所示：
| 参数 | 作用 |
| --- | --- |
| -a | 显示所有进程（包括其他用户的进程）|
| -u | 用户以及其他详细信息 |
| -x | 显示没有控制终端的进程 |

&emsp;&emsp;Linux 系统中时刻运行着许多进程，如果能够合理地管理它们，则可以优化系统的性能。在Linux 系统中，有5 种常见的进程状态，分别为运行、中断、不可中断、僵死与停止，其各自含义如下所示：

&emsp;&emsp;**R（运行）：进程正在运行或在运行队列中等待。**
&emsp;&emsp;**S（中断）：进程处于休眠中，当某个条件形成后或者接收到信号时，则脱离该状态。**
&emsp;&emsp;**D（不可中断）：进程不响应系统异步信号，即便用kill 命令也不能将其中断。**
&emsp;&emsp;**Z（僵死）：进程已经终止，但进程描述符依然存在，直到父进程调用`wait4()`系统函数后将进程释放。**
&emsp;&emsp;**T（停止）：进程收到停止信号后停止运行。**

&emsp;&emsp;当执行`ps aux` 命令后通常会看到如下所示的进程状态，仅部分例子，而且正常的输出值中不包括中文注释。

| USER | PID | %CPU | %MEM | VSZ | RSS | TTY | STAT | START | TIME | COMMAND |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 进程的所有者 | 进程 ID 号 | 运算器占用率 | 内存占用率 | 虚拟内存使用量（单位是KB） | 占用的固定内存量（单位是KB） | 所在终端 | 进程状态 | 被启动的时间 | 实际使用CPU的时间 | 命令名称与参数 |
| root |  1 | 0.1 | 0.3 | 52840 | 6664 | ? | Ss | 13:48 | 0:02 | /usr/lib/systemd/systemd --switched-root --system --deserialize 23 |
| root |  2 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [kthreadd] |
| root |  3 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [ksoftirqd/0] |
| root |  5 | 0.0 | 0.0 |     0 |    0 | ? | S< | 13:48 | 0:00 | [kworker/0:0H] |
| root |  7 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [migration/0] |
| root |  8 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [rcu_bh] |
| root |  9 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [rcuob/0] |
| root | 10 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [rcuob/1] |
| root | 11 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [rcuob/2] |
| root | 12 | 0.0 | 0.0 |     0 |    0 | ? | S  | 13:48 | 0:00 | [rcuob/3] |
**------------------------------------------------------------------------省略...------------------------------------------------------------------------**

---

> &emsp;&emsp;在Linux系统中的命令参数有`长短格式之分`，长格式和长格式 之间不能合并，长格式和短格式之间也不能合并，但短格式和短格式之间是可以合并 的，合并后仅保留一个`-`(减号）即可。另外ps命令可`允许参数不加减号`（-)，因此 可直接写成`ps aux`的样子。

### 7.top 命令

&emsp;&emsp;`top` 命令用于动态地监视进程活动与系统负载等信息，其格式为`top`。
&emsp;&emsp;`top` 命令相当强大，能够动态地查看系统运维状态，完全将它看作Linux 中的`强化版的Windows 任务管理器`。top 命令的运行界面如图所示：
![top视图](http://img.lynchj.com/f3c8c5515b51481aa73574bc811635fb.png)

在上图中，top 命令执行结果的前5 行为系统整体的统计信息，其所代表的含义如下：

&emsp;&emsp;**第1 行：系统时间、运行时间、登录终端数、系统负载（三个数值分别为1 分钟、5分钟、15 分钟内的平均值，数值越小意味着负载越低）。**
&emsp;&emsp;**第2 行：进程总数、运行中的进程数、睡眠中的进程数、停止的进程数、僵死的进程数。**
&emsp;&emsp;**第3 行：用户占用资源百分比、系统内核占用资源百分比、改变过优先级的进程资源百分比、空闲的资源百分比等。**
&emsp;&emsp;**第4 行：物理内存总量、内存使用量、内存空闲量、作为内核缓存的内存量。**
&emsp;&emsp;**第5 行：虚拟内存总量、虚拟内存使用量、虚拟内存空闲量、已被提前加载的内存量。**

> &emsp;&emsp;第3行中的数据均为CPU数据并以百分比格式显示，例如`99.6%id`意味着有 99.6%的CPU处理器资源处于空闲。

### 8.pidof 命令

&emsp;&emsp;`pidof` 命令用于查询某个指定服务进程的PID 值，格式为`pidof [参数] [服务名称]`。
&emsp;&emsp;每个进程的进程号码值（PID）是唯一的，因此可以通过PID 来区分不同的进程。例如，可以使用如下命令来查询本机上`top`服务程序的PID：
```
[root@lynchj tmp]# pidof top
3517
```

### 9.kill 命令

&emsp;&emsp;`kill` 命令用于终止某个指定PID 的服务进程，格式为`kill [参数] [进程PID]`。
&emsp;&emsp;接下来，我们使用`kill` 命令把上面用pidof 命令查询到的PID 所代表的进程终止掉，其命令如下所示。这种操作的效果等同于强制停止`top` 服务。
```
[root@lynchj tmp]# kill 3517
```

### 10.killall 命令

&emsp;&emsp;`killall` 命令用于终止某个指定名称的服务所对应的全部进程，格式为：`killall [参数] [进
程名称]`。
&emsp;&emsp;通常来讲，复杂软件的服务程序会有多个进程协同为用户提供服务，如果逐个去结束这些进程会比较麻烦，此时可以使用`killall` 命令来批量结束某个服务程序带有的全部进程。下面以`sshd` 服务程序为例，来结束其全部进程。
```
[root@lynchj ~]# pidof sshd
3457 3453 2704 2700 1695
[root@lynchj ~]# killall sshd
```

> &emsp;&emsp;如果我们在系统终端中执行一个命令后想立即停止它，可以同时按下`Ctrl + c`组合键（生产环境中比较常用的一个快捷键），这样将立即终止该命令的进程。或者，如果有些命令在执行时不断地在屏幕上输出信息，影响到后续命令的输入，则可 以在执行命令时在末尾添加上一个`&`符号，这样命令将进入系统后台来执行。

---

## 系统状态检测命令

---

### 1.ifconfig 命令

&emsp;&emsp;`ifconfig` 命令用于获取网卡配置与网络状态等信息，格式为`ifconfig [网络设备] [参数]`。
&emsp;&emsp;使用`ifconfig `命令来查看本机当前的网卡配置与网络状态等信息时，其实主要查看的就是网卡名称、inet 参数后面的IP 地址、ether 参数后面的网卡物理地址（又称为MAC 地址），以及RX、TX 的接收数据包与发送数据包的个数及累计流量：

```
[root@lynchj tmp]# ifconfig
eno16777728: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.175  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:fed4:9f4  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:d4:09:f4  txqueuelen 1000  (Ethernet)
        RX packets 18407  bytes 11388467 (10.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4151  bytes 372628 (363.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 6  bytes 560 (560.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6  bytes 560 (560.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 2.uname 命令

&emsp;&emsp;`uname` 命令用于查看系统内核与系统版本等信息，格式为`uname [-a]`。
&emsp;&emsp;在使用`uname` 命令时，一般会固定搭配上`-a 参数`来完整地查看当前系统的内核名称、主机名、内核发行版本、节点名、系统时间、硬件名称、硬件平台、处理器类型以及操作系统名称等信息。

```
[root@lynchj tmp]# uname -a
Linux lynchj.com 3.10.0-123.el7.x86_64 #1 SMP Mon May 5 11:16:57 EDT 2014 x86_64 x86_64 x86_64 GNU/Linux
```
&emsp;&emsp;顺带一提，如果要查看当前系统版本的详细信息，则需要查看redhat-release 文件，其命令以及相应的结果如下：
```
[root@lynchj tmp]# cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.0 (Maipo)
```

### 3.uptime 命令

&emsp;&emsp;`uptime` 用于查看系统的负载信息，格式为`uptime`。
&emsp;&emsp;`uptime` 命令真的很棒，它可以显示`当前系统时间`、`系统已运行时间`、`启用终端数量`以及`平均负载值`等信息。`平均负载值`指的是系统在最近`1 分钟`、`5 分钟`、`15 分钟`内的压力情况；负载值越低越好，尽量不要长期超过1，在生产环境中不要超过5。

```
[root@lynchj tmp]# uptime
15:06:43 up  1:17,  2 users,  load average: 0.00, 0.01, 0.05
```

### 4.free 命令

&emsp;&emsp;`free` 用于显示当前系统中内存的使用量信息，格式为`free [-h]`。
&emsp;&emsp;为了保证Linux 系统不会因资源耗尽而突然宕机，运维人员需要时刻关注内存的使用量。在使用`free` 命令时，可以结合使用`-h 参数`以更人性化的方式输出当前内存的实时使用量信息。下表所示为执行`free -h` 命令之后的输出信息。需要注意的是，输出信息中的中文注释是自行添加的内容，实际输出时没有相应的参数解释。

```
[root@lynchj tmp]# free -h
```

| ` ` | total | used | free | shared | buffers | cached |
| --- | --- | --- | --- | --- | --- | --- |
| | 内存总量 | 已用量 | 可用量 | 进程共享的内存量 | 磁盘缓存的内存量 | 缓存的内存量 |
| Mem: | 1.9G | 924M | 1.0G | 9.8M | 884K | 317M |
| -/+ buffers/cache: |  | 605M | 1.4G | | | | 
| Swap: | 2.0G | 0B | 2.0G | | | |

### 5.who 命令

&emsp;&emsp;`who` 用于查看当前登入主机的用户终端信息，格式为`who [参数]`。
&emsp;&emsp;这三个简单的字母可以快速显示出所有正在登录本机的用户的名称以及他们正在开启的终端信息。代码示例为：
```
[root@lynchj tmp]# who
root     :0           2018-05-14 13:50 (:0)
root     pts/1        2018-05-14 13:52 (192.168.1.177)
```

### 6.last 命令

&emsp;&emsp;`last` 命令用于查看所有系统的登录记录，格式为`last [参数]`。
&emsp;&emsp;使用`last` 命令可以查看本机的登录记录。但是，由于这些信息都是以日志文件的形式保存在系统中，因此黑客可以很容易地对内容进行篡改。千万不要单纯以该命令的输出信息而判断系统有无被恶意入侵！
```
[root@lynchj tmp]# last
root     pts/2        192.168.1.177    Mon May 14 15:25   still logged in
root     pts/0        192.168.1.177    Mon May 14 15:22   still logged in
root     pts/0        192.168.1.177    Mon May 14 14:52 - 15:06  (00:14)
root     pts/1        192.168.1.177    Mon May 14 13:52   still logged in
root     pts/0        :0               Mon May 14 13:50 - 13:53  (00:02)
root     :0           :0               Mon May 14 13:50   still logged in
(unknown :0           :0               Mon May 14 13:49 - 13:50  (00:00)
reboot   system boot  3.10.0-123.el7.x Mon May 14 21:49 - 15:25  (-6:-24)
root     pts/3        :1               Mon May 14 11:08 - 13:48  (02:40)
root     pts/2        :1               Mon May 14 11:07 - 11:08  (00:00)
root     :1           :1               Mon May 14 11:07 - 13:48  (02:41)
(unknown :1           :1               Mon May 14 11:06 - 11:07  (00:00)
guoqings pts/1        192.168.158.1    Sat May 12 16:04 - 13:48 (1+21:44)
guoqings pts/0        :0               Sat May 12 16:03 - 13:48 (1+21:45)
guoqings :0           :0               Sat May 12 16:02 - 13:48 (1+21:46)
(unknown :0           :0               Sat May 12 16:00 - 16:02  (00:01)
reboot   system boot  3.10.0-123.el7.x Sun May 13 00:00 - 13:48 (1+13:48)
reboot   system boot  3.10.0-123.el7.x Sat May 12 23:59 - 13:48 (1+13:49)
guoqings pts/0        :0               Sat May 12 15:53 - 15:53  (00:00)
guoqings pts/0        :0               Sat May 12 15:53 - 15:53  (00:00)
guoqings :0           :0               Sat May 12 15:52 - 15:55  (00:02)
(unknown :0           :0               Sat May 12 15:51 - 15:52  (00:01)
reboot   system boot  3.10.0-123.el7.x Sat May 12 23:47 - 13:48 (1+14:01)

wtmp begins Sat May 12 23:47:41 2018
```

### 7.history 命令

&emsp;&emsp;`history` 命令用于显示历史执行过的命令，格式为`history [-c]`。
&emsp;&emsp;`history` 命令应该是作者最喜欢的命令。执行`history 命令`能显示出当前用户在本地计算机中执行过的`最近1000 条命令记录`。如果觉得1000 不够用，还可以自定义`/etc/profile` 文件中的`HISTSIZE 变量值`。在使用`history` 命令时，如果使用`-c 参数`则会清空所有的命令历史记录。还可以使用`!编码数字`的方式来重复执行某一次的命令。总之，history 命令有很多有趣的玩法等待您去开发。

```
[root@lynchj tmp]# history
    1  history
[root@lynchj tmp]# !1
history
    1  history
```

&emsp;&emsp;历史命令会被保存到用户家目录中的`.bash_history` 文件中。Linux 系统中以点（.）开头的文件均代表隐藏文件，这些文件大多数为系统服务文件，可以用`cat` 命令查看其文件内容。
```
[root@lynchj tmp]# cat /root/.bash_history
man
man man
poweroff
cd /
echo "测试"
echo $SHELL
cd /usr/local/
mkdir tmp
cd tmp/
echo "新建文件" > text.txt
ll
vim text.txt
echo fdsf > text.txt
vim text.txt
echo fdsffjdklsfjl >> text.txt
vim text.txt
echo "追加内容" >> text.txt
echo "测试"
date
date "+%Y-%m-%d %H:%M:%S"
date -s "20180514 13:37:00"
wget -C https://notepad-plus-plus.org/repository/7.x/7.5.6/npp.7.5.6.Installer.exe
wget -c https://notepad-plus-plus.org/repository/7.x/7.5.6/npp.7.5.6.Installer.exe
ip addr
top
pidof sshd
```

&emsp;&emsp;要清空当前用户在本机上执行的Linux 命令历史记录信息，可执行如下命令：
```
[root@lynchj tmp]# history -c
```

### 8.sosreport 命令

&emsp;&emsp;`sosreport `命令用于收集系统配置及架构信息并输出诊断文档，格式为`sosreport`。
&emsp;&emsp;当Linux 系统出现故障需要联系技术支持人员时，大多数时候都要先使用这个命令来简单收集系统的运行状态和服务配置信息，以便让技术支持人员能够远程解决一些小问题，亦或让他们能提前了解某些复杂问题。如下所示：
```
[root@lynchj tmp]# sosreport

sosreport (version 3.0)

This command will collect diagnostic and configuration information from
this Red Hat Enterprise Linux system and installed applications.

An archive containing the collected information will be generated in
/var/tmp and may be provided to a Red Hat support representative.

Any information provided to Red Hat will be treated in accordance with
the published support policies at:

  https://access.redhat.com/support/

The generated archive may contain data considered sensitive and its
content should be reviewed by the originating organization before being
passed to any third party.

No changes will be made to system configuration.

Press ENTER to continue, or CTRL-C to quit. // 此处回车来确认收集信息

Please enter your first initial and last name [lynchj.com]:此处回车来确认主机编号
Please enter the case number that you are generating this report for:此处回车来确认主机编号

 Running plugins. Please wait ...

  Running 69/69: yum...
Creating compressed archive...

Your sosreport has been generated and saved in:
  // 收集好的压缩文件
  /var/tmp/sosreport-lynchj.com-20180514154217.tar.xz
// 校验码
The checksum is: f09ed896c1db2cf8381a1db4e699e362

Please send this file to your support representative.
```

---

## 工作目录切换命令

---

### 1.pwd 命令

&emsp;&emsp;`pwd` 命令用于显示用户当前所处的工作目录，格式为`pwd [选项]`。
```
[root@lynchj tmp]# pwd
/usr/local/tmp
```

### 2.cd 命令

&emsp;&emsp;`cd` 命令用于切换工作路径，格式为`cd [目录名称]`。
&emsp;&emsp;这个命令应该是最常用的一个Linux 命令了。可以通过cd 命令迅速、灵活地切换到不同的工作目录。除了常见的切换目录方式，还可以使用`cd -`命令返回到上一次所处的目录，使用`cd ..`命令进入上级目录，以及使用`cd ~`命令切换到当前用户的家目录，亦或使用`cd ~username`切换到其他用户的家目录。例如，可以使用`cd 路径`的方式切换进/etc 目录中：
```
[root@lynchj tmp]# cd /etc
```

### 3.ls 命令

&emsp;&emsp;`ls`命令用于显示目录中的文件信息，格式为`ls [选项] [文件] `。
&emsp;&emsp;所处的工作目录不同，当前工作目录下的文件肯定也不同。使用`ls` 命令的`-a`参数看到全部文件（包括隐藏文件），使用`-l`参数可以查看文件的属性、大小等详细信息。将这两个参数整合之后，再执行`ls` 命令即可查看当前目录中的所有文件并输出这些文件的属性信息：
```
[root@lynchj tmp]# ls -al
total 4208
drwxr-xr-x.  2 root root      51 May 14 15:56 .
drwxr-xr-x. 13 root root    4096 May 14 11:24 ..
-rw-r--r--.  1 root root 4299968 Mar 19 07:51 npp.7.5.6.Installer.exe
-rw-r--r--.  1 root root       7 May 14 15:56 text.txt
```

&emsp;&emsp;如果想要查看目录属性信息，则需要额外添加一个`-d 参数`。例如，可使用如下命令查看`/etc` 目录的权限与属性信息：
```
[root@lynchj tmp]# ls -ld /etc
drwxr-xr-x. 132 root root 8192 May 14 13:50 /etc
```

---

## 文本文件编辑命令

---

### 1.cat 命令

&emsp;&emsp;`cat` 命令用于查看纯文本文件（内容较少的），格式为`cat [选项] [文件]`。
&emsp;&emsp;Linux 系统中有多个用于查看文本内容的命令，每个命令都有自己的特点，比如这个`cat`命令就是用于查看内容较少的纯文本文件的。`cat` 这个命令也很好记，因为`cat` 在英语中是`猫`的意思，小猫咪是不是给您一种娇小、可爱的感觉呢？如果在查看文本内容时还想顺便显示行号的话，不妨在cat 命令后面追加一个`-n` 参数：
```
[root@lynchj tmp]# cat -n text.txt
     1  哈哈
```

### 2.more 命令

&emsp;&emsp;`more` 命令用于查看纯文本文件（内容较多的），格式为`more [选项]文件`。
&emsp;&emsp;如果需要阅读长篇小说或者非常长的配置文件，那么`小猫咪`可就真的不适合了。因为一旦使用`cat 命令`阅读长篇的文本内容，信息就会在屏幕上快速翻滚，导致自己还没有来得及看到，内容就已经翻篇了。因此对于长篇的文本内容，推荐使用`more 命令`来查看。`more 命令`会在最下面使用百分比的形式来提示您已经阅读了多少内容。您还可以使用空格键或回车键向下翻页：
```
[root@lynchj tmp]# more text.txt
哈哈
1
1
23
23

234
32
43
2
5
43
5
435
4
3
6
3
5
43
5
43
5
43
5
4
3
54
3
5
43
5
23
43
24

324
32

54
35
43

435
43
543
5
435
43
5
df
sfd
s
fd
a
gfd
fda
f
e
w
--More--(85%)
```

### 3.head 命令

&emsp;&emsp;`head` 命令用于查看纯文本文档的前N 行，格式为`head [选项] [文件]`。
&emsp;&emsp;在阅读文本内容时，谁也难以保证会按照从头到尾的顺序往下看完整个文件。如果只想
查看文本中前20 行的内容，该怎么办呢？head 命令可以派上用场了：
```
[root@lynchj tmp]# head -n 20 text.txt
哈哈
1
1
23
23

234
32
43
2
5
43
5
435
4
3
6
3
5
43
```

### 4.tail 命令

&emsp;&emsp;`tail` 命令用于查看纯文本文档的后N 行或持续刷新内容，格式为`tail [选项] [文件]`。
&emsp;&emsp;我们可能还会遇到另外一种情况，比如需要查看文本内容的最后20 行，这时就需要用到`tail 命令`了。`tail 命令`的操作方法与`head 命令`非常相似，只需要执行`tail -n 20 文件名`命令就可以达到这样的效果。`tail 命令`最强悍的功能是可以持续刷新一个文件的内容，当想要实时查看最新日志文件时，这特别有用，此时的命令格式为`tail -f 文件名`：
```
[root@lynchj tmp]# tail -f text.txt
w

fd
afd
saf
ds
af
dsf
ds
f

```

### 5.wc 命令

&emsp;&emsp;`wc` 命令用于统计指定文本的行数、字数、字节数，格式为`wc [参数] 文本`。
&emsp;&emsp;这可不是一种公共设施，其实这两者毫无关联。Linux系统中的`wc 命令`用于统计文本的行数、字数、字节数等。如果为了方便自己记住这个命令的作用，也可以联想到上厕所时好无聊，无聊到数完了手中的如厕读物上有多少行字。`wc` 的参数以及相应的作用如下表所示：
| 参数 | 作用 |
| --- | --- |
| -l | 只显示行数 |
| -w | 只显示单词数 |
| -c | 只显示字节数 |

&emsp;&emsp;在Linux 系统中，`/etc/passwd` 是用于保存系统账户信息的文件，要统计当前系统中有多少个
用户，可以使用下面的命令来进行查询，是不是很神奇：
```
[root@lynchj tmp]# wc -l /etc/passwd
38 /etc/passwd
```

### 6.stat 命令


&emsp;&emsp;`stat` 命令用于查看文件的具体存储信息和时间等信息，格式为`stat 文件名称`。
&emsp;&emsp;`stat 命令`可以用于查看文件的存储信息和时间等信息，命令`stat text.txt` 会显示出文件的三种时间状态：`Access`、`Modify`、`Change`。这三种时间的区别将在下面的touch命令中详细详解：
```
[root@lynchj tmp]# stat text.txt
  File: ‘text.txt’
  Size: 187             Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 102795020   Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:usr_t:s0
Access: 2018-05-14 16:19:27.347660644 +0800
Modify: 2018-05-14 16:19:26.138723458 +0800
Change: 2018-05-14 16:19:26.139723388 +0800
 Birth: -
```

### 7.cut 命令

&emsp;&emsp;`cut` 命令用于按“列”提取文本字符，格式为`cut [参数] 文本`。
&emsp;&emsp;在Linux 系统中，如何准确地提取出最想要的数据，这也是我们应该重点学习的内容。一般而言，按基于“行”的方式来提取数据是比较简单的，只需要设置好要搜索的关键词即可。但是如果按列搜索，不仅要使用`-f 参数`来设置需要看的列数，还需要使用`-d 参数`来设置间隔符号。`/etc/passwd` 在保存用户数据信息时，用户信息的每一项值之间是采用冒号来间隔的，接下来我们使用下述命令尝试提取出`/etc/passwd` 文件中的用户名信息，即提取以冒号（：）为间隔符号的第一列内容：
```
[root@lynchj tmp]# head -n 10 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
[root@lynchj tmp]# cut -d: -f1 /etc/passwd
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
operator
games
ftp
nobody
dbus
polkitd
unbound
colord
usbmuxd
avahi
avahi-autoipd
libstoragemgmt
saslauth
qemu
rpc
rpcuser
nfsnobody
rtkit
radvd
ntp
chrony
abrt
pulse
gdm
gnome-initial-setup
postfix
sshd
tcpdump
guoqingsong
```

### 8.diff 命令

&emsp;&emsp;`diff` 命令用于比较多个文本文件的差异，格式为`diff [参数] 文件`。
&emsp;&emsp;在使用`diff 命令`时，不仅可以使用`--brief 参数`来确认两个文件是否不同，还可以使用`-c 参数`来详细比较出多个文件的差异之处，这绝对是判断文件是否被篡改的有力神器。例如，先使用`cat 命令`分别查看`diff_A.txt `和`diff_B.txt` 文件的内容，然后进行比较：
```
[root@lynchj tmp]# cat diff_A.txt
Welcome to linuxprobe.com
Red Hat certified
Free Linux Lessons
Professional guidance
Linux Course
[root@linuxprobe ~]# cat diff_B.txt
Welcome tooo linuxprobe.com
Red Hat certified
Free Linux LeSSonS
////////.....////////
Professional guidance
Linux Course
```
&emsp;&emsp;接下来使用`diff --brief 命令`显示比较后的结果，判断文件是否相同：
```
[root@lynchj tmp]# diff --brief diff_A.txt diff_B.txt
Files diff_A.txt and diff_B.txt differ
```

&emsp;&emsp;最后使用带有`-c 参数`的`diff 命令`来描述文件内容具体的不同：
```
[root@lynchj tmp]# diff -c diff_A.txt diff_B.txt
*** diff_A.txt 2017-08-30 18:07:45.230864626 +0800
--- diff_B.txt 2017-08-30 18:08:52.203860389 +0800
***************
*** 1,5 ****
! Welcome to linuxprobe.com
Red Hat certified
! Free Linux Lessons
Professional guidance
Linux Course
--- 1,7 ----
! Welcome tooo linuxprobe.com
!
Red Hat certified
! Free Linux LeSSonS
! ////////.....////////
Professional guidance
Linux Course
```

### 9.awk 命令

&emsp;&emsp;`awk` 命令是一种处理文本文件的语言，是一个强大的文本分析工具，格式为`awk [选项参数] 'script' var=value file(s)`。
&emsp;&emsp;结合管道符使用可以让我们快速的获取到想要的某一列（如`端口号`，`进程号`）数据。在文本文件的筛选过滤也有着强大的处理功能。

##### 常见的使用：

&emsp;&emsp;`log.txt`测试文本：
```
[root@localhost tmp]# vim log.txt
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

&emsp;&emsp;**用法一：**

```
awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
```

&emsp;&emsp;**实例：**

```
# 每行按空格或TAB分割，输出文本中的1、4项
[root@localhost tmp]# awk '{print $1,$4}' log.txt
2 a
3 like
This's
10 orange,apple,mongo
# 格式化输出
[root@localhost tmp]# awk '{printf "%-8s %-10s\n",$1,$4}' log.txt
2        a
3        like
This's
10       orange,apple,mongo
```

&emsp;&emsp;**用法二：**

```
awk -v  # 设置变量
```

&emsp;&emsp;**实例：**

```
[root@localhost tmp]# awk -va=1 '{print $1,$1+a}' log.txt
2 3
3 4
This's 1
10 11
[root@localhost tmp]# awk -va=1 -vb=s '{print $1,$1+a,$1b}' log.txt
2 3 2s
3 4 3s
This's 1 This'ss
10 11 10s
```

&emsp;&emsp;**用法三：**

```
awk -f {awk脚本} {文件名}
```

&emsp;&emsp;**实例：**

```
[root@localhost tmp]# vim cal.awk
{print $1,$4}
[root@localhost tmp]# awk -f cal.awk log.txt
2 a
3 like
This's
10 orange,apple,mongo
```

##### 运算符

| 运算符 | 描述 |
| --- | --- |
| `= += -= *= /= %= ^= **=` | 赋值 |
| `?:` | C条件表达式 |
| `\|\|` | 逻辑或 |
| `&&` | 逻辑与 |
| `~ ~!` | 匹配正则表达式和不匹配正则表达式 |
| `< <= > >= != ==` | 关系运算符 |
| `空格` | 连接 |
| `+ -` | 加，减 |
| `* / %` | 乘，除与求余 |
| `+ - !` | 一元加，减和逻辑非 |
| `^ ***` | 求幂 |
| `++ --` | 增加或减少，作为前缀或后缀 |
| `$` | 字段引用 |
| `in` | 数组成员 |

&emsp;&emsp;**过滤第一列大于2的行：**

```
[root@localhost tmp]# awk '$1>2' log.txt
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

&emsp;&emsp;**过滤第一列等于2的行：**

```
[root@localhost tmp]# awk '$1==2 {print $1,$4}' log.txt
2 a
```

&emsp;&emsp;**过滤第一列大于2并且第二列等于`Are`的行：**

```
[root@localhost tmp]# awk '$1>2 && $2=="Are" {print $1,$4}' log.txt
3 like
```

---

## 文件目录管理命令

---

### 1.touch 命令

&emsp;&emsp;`touch` 命令用于创建空白文件或设置文件的时间，格式为`touch [选项] [文件]`。
&emsp;&emsp;在创建空白的文本文件方面，这个`touch 命令`相当简捷，简捷到没有必要铺开去讲。比如，`touch text.txt` 命令可以创建出一个名为`text.txt` 的空白文本文件。对`touch 命令`来讲，有难度的操作主要是体现在设置文件内容的修改时间（mtime）、文件权限或属性的更改时间（ctime）与文件的读取时间（atime）上面。touch 命令的参数及其作用如下表所示：

| 参数 | 作用 |
| --- | --- |
| -a | 仅修改“读取时间”（atime） |
| -m | 仅修改“修改时间”（mtime） |
| -d | 同时修改atime 与mtime |

&emsp;&emsp;接下来，我们先使用`ls 命令`查看一个文件的修改时间，然后修改这个文件，最后再通过`touch 命令`把修改后的文件时间设置成修改之前的时间（很多黑客就是这样做的呢）：
```
[root@lynchj tmp]# ls -l
total 4204
-rw-r--r--. 1 root root 4299968 Mar 19 07:51 npp.7.5.6.Installer.exe
-rw-r--r--. 1 root root     196 May 14 16:19 text.txt
[root@lynchj tmp]# vim text.txt
[root@lynchj tmp]# ls -l
total 4204
-rw-r--r--. 1 root root 4299968 Mar 19 07:51 npp.7.5.6.Installer.exe
-rw-r--r--. 1 root root     204 May 14 16:49 text.txt
[root@lynchj tmp]# touch -d "2018-05-14 16:19" text.txt
[root@lynchj tmp]# ls -l
total 4204
-rw-r--r--. 1 root root 4299968 Mar 19 07:51 npp.7.5.6.Installer.exe
-rw-r--r--. 1 root root     204 May 14 16:19 text.txt
```

### 2.mkdir 命令

&emsp;&emsp;`mkdir` 命令用于创建空白的目录，格式为`mkdir [选项] 目录`。
&emsp;&emsp;在Linux 系统中，文件夹是最常见的文件类型之一。除了能创建单个空白目录外，`mkdir
命令`还可以结合`-p 参数`来递归创建出具有嵌套叠层关系的文件目录。

### 3.cp 命令

&emsp;&emsp;`cp 命令`用于复制文件或目录，格式为`cp [选项] 源文件 目标文件`。
&emsp;&emsp;大家对文件复制操作应该不陌生，在Linux 系统中，复制操作具体分为3 种情况：
&emsp;&emsp;**如果目标文件是目录，则会把源文件复制到该目录中；**
&emsp;&emsp;**如果目标文件也是普通文件，则会询问是否要覆盖它；**
&emsp;&emsp;**如果目标文件不存在，则执行正常的复制操作；**

&emsp;&emsp;`cp 命令`的参数及其作用如下表所示：

| 参数 | 作用 |
| --- | --- |
| -p | 保留原始文件的属性 |
| -d | 若对象为`链接文件`，则保留该`链接文件`的属性 |
| -r | 递归持续复制（用于目录） |
| -i | 若目标文件存在则询问是否覆盖 |
| -a | 相当于-pdr（p、d、r 为上述参数） |

### 4.mv 命令

&emsp;&emsp;`mv 命令`用于剪切文件或将文件重命名，格式为`mv [选项] 源文件 [目标路径|目标文件名]`。
&emsp;&emsp;剪切操作不同于复制操作，因为它会默认把源文件删除掉，只保留剪切后的文件。如果在同一个目录中对一个文件进行剪切操作，其实也就是对其进行重命名。

### 5.rm 命令

&emsp;&emsp;`rm 命令`用于删除文件或目录，格式为`rm [选项] 文件`。
&emsp;&emsp;在Linux 系统中删除文件时，系统会默认向您询问是否要执行删除操作，如果不想总是看到这种反复的确认信息，可在`rm 命令`后跟上`-f 参数`来强制删除。另外，想要删除一个目录，需要在`rm 命令`后面一个`-r 参数`才可以，否则删除不掉。

### 6.file 命令

&emsp;&emsp;`file` 命令用于查看文件的类型，格式为`file 文件名`。
&emsp;&emsp;在 Linux 系统中，由于文本、目录、设备等所有这些一切都统称为文件，而我们又不能单凭后缀就知道具体的文件类型，这时就需要使用 file 命令来查看文件类型了。
```
[root@lynchj tmp]# file text.txt
text.txt: UTF-8 Unicode text
[root@lynchj tmp]# file /dev/sda
/dev/sda: block special
```

---

## 打包压缩与搜索命令

---

### 1.tar 命令

&emsp;&emsp;`tar` 命令用于对文件进行打包压缩或解压，格式为`tar [选项] [文件]`。
&emsp;&emsp;在 Linux 系统中，常见的文件格式比较多，其中主要使用的是`.tar` 或`.tar.gz` 或`.tar.bz2` 格式，我们不用担心格式太多而记不住，其实这些格式大部分都是由 tar 命令来生成的。下面列举一下常用命令参数：

| 命令 | 作用 |
| --- | --- |
| -c | 创建压缩文件 |
| -x | 解开压缩文件 |
| -t | 查看压缩包内有哪些文件 |
| -z | 用Gzip 压缩或解压 |
| -j | 用bzip2 压缩或解压 |
| -v | 显示压缩或解压的过程 |
| -f | 目标文件名 |
| -p | 保留原始的权限与属性 |
| -P | 使用绝对路径来压缩 |
| -C | 指定解压到的目录 |

&emsp;&emsp;首先，`-c 参数`用于创建压缩文件，`-x 参数`用于解压文件，因此这两个参数不能同时使用。其次，`-z 参数`指定使用Gzip 格式来压缩或解压文件，`-j 参数`指定使用bzip2 格式来压缩或解压文件。用户使用时则是根据文件的后缀来决定应使用何种格式参数进行解压。在执行某些压缩或解压操作时，可能需要花费数个小时，如果屏幕一直没有输出，您一方面不好判断打包的进度情况，另一方面也会怀疑电脑死机了，因此非常推荐使用`-v 参数`向用户不断显示压缩或解压的过程。`-C 参数`用于指定要解压到哪个指定的目录。`-f 参数`特别重要，它必须放到参数的最后一位，代表要压缩或解压的软件包名称。一般使用`tar -czvf 压缩包名称.tar.gz 要打包的目录`命令把指定的文件进行打包压缩；相应的解压命令为`tar -xzvf 压缩包名称.tar.gz`。下面我们来逐个演示下打包压缩与解压的操作。先使用`tar 命令`把`/etc` 目录通过`gzip 格式`进行打包压缩，并把文件命名为`etc.tar.gz`：
```
[root@lynchj tmp]# tar -czvf etc.tar.gz /etc
tar: Removing leading `/' from member names
/etc/
/etc/fstab
/etc/crypttab
/etc/mtab
/etc/fonts/
/etc/fonts/conf.d/
/etc/fonts/conf.d/65-0-madan.conf
/etc/fonts/conf.d/59-liberation-sans.conf
/etc/fonts/conf.d/90-ttf-arphic-uming-embolden.conf
/etc/fonts/conf.d/59-liberation-mono.conf
/etc/fonts/conf.d/66-sil-nuosu.conf
………………省略部分压缩过程信息………………
```

&emsp;&emsp;接下来将打包后的压缩包文件指定解压到/root/etc 目录中（先使用mkdir 命令来创建
/root/etc 目录）：
```
[root@lynchj tmp]# mkdir /root/etc
[root@lynchj tmp]# tar -xzvf etc.tar.gz -C /root/etc/
etc/
etc/fstab
etc/crypttab
etc/mtab
etc/fonts/
etc/fonts/conf.d/
etc/fonts/conf.d/65-0-madan.conf
etc/fonts/conf.d/59-liberation-sans.conf
etc/fonts/conf.d/90-ttf-arphic-uming-embolden.conf
etc/fonts/conf.d/59-liberation-mono.conf
etc/fonts/conf.d/66-sil-nuosu.conf
etc/fonts/conf.d/65-1-vlgothic-gothic.conf
etc/fonts/conf.d/65-0-lohit-bengali.conf
etc/fonts/conf.d/20-unhint-small-dejavu-sans.conf
………………省略部分解压过程信息………………
```

### 2.grep 命令

&emsp;&emsp;`grep` 命令用于在文本中执行关键词搜索，并显示匹配的结果，格式为`grep [选项] [文件]`。
&emsp;&emsp;`grep 命令`的参数及其作用如下表所示。

| 参数 | 命令 |
| --- | --- |
| -b | 将可执行文件（binary）当作文本文件（text）来搜索 |
| -c | 仅显示找到的行数 |
| -i | 忽略大小写 |
| -n | 显示行号 |
| -v | 反向选择—仅列出没有“关键词”的行 |

&emsp;&emsp;`grep 命令`是用途最广泛的文本搜索匹配工具，虽然有很多参数，但是大多数基本上都用不到。我们在这里只讲两个最最常用的参数：`-n 参数`用来显示搜索到信息的行号；`-v 参数`用于反选信息（即没有包含关键词的所有信息行）。这两个参数几乎能完成您日后80%的工作需要，至于其他上百个参数，即使以后在工作期间遇到了，再使用`man grep 命令`查询也来得及。在Linux 系统中，`/etc/passwd` 文件是保存着所有的用户信息，而一旦用户的登录终端被设置成`/sbin/nologin`，则不再允许登录系统，因此可以使用`grep 命令`来查找出当前系统中不允许登录系统的所有用户信息：

```
[root@lynchj tmp]# grep /sbin/nologin /etc/passwd
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
………………省略部分输出过程信息………………
```

### 3.find 命令

&emsp;&emsp;`find` 命令用于按照指定条件来查找文件，格式为`find [查找路径] 寻找条件 寻找关键字`。
&emsp;&emsp;大家都知道`Linux 系统中的一切都是文件`，接下来就要见证这句话的分量了。在Linux 系统中，搜索工作一般都是通过`find 命令`来完成的，它可以使用不同的文件特性作为寻找条件（如文件名、大小、修改时间、权限等信息），一旦匹配成功则默认将信息显示到屏幕上。`find 命令`的参数以及作用下表所示。

| 参数 | 作用 |
| --- | --- |
| -name | 匹配名称 |
| -perm | 匹配权限（mode 为完全匹配，-mode 为包含即可） |
| -user | 匹配所有者 |
| -group | 匹配所有组 |
| -mtime -n +n | 匹配修改内容的时间（-n 指n 天以内，+n 指n 天以前） |
| -atime -n +n | 匹配访问文件的时间（-n 指n 天以内，+n 指n 天以前） |
| -ctime -n +n | 匹配修改文件权限的时间（-n 指n 天以内，+n 指n 天以前） |
| -nouser | 匹配无所有者的文件 |
| -nogroup | 匹配无所有组的文件 |
| -newer f1 !f2 | 匹配比文件f1 新但比f2 旧的文件 |
| `--type b/d/c/p/l/f` | 匹配文件类型（后面的字母参数依次表示块设备、目录、字符设备、管道、链接文件、文本文件） |
| -size | 匹配文件的大小（+50KB 为查找超过50KB 的文件，而-50KB 为查找小于50KB 的文件） |
| -prune | 忽略某个目录 |
| -exec …… {}\; | 后面可跟用于进一步处理搜索结果的命令（下文会有演示） |

&emsp;&emsp;这里需要重点讲解一下`-exec 参数`重要的作用。这个参数用于把`find 命令`搜索到的结果交由紧随其后的命令作进一步处理，它十分类似于管道符技术，并且由于`find 命令`对参数的特殊要求，因此虽然`exec` 是长格式形式，但依然只需要一个减号（-）。
&emsp;&emsp;根据文件系统层次标准（Filesystem Hierarchy Standard）协议，Linux 系统中的配置文件会保存到`/etc 目录`中。如果要想获取到该目录中所有以`host 开头`的文件列表，可以执行如下命令：

```
[root@lynchj tmp]# find /etc -name "host*" -print
/etc/avahi/hosts
/etc/host.conf
/etc/hosts
/etc/hosts.allow
/etc/hosts.deny
/etc/selinux/targeted/modules/active/modules/hostname.pp
/etc/hostname
```

&emsp;&emsp;如果要在整个系统中搜索权限中包括`SUID 权限`的所有文件，只需使用`-4000` 即可：

```
[root@lynchj tmp]# find / -perm -4000
find: ‘/proc/19864/task/19864/fd/6’: No such file or directory
find: ‘/proc/19864/task/19864/fdinfo/6’: No such file or directory
find: ‘/proc/19864/fd/6’: No such file or directory
find: ‘/proc/19864/fdinfo/6’: No such file or directory
/usr/bin/fusermount
/usr/bin/su
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/mount
/usr/bin/umount
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/crontab
/usr/bin/Xorg
/usr/bin/staprun
/usr/bin/at
/usr/bin/sudo
/usr/sbin/pam_timestamp_check
/usr/sbin/unix_chkpwd
/usr/sbin/userhelper
/usr/sbin/usernetctl
/usr/sbin/mount.nfs
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib64/dbus-1/dbus-daemon-launch-helper
/usr/libexec/spice-gtk-x86_64/spice-client-glib-usb-acl-helper
/usr/libexec/pulse/proximity-helper
/usr/libexec/qemu-bridge-helper
/usr/libexec/abrt-action-install-debuginfo-to-abrt-cache
```

> &emsp;&emsp;进阶实验：在整个文件系统中找出`SUID 权限`的文件并复制到 `/user/local/tmp` 目录。
&emsp;&emsp;该实验的重点是`-exec {} \;` 参数，其中的`{}`表示find命令搜索出的每一个文件，并且命令的结尾必须是`\;`，完成该实验的具体命令如下：
```
[root@linuxprobe 〜]# find / -perm -4000 -exec cp -a {} /usr/local/tmp/ \;
```

> 参考资料：《Linux就该这么学》
