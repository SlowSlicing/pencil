[toc]

# 安装 Docker

　　Docker 在 1.13 版本之后，从 2017 年的 3 月 1 日开始，版本命名规则变为如下：

| 项目 | 说明 |
| --- | --- |
| 版本格式 | YY.MM |
| Stable 版本 | 每个季度发行 |
| Edge 版本 | 每个月发行 |

　　同时 Docker 划分为 CE 和 EE。`CE 即社区版`（免费，支持周期三个月），`EE 即企业版`，强调安全，付费使用。

　　Docker CE 每月发布一个 Edge 版本 (17.03, 17.04, 17.05...)，每三个月发布一个 Stable 版本(17.03, 17.06, 17.09...)，Docker EE 和 Stable 版本号保持一致，但每个版本提供一年维护。

　　官方网站上有各种环境下的 [安装指南](https://docs.docker.com/install/)，这里主要介绍 Docker CE 在 Linux 发行版 Cent OS 上的安装。

# CentOS 安装 Docker CE

> 　　警告：切勿在没有配置 Docker YUM 源的情况下直接使用 yum 命令安装 Docker.

## 基本安装

### 准备工作

　　配置国内yum源

### 系统要求
　　Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 overlay2 存储层驱动）无法使用，并且部分功能可能不太稳定。

### 卸载旧版本

```
[root@localhost ~]# sudo yum remove docker \
									docker-client \
									docker-client-latest \
									docker-common \
									docker-latest \
									docker-latest-logrotate \
									docker-logrotate \
									docker-selinux \
									docker-engine-selinux \
									docker-engine
```

### 使用 yum 安装

　　执行以下命令安装依赖包：

```
[root@localhost ~]# sudo yum install -y yum-utils \
										device-mapper-persistent-data \
										lvm2
```

　　执行下面的命令添加 yum 软件源：

```
[root@localhost ~]# sudo yum-config-manager \
						--add-repo \
						https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo

# 官方源
# [root@localhost ~]# sudo yum-config-manager \
# 							--add-repo \
# 							https://download.docker.com/linux/centos/docker-ce.repo
```

　　如果需要最新版本的 Docker CE 请使用以下命令：

```
[root@localhost ~]# sudo yum-config-manager --enable docker-ce-edge
```

　　如果需要测试版本的 Docker CE 请使用以下命令：

```
[root@localhost ~]# sudo yum-config-manager --enable docker-ce-test
```

### 安装 Docker CE

　　更新 yum 软件源缓存，并安装 docker-ce 。

```
[root@localhost ~]# sudo yum makecache fast
[root@localhost ~]# sudo yum install docker-ce
```

### 启动 Docker CE

```
# 开机自启
[root@localhost ~]# sudo systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
# 启动Docker CE
[root@localhost ~]# sudo systemctl start docker
```

### 建立 docker 用户组

　　默认情况下， docker 命令会使用 ·Unix socket· 与 ·Docker 引擎·通讯。而只有 root 用户和docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker用户组。

　　建立 docker 组：

```
[root@localhost ~]# sudo groupadd docker
```

　　将当前用户（非root）加入 docker 组：

```
[dockerUser@localhost ~]# sudo usermod -aG docker $USER
```

　　退出当前终端并重新登录，进行如下测试。

### 测试 Docker 是否安装正确

```
[root@localhost ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

　　若能正常输出以上信息，则说明安装成功。

## 使用脚本自动安装

　　在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的`安装脚本`，CentOS系统上可以使用这套脚本安装：

```
[root@localhost ~]# curl -fsSL get.docker.com -o get-docker.sh
[root@localhost ~]# sudo sh get-docker.sh --mirror Aliyun
```

　　执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中。


## 添加内核参数

　　默认配置下，如果在 CentOS 使用 Docker CE 看到下面的这些警告信息：

```
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

　　请添加内核配置参数以启用这些功能。

```
[root@localhost ~]# sudo tee -a /etc/sysctl.conf <<-EOF
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

　　然后重新加载 sysctl.conf 即可

```
[root@localhost ~]# sudo sysctl -p
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

