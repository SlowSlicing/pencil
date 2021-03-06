### 一切从`/`开始

　　在Linux 系统中，目录、字符设备、块设备、套接字、打印机等都被抽象成了文件，即：`Linux 系统中一切都是文件`。既然平时我们打交道的都是文件，那么又应该如何找到它们呢？在Windows 操作系统中，想要找到一个文件，我们要依次进入该文件所在的磁盘分区（假设这里是D 盘），然后在进入该分区下的具体目录，最终找到这个文件。但是在Linux系统中并不存在C/D/E/F 等盘符，Linux 系统中的一切文件都是从`根（/）`目录开始的，并按照`文件系统层次化标准` `（FHS，Filesystem Hierarchy Standard）`采用树形结构来存放文件，以及定义了常见目录的用途。另外，Linux系统中的文件和目录名称是`严格区分大小写`的。例如，root、rOOt、Root、rooT 均代表不同的目录，并且文件名称中`不得包含斜杠（/）`。Linux 系统中的文件存储结构如下图所示：

![Linux 系统中的文件存储结构](http://img.lynchj.com/1d80e197678541c58f4dffb4ae2f5b39.png)

　　前文提到的`FHS` 是根据以往无数Linux 系统用户和开发者的经验而总结出来的，是用户在Linux 系统中存储文件时需要遵守的规则，用于指导我们应该把文件保存到什么位置，以及告诉用户应该在何处找到所需的文件。但是，FHS 对于用户来讲只能算是一种道德上的约束，有些用户就是懒得遵守，依然会把文件到处乱放，有些甚至从来没有听说过它。这里并不是号召各位读者去谴责他们，而是建议大家要灵活运用所学的知识，千万`不要认准`这个FHS协定只讲死道理，不然吃亏的可就是自己了。在Linux 系统中，最常见的目录以及所对应的存放内容如下表所示：

| 目录名称 | 应放置文件的内容 |
| --- | --- |
| /boot | 开机所需文件—内核、开机菜单以及所需配置文件等 |
| /dev | 以文件形式存放任何设备与接口 |
| /etc | 配置文件 |
| /home | 用户家目录 |
| /bin | 存放单用户模式下还可以操作的命令 |
| /lib | 开机时用到的函数库，以及/bin 与/sbin 下面的命令要调用的函数 |
| /sbin | 开机过程中需要的命令 |
| /media | 用于挂载设备文件的目录 |
| /opt | 放置第三方的软件 |
| /root | 系统管理员的家目录 |
| /srv | 一些网络服务的数据文件目录 |
| /tmp | 任何人均可使用的“共享”临时目录 |
| /proc | 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等 |
| /usr/local | 用户自行安装的软件 |
| /usr/sbin | Linux 系统开机时不会使用到的软件/命令/脚本 |
| /usr/share | 帮助与说明文件，也可放置共享文件 |
| /var | 主要存放经常变化的文件，如日志 |
| /lost+found | 当文件系统发生错误时，将一些丢失的文件片段存放在这里 |

　　在Linux 系统中另外还有一个重要的概念—路径。路径指的是如何定位到某个文件，分为`绝对路径`与`相对路径`。绝对路径指的是从根目录（/）开始写起的文件或目录名称，而相对路径则指的是相对于当前路径的写法。我们来看下面这个例子，以帮助大家理解。假如有位外国游客来到中国潘家园旅游，当前内急但是找不到洗手间，特意向您问路，那么您有两种正确的指路方法。

　　**1、绝对路径（absolute path）：**首先坐飞机来到中国，到了北京出首都机场坐机场快轨到三元桥，然后换乘10 号线到潘家园站，出站后坐34 路公交车到农光里，下车后路口左转。
　　**2、相对路径（relative path）：**前面路口左转。

　　这两种方法都正确。如果您说的是绝对路径，那么任何一位外国游客都可以按照这个提示找到潘家园的洗手间，但是太繁琐了。如果您说的是相对路径，虽然表达很简练，但是这位外国游客只能从当前位置（不见得是潘家园）出发找到洗手间，因此并不能保证在前面的路口左转后可以找到洗手间，由此可见，相对路径不具备普适性。
