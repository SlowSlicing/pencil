Linux
SWAP,虚拟内存,RHEL,CentOS
&emsp;&emsp;`SWAP`（交换）分区是一种通过在硬盘中预先划分一定的空间，然后将把内存中暂时不常用的数据临时存放到硬盘中，以便腾出物理内存空间让更活跃的程序服务来使用的技术，其设计目的是为了解决真实物理内存不足的问题。但由于交换分区毕竟是通过硬盘设备读写数据的，速度肯定要比物理内存慢，所以只有当真实的物理内存耗尽后才会调用交换分区的资源。

&emsp;&emsp;交换分区的创建过程与前文讲到的挂载并使用存储设备的过程非常相似。在对`/dev/sdb` 存储设备进行分区操作前，有必要先说一下交换分区的划分建议：在生产环境中，交换分区的大小一般为真实物理内存的`1.5～2` 倍，为了更明显地感受交换分区空间的变化，这里取出一个大小为`5GB` 的主分区作为交换分区资源。在分区创建完毕后保存并退出即可：

```
[root@lynchj ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xb3d27ce1.
Command (m for help): n
Partition type:
p primary (1 primary, 0 extended, 3 free)
e extendedSelect (default p): p
Partition number (2-4, default 2):
First sector (4196352-41943039, default 4196352):
Using default value 4196352
Last sector, +sectors or +size{K,M,G} (4196352-41943039, default 41943039): +5G
Partition 2 of type Linux and of size 5 GiB is set
Command (m for help): p
Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xb0ced57f
Device Boot Start End Blocks Id System
/dev/sdb1 2048 4196351 2097152 83 Linux
/dev/sdb2 4196352 14682111 5242880 83 Linux
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

&emsp;&emsp;使用`SWAP 分区`专用的格式化命令`mkswap`，对新建的主分区进行格式化操作：

```
[root@lynchj ~]# mkswap /dev/sdb2
Setting up swapspace version 1, size = 5242876 KiB
no label, UUID=2972f9cb-17f0-4113-84c6-c64b97c40c75
```

&emsp;&emsp;使用`swapon 命令`把准备好的`SWAP 分区`设备正式挂载到系统中。我们可以使用`free -m` 命令查看交换分区的大小变化（由2047MB 增加到7167MB）：

```
[root@lynchj ~]# free -m
              total used free shared buffers cached
Mem:          1483  782  701  9      0       254
-/+ buffers/cache:  526  957
Swap:         2047  0    2047
[root@lynchj ~]# swapon /dev/sdb2
[root@lynchj ~]# free -m
              total used free shared buffers cached
Mem:          1483  785  697  9 	 0 		 254
-/+ buffers/cache:  530  953
Swap: 		  7167  0    7167
```

&emsp;&emsp;为了能够让新的交换分区设备在重启后依然生效，需要按照下面的格式将相关信息写入到配置文件中，并记得保存：

```
[root@lynchj ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Wed May 4 19:26:23 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root 						/ 				xfs 		defaults 	1	1
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b 	/boot 			xfs 		defaults 	1 	2
/dev/mapper 								/rhel-swap 		swap swap 	defaults 	0 	0
/dev/cdrom 									/media/cdrom 	iso9660 	defaults 	0 	0
/dev/sdb1 									/newFS 			xfs 		defaults 	0 	0
/dev/sdb2 									swap 			swap 		defaults 	0 	0
```
