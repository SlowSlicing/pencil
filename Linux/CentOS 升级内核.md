# 关于内核的版本

| 性质 | 解释 |
| --- | --- |
| ml(mainline) | 主分支 |
| stable | 稳定版 |
| lt(longterm) | 长期维护版 |

* 版本命名格式：`A.B.C`

1. `A` 是内核版本号：版本号只有在代码和内核的概念有重大改变的时候才会改变。
2. `B` 是内核主版本号：主版本号根据传统的奇-偶系统版本编号来分配：奇数为开发版，偶数为稳定版。
3. `C` 是内核次版本号：次版本号是无论在内核增加安全补丁、修复bug、实现新的特性或者驱动时都会改变。

　　查看内核版本号：

```
# uname -r
3.10.0-957.el7.x86_64
# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

# 升级内核

## 方法一

* 使用 yum 升级

```
[root@k8s-node-1 ~]# rpm -import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
[root@k8s-node-1 ~]# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
获取http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
获取http://elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
准备中...                          ################################# [100%]
正在升级/安装...
   1:elrepo-release-7.0-3.el7.elrepo  ################################# [100%]
```

　　安装好仓库源之后查看：

```
[root@k8s-node-1 ~]# yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

![](https://pencil.file.lynchj.com/blogger/20190428110936.png)

　　这里查看的结果是，长期维护版本为 `4.4`，追新主线主线版本为 `5.0`，这里需要安装最新的。

```
# yum -y --enablerepo=elrepo-kernel install kernel-ml.x86_64 kernel-ml-devel.x86_64 
```

## 方法二

* 离线安装

　　先去下载一下官方的 rpm 包：

| 版本 | 连接 |
| --- | --- |
| CentOS 6 | [Download](http://elrepo.org/linux/kernel/el6/x86_64/RPMS/) |
| CentOS 7 | [Download](http://elrepo.org/linux/kernel/el7/x86_64/RPMS/) |

![](https://pencil.file.lynchj.com/blogger/20190428112001.png)

　　在下载文件所在目录执行：

```
# yum -y install `ls`
```

# 修改默认内核版本

　　升级完了内核版本不算完，当前使用的内核版本其实还是默认的版本，即使你 reboot 了，还是不顶用。因为启动顺序不对。先查看当前启动顺序：

```
# awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (5.0.10-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-ef56949a51de1447b8661f55966dba12) 7 (Core)
```

　　新内核 `5.0.10` 所在的顺序是 0，`3.10.0` 所在位置是 1，如果想要使新内核生效，要修改内核的启动顺序为 0。

```
# vim /etc/default/grub
```

![](https://pencil.file.lynchj.com/blogger/20190428112721.png)

　　运行 `grub2-mkconfig` 命令来重新创建内核配置：

```
# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.0.10-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.0.10-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-957.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-957.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-ef56949a51de1447b8661f55966dba12
Found initrd image: /boot/initramfs-0-rescue-ef56949a51de1447b8661f55966dba12.img
done
```

　　重启使内核生效：

```
# reboot

### 重启后查看
# uname -r
5.0.10-1.el7.elrepo.x86_64
```
