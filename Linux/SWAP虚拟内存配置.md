　　`swap`是Linux中的虚拟内存，用于扩充物理内存不足而用来存储临时数据存在的。它类似于Windows中的虚拟内存。在Windows中，只可以使用文件来当作虚拟内存。而linux可以文件或者分区来当作虚拟内存。

　　首先查看当前的内存和`swap 空间`大小(默认单位为k, -m 单位为M)：

```
free -m
```

![查看swap空间](http://www.huzs.net/wp-content/uploads/2013/08/f19e344a2a6f398ff867bf4a14404d3c.gif)

　　此处可以看到总内存是503M,swap不存在。

　　查看swap信息，包括文件和分区的详细信息

```
swapon -s
```

　　或者

```
cat /proc/swaps
```

　　如果都没有，我们就需要手动添加交换分区。注意，OPENVZ架构的VPS是不支持手动添加交换分区的。

　　添加交换空间有两种选择：添加一个交换分区或添加一个交换文件。推荐你添加一个交换分区；不过，若你没有多少空闲空间可用， 则添加交换文件。

### 一、增加swap交换文件

　　**1、使用dd命令创建一个swap交换文件**

```
dd if=/dev/zero of=/home/swap bs=1024 count=1024000
```

　　这样就建立一个`/home/swap`的分区文件，大小为1G。

　　**2、制作为swap格式文件**

```
mkswap /home/swap
```

　　**3、再用swapon命令把这个文件分区挂载swap分区**

```
/sbin/swapon /home/swap
```

　　我们用`free -m`命令看一下，发现已经有交换分区了。

![查看swap空间](http://www.huzs.net/wp-content/uploads/2013/08/cf643d69451cb6b1de3ad6ec95cd53a61.gif)

　　但是重启系统后，swap分区又变成0了。

　　**4、为防止重启后swap分区变成0，要修改`/etc/fstab`文件**

```
vim /etc/fstab
```

　　在文件末尾（最后一行）加上

```
/home/swap swap swap default 0 0
```

　　这样就算重启系统，swap分区还是有值。

#### 删除swap交换文件

　　**1、先停止swap分区**

```
/sbin/swapoff /home/swap
```

　　**2、删除swap分区文件**

```
rm -rf /home/swap
```

　　**3、删除自动挂载配置命令**

```
vim /etc/fstab
```
　　这行删除

```
/home/swap swap swap default 0 0
```

　　**这样就能把手动增加的交换文件删除了。**

>注意：
　　1、增加删除`swap`的操作只能使用root用户来操作。
　　2、装系统时分配的swap分区貌似删除不了。
　　3、swap分区一般为内存的1.5-2倍

### 二、使用分区来做SWAP(虚拟内存).
　　**1、使用fdisk来创建交换分区（假设 /dev/sdb2 是创建的交换分区）**

　　**2、使用 mkswap 命令来设置交换分区：**

```
mkswap /dev/sdb2
```

　　**3、启用交换分区**

```
swapon /dev/sdb2
```

　　**4、写入/etc/fstab,以便在引导时启用**

```
/dev/sdb2 swap swap defaults 0 0
```

#### 删除交换分区
> 步骤如下：

　　**1、先停止swap分区**

```
/sbin/swapoff /dev/sdb2
```

　　**2、删除自动挂载配置命令**

```
vim /etc/fstab
```

　　这行删除

```
/dev/sdb2 swap swap defaults 0 0
```

　　**这样就能把手动增加的交换分区删除了。**
