GitLab
SSH,GitLab,Docker,密码
[toc]
> &emsp;&emsp;在[上一篇笔记](https://blog.csdn.net/wo18237095579/article/details/81082349)中安装完毕的基础上发现一个坑，那就是 SSH 不好使，一直出现“我都已经配置好公钥了，还让输入密码”的问题。

## 正常配置 SSH 方式

&emsp;&emsp;1、在客户端生成密钥

```
[root@izj6c4t8bnakm3rm33nv53z ~]# ssh-keygen -t rsa -C "your.email@example.com" -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):【直接回车】
Enter passphrase (empty for no passphrase):【直接回车】
Enter same passphrase again:【直接回车】
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:jeR3krbOrSBkjQiQv85w061igR2XHNVLDOfChFSrSgc your.email@example.com
The key's randomart image is:
+---[RSA 4096]----+
|.. ..+=+.        |
|..  oo ++        |
| .E. o+.o.       |
|  oo+o *.o .     |
| o.=+.+ S * .    |
|o.*o.o.  o +     |
| =.o .. . .      |
|  = .  . + .     |
| . .      +..    |
+----[SHA256]-----+
```

&emsp;&emsp;2、把公钥配置到 GitLab 中

&emsp;&emsp;在客户端中查看公钥内容

```
[root@izj6c4t8bnakm3rm33nv53z ~]# cd /root/.ssh/
[root@izj6c4t8bnakm3rm33nv53z .ssh]# cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDvaqFYCYZ+4YJHR2AiDpmTd3JM+cOTQNQgnK+Mwk9RQQ6EEbDPkuSkLVvKtynng4eQAezv1Y3FW/S/1rVYvZUcNbz01sp/Fkh3OHz7lhJ1b8KRP6Cw/SryXr3sWSyb+MVAkoNCLBpTcnK43GlPmc2rm/ZC8NleOGTDpfdpPgjjNDcNj6N4W4/WF6+fZD00B2l9O4bJcGMPVA7omWI5mTPddGRwxHB1OmjW6oUboQ7E1/y3EvRDighusotcL/l6510OgLqZHDnVvrLT0VYfXfP7DoLoWwQngecB6DawEGVoWsjuP8NUXBXQXWQ8NG6eIb5AMk7Q/HDVO1ftecSmf63noQsLBtGTuRxdnTQwXKzYqmuoJJU6x/5nuujP0RppMvO6ZdpwqzLMQrME3gZt/oqmBZ02leggkaSVWty/48aJ6PfU9nZ3iEzDTTHSVxPufi4Um0obSzHu4DBmB9VTF0DT4dvOW7fDhQZDoKtVnnw//3VERCUrgmHoKSp2dgAy6RjqAj5lATuWCfdjp2Q6nKDSiKt2VrplmGCef0lks3jpF6MZAOZz+7bk4KI0kbzVH2jRliq5lPQ5tVnJ4IIaRrGEGEMXuLYG7+1i8Vq72JPmEgqizSlmHKfp+kvkR2q7/TPb2RcEJa4eNg2iiYuzv5Zcg4ypKLEtAwR+UxCEM9tjSw== your.email@example.com
```

&emsp;&emsp;把公钥配置在 GitLab 中

![配置公钥位置](http://img.lynchj.com/f8ab59cfe62744088902967ff43bceed.png)

![添加公钥](http://img.lynchj.com/eafeba33845f484f8a97a8562070d428.png)

&emsp;&emsp;本来这样就完事了，复制下 SSH 地址就能直接 clone 了，但是不行，如下：

```
[root@izj6c4t8bnakm3rm33nv53z .ssh]# git clone ssh://git@你的域名/你的GitLab用户名/test.git
Cloning into 'test'...
The authenticity of host '你的域名 (你的ip)' can't be established.
ECDSA key fingerprint is SHA256:dHEBUlufvduird8ANLoQPRZ5Ot2yoejcPXurl8Ih0qc.
ECDSA key fingerprint is MD5:3c:f3:bc:97:89:8a:9e:24:fa:f5:81:0c:bf:4d:ce:56.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '你的域名,你的ip' (ECDSA) to the list of known hosts.
git@git.chipparking.com's password: 【Excuse Me？怎么还让输入密码？】
```

## 解决问题

&emsp;&emsp;后来想到 GitLab 镜像启动后是占用容器的 22 端口，而我是使用宿主机的 4422 端口跟 GitLab 容器 22 端口进行的映射，主要是防止和我宿主机 22 端口冲突。想到问题的关键，解决就简单了，编辑 GitLab 配置文件，指定 SSH 端口为 4422 即可。

```
[root@izwz9cjwo2hniwcakynyg1z ~]# cd /data/docker/volumes/gitlab/
[root@izwz9cjwo2hniwcakynyg1z gitlab]# vim config/gitlab.rb
gitlab_rails['gitlab_shell_ssh_port'] = 4422

# 重启容器
[root@izwz9cjwo2hniwcakynyg1z gitlab]# docker container restart gitlab
```

&emsp;&emsp;之后再 GitLab 生成的 SSH 地址中就会自动加上 4422 端口，如：`ssh://git@你的域名:4422/用户名/test.git`

&emsp;&emsp;再次 clone 一次

```
[root@izj6c4t8bnakm3rm33nv53z .ssh]# git clone ssh://git@你的域名:4422/用户名/test.git
Cloning into 'test'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.
```

## 特殊情况

&emsp;&emsp;还有一种情况就是，上面的都做了还不起作用，有可能是没有开启 IP Forward 功能。

* 查看 IP Forward 功能

```
[root@iZ28o12qifoZ ~]# sysctl -q net.ipv4.ip_forward
# 0：未开启，1：已开启
net.ipv4.ip_forward = 0
```

* 开启 

```
[root@iZ28o12qifoZ ~]# sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
[root@iZ28o12qifoZ ~]# sysctl -q net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
