# 入门

## 系统启动过程

　　CentOS 7 的启动过程是这样的：

| 顺序 | 说明 |
| --- | --- |
| post | 加电 |
| BISO | 进入BIOS |
| bootloader(MBR) | 加载磁盘主引导记录 |
| kernel(ramdisk) | 加载内核 |
| rootfs | 初始化rootfs |
| /sbin/init   | 系统初始化。这里的 init 在不同系统上还有所不同，CentOS 5：Sysv lnit，CentOS 6：Upstart，CentOS 7：systemd 系统守护进程 |

```
UEFi或BIOS初始化，运行POST开机自检
选择启动设备
引导装载程序, centos7是grub2
加载装载程序的配置文件： /etc/grub.d/
    /etc/default/grub /boot/grub2/grub.cfg
加载initramfs驱动模块
加载内核选项
内核初始化， centos7使用systemd代替init
执行initrd.target所有单元，包括挂载/etc/fstab
从initramfs根文件系统切换到磁盘根目录
systemd执行默认target配置，配置文件
    /etc/systemd/default.target /etc/systemd/system/
systemd执行sysinit.target初始化系统及basic.target准备操
作系统
systemd启动multi-user.target下的本机与服务器服务
systemd执行multi-user.target下的/etc/rc.d/rc.local
Systemd执行multi-user.target下的getty.target及登入服
务
systemd执行graphical需要的服务
```

## systemd

　　`Systemd` 是系统启动和服务器守护进程管理器，负责在系统启动或运行时，激活系统资源，服务器进程和其它进程。

　　**特性**：
* 系统引导时实现服务并行启动，实现快速开机。
* 按需启动守护进程。
* 能自动保存系统状态快照。
* 基于依赖关系定义服务控制逻辑（自动化的服务依赖关系管理）。
* 同时采用 `socket` 式与 `D-Bus` 总线式激活服务。

### 核心概念 unit（单元）

　　`unit` 表示不同类型的 `systemd` 对象，unit 由其相关的配置文件进行标识，识别和配置。相关文件中主要包含了系统服务，监听的 socket，保存的快照以及其他与 `init` 相关的信息。

　　这些配置文件主要保存在：

* **/usr/lib/systemd/system：**每个服务最主要的启动脚本设置，类似于之前的 `/etc/init.d/`。
* **/run/systemd/system：**系统执行过程中所产生的服务脚本，比上面目录优先运行。
* **/etc/systemd/system：**管理员建立的执行脚本，类似于 `/etc/rc.d/rcN.d/Sxx` 类的功能，比上面目录优先运行。

> 　　这三个目录的配置文件优先级从低到高，如果同一选项三个地方都配置了，优先级高的会覆盖优先级低的。

　　系统安装时，默认会将 unit 文件放在 `/usr/lib/systemd/system` 目录。如果我们想要修改系统默认的配置，比如 `sshd.service`，一般有两种方法：

1. 在 `/etc/systemd/system` 目录下创建 `sshd.service` 文件，里面写上我们自己的配置。
2. 在 `/etc/systemd/system` 下面创建 `sshd.service` 目录，在这个目录里面新建任何以 `.conf` 结尾的文件，然后写入我们自己的配置。推荐这种做法。

　　`/run/systemd/system` 这个目录一般是进程在运行时动态创建 unit 文件的目录，一般很少修改，除非是修改程序运行时的一些参数时，即 `Session` 级别的，才在这里做修改。

　　unit 的常见类别：

```
$ systemctl -t help
Available unit types:
service
socket
busname
target
snapshot
device
mount
automount
swap
timer
path
slice
scope
```

* **Service unit：**文件扩展名为 `.service`，用于定义系统服务，类似服务脚本。
* **Target unit：**文件扩展为 `.target`，用于模拟实现`运行级别`；与 `/etc/inittab` 不一样，只是为了对应老版本的系统。
* **Device unit：**`.device`，用于定义内核识别的设备；CentOS 6 下，`/dev` 目录下的设备文件是由 `udev` 根据 `/sys/` 目录下由内核探测输出的信息而创建的，对 CentOS 7 来说，`/dev` 目录下是由 `systemd` 和 `udev` 联合创建的，主要由 systemd 完成 `.systemd` 识别硬件，主要靠 `*.device` 文件。
* **Mount unit：**`.mount`，用于定义文件系统挂载点；`kernel 3.0` 版本后的系统，大都是用 `cgroup`（k控制组）来实现资源分配 `.mount` 后看到大量的 `cgroup` 信息，cgroup 实现对资源分配的一种内核中的资源分配的机制 `.systemd` 负责对 cgroup 的激活，实现资源分配。
* **Socket unit：**`.socket`，用于标识进程间通信用到的 socket 文件。任何主机监听在一个套接字上，任何进程监听在一个套接字上，或者任何进程打开一个随机端口去与别的服务器的进程进行通道，都要创建 socket 文件。现在这些创建等功能是有 systemd 负责管理实现的。
* **Snapshot unit：**`.snapshot`，管理系统快照。
* **Swap unit：**`.swap`，用于表示swap设备。
* **Autommount unit：**`.autommount`，文件系统自动挂载点设备。
* **Path unit：**`.path`，用于定于文件系统中的文件或目录使用，常用于当文件系统发生变化时，延迟激活服务，如： `spool 目录`。

　　**关键特性：**

1. 基于 socket 的激活机制；socket 与程序分离。
2. 基于D-bus（总线）的激活机制；如果总线上有对某一个服务的访问，那么就基于总线的请求，将某一设备激活。
3. 基于 device 的激活机制；当某个设备插入，能自动激活 `mount unit` 和 `deviceunit` 或 `autommount unit`，能监控当前系统和内核所输出的硬件信息，一旦发现某个硬件出现，先创建设备文件，在自动挂载至某挂载点；挂载点不存在，还能自动创建。
4. 基于 Path 的激活机制；系统能够监控某个目录或文件的存在，来激活一个服务或进程，比如说，某个进程崩溃时创建一个锁文件，系统会根据判断锁文件的存在，去启动另一个进程来判断崩溃进程的原因; 。
5. 系统快照；保存各 `unit` 的当前状态信息于持久存储设备中。
6. 向后兼容 `sysv lnit` 脚本：`/etc/init.d/` 下的服务脚本。

## 管理命令

　　service 类型的 unit 文件；所有命令要靠 systemctl 控制，兼容早期服务脚本。

　　**命令示例：**

```
显示所有单元状态
    systemctl 或 systemctl list-units
只显示服务单元的状态
    systemctl --type=service
显示sshd服务单元
    systemctl status sshd.service –l
验证sshd服务当前是否活动
    systemctl is-active sshd
启动，停止和重启sshd服务
    systemctl start sshd.service
    systemctl stop sshd.service
    systemctl restart sshd.service
重新加载配置
    systemctl reload sshd.service
列出活动状态的所有服务单元
    systemctl list-units --type=service
列出所有服务单元
    systemctl list-units --type=service --all
查看服务单元的启用和禁用状态。
    systemctl list-unit-files --type=service
列出失败的服务
    systemctl --failed --type=service
列出依赖的单元
    systemctl list-dependencies sshd
验证sshd服务是否开机启动
    systemctl is-enabled sshd
禁用network，使之不能自动启动,但手动可以
    systemclt disable network
启用network
    systemctl enable network
禁用network，使之不能手动或自动启动
    systemclt mask network
启用network
    systemctl umask network
```

# 自定义启动 tomcat service

* 环境

| 路径 | 解释 |
| --- | --- |
| /home/tomcat/jenkins_8093 | tomcat 路径，这里是用来部署一个 jenkins 的 tomcat |
| /usr/local/jdk1.8.0_144 | JAVA_HOME |

> 　　新建一个 tomcat 用户，不要是用 root，管理好对应的权限对服务器安全有很大的帮助。

* 编写 systemd service 文件

　　每一个服务以 `.service` 结尾，一般会分为3部分：`[Unit]`、`[Service]` 和 `[Install]`。

```
$ vim /etc/systemd/system/jenkins.service
[Unit]
Description=Jenkins
After=network.target

[Service]
# 环境变量
Environment='JAVA_HOME=/usr/local/jdk1.8.0_144'

User=tomcat
Group=tomcat

Type=forking

ExecStart=/home/tomcat/jenkins_8093/bin/startup.sh
ExecStop=/home/tomcat/jenkins_8093/bin/shutdown.sh
ExecReload=/bin/kill -s HUP $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

　　**配置说明：**

　　**`[Unit]`**

1. Description：服务的简单描述；
2. Documentation：服务文档；
3. After：依赖，仅当依赖的服务启动之后再启动自定义的服务单元；

　　**`[Service]`**

1. Type：启动类型 simple、forking、oneshot、notify、dbus；
    1. Type = simple（默认值）：systemd 认为该服务将立即启动，服务进程不会 fork。如果该服务要启动其他服务，不要使用此类型启动，除非该服务是 socket 激活型；
    2. Type = forking：systemd 认为当该服务进程 fork，且父进程退出后服务启动成功。对于常规的守护进程（daemon），除非你确定此启动方式无法满足需求， 使用此类型启动即可。使用此启动类型应同时指定 `PIDFile=`，以便 systemd 能够跟踪服务的主进程。
    3. Type = oneshot：这一选项适用于只执行一项任务、随后立即退出的服务。可能需要同时设置 `RemainAfterExit = yes` 使得 systemd 在服务进程退出之后仍然认为服务处于激活状态。
    4. Type = notify：与 Type = simple 相同，但约定服务会在就绪后向 systemd 发送一个信号，这一通知的实现由 `libsystemd-daemon.so` 提供。
    5. Type = dbus：若以此方式启动，当指定的 BusName 出现在 DBus 系统总线上时，systemd 认为服务就绪。
2. PIDFile：pid 文件路径；
3. ExecStartPre：启动前要做什么，比如测试一下配置文件是否正常；
4. ExecStart：启动；
5. ExecReload：重载；
6. ExecStop：停止；
7. PrivateTmp：True 表示给服务分配独立的临时空间；
8. Environment：指定环境变量，如果不指定，想让它默认区使用系统环境变量会出错。格式：`Environment='key1=val1' 'key2=val2'` 
9. User：指定用哪个用户
10. Group：指定用哪个组

　　**`[Install]`**

1. WantedBy：服务安装的用户模式，从字面上看，就是想要使用这个服务的有是谁？上文中使用的是：`multi-user.target`，就是指想要使用这个服务的目录是多用户。

>　　每一个 `.target` 实际上是链接到我们单位文件的集合，当我们执行 `systemctl enable sshd.service` 时，就会在 `/etc/systemd/system/multi-user.target.wants/` 目录下新建一个 `/usr/lib/systemd/system/sshd.service` 文件的符号链接。

3. 把 tomcat 设置开机自启

```
$ systemctl enable jenkins
Created symlink from /etc/systemd/system/multi-user.target.wants/jenkins.service to /etc/systemd/system/jenkins.service.

# 重启看结果
$ reboot
```

