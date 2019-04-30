> &emsp;&emsp;国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

&emsp;&emsp;**[Docker 官方提供的中国 registry mirror](https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror)**
&emsp;&emsp;**[阿里云加速器](https://cr.console.aliyun.com/#/accelerator)**
&emsp;&emsp;**[DaoCloud 加速器](https://www.daocloud.io/mirror#accelerator-doc)**

&emsp;&emsp;这里以 Docker 官方加速器为例进行介绍。

### Ubuntu 16.04+、Debian 8+、CentOS 7

&emsp;&emsp;对于使用 systemd 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```
{
	"registry-mirrors": [
		"https://registry.docker-cn.com"
	]
}
```

> &emsp;&emsp;注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

&emsp;&emsp;之后重新启动服务。

```
[root@localhost ~]# sudo systemctl daemon-reload
[root@localhost ~]# sudo systemctl restart docker
```

