## 基础环境

> CentOS 7.4
> Docker

## 安装

　　1、创建数据目录

```
[root@iZ28o12qifoZ ~]# mkdir -p /data/docker/volumes/gitlab
[root@iZ28o12qifoZ ~]# chmod 777 /data/docker/volumes/gitlab
[root@iZ28o12qifoZ ~]# cd /data/docker/volumes/gitlab
[root@iZ28o12qifoZ git]# mkdir config
[root@iZ28o12qifoZ git]# mkdir logs
[root@iZ28o12qifoZ git]# mkdir data
```

　　2、拉取运行镜像

```
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 4443:443 --publish 8880:80 --publish 4422:22 \
    --name gitlab \
    --restart always \
    --volume /data/docker/volumes/gitlab/config:/etc/gitlab \
    --volume /data/docker/volumes/gitlab/logs:/var/log/gitlab \
    --volume /data/docker/volumes/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

　　查看运行情况

```
[root@iZ28o12qifoZ gitlab]# docker container ls
CONTAINER ID        IMAGE                     COMMAND             CREATED             STATUS                             PORTS                                                               NAMES
b7bb57b05fe5        gitlab/gitlab-ce:latest   "/assets/wrapper"   21 seconds ago      Up 20 seconds (health: starting)   0.0.0.0:4422->22/tcp, 0.0.0.0:8880->80/tcp, 0.0.0.0:4443->443/tcp   gitlab
```

　　3、访问 8880 端口

![IP 访问](http://img.lynchj.com/6a4bc27c2a3046d5ac5e3bde4ab811a1.png)

## 配置 Nginx

　　1、编写配置文件

```
[root@iZ28o12qifoZ nginx]# vim config/vhost/gitlab.conf
server {
        listen       80;
        server_name  你的域名;
        index index.html index.htm index.php;

        server_name_in_redirect off;

        location / {
                proxy_pass http://localhost:8880;
                proxy_redirect off;
                proxy_set_header Host $host:$server_port;
        }

        access_log  /data/logs/nginx/gitlab/gitlab.log;
}
```

　　2、测试访问

![域名访问](http://img.lynchj.com/9c8d087f9019420b99eee2ef564f21c1.png)

## 配置 HTTPS

　　1、首先要申请证书，阿里云上可以[申请免费的证书](https://common-buy.aliyun.com/?spm=5176.2020520163.cas.1.2c5b2b7aalok1d&commodityCode=cas#/buy)，此处不再多说。如图：

![申请免费 SSL 证书](http://img.lynchj.com/7bc3fa6d30194c4598a398f631eac04b.png)

　　2、配置 Nginx 

```
[root@iZ28o12qifoZ nginx]# vim config/vhost/gitlab.conf
### HTTPS server
server {
    listen 443 ssl;
    server_name 你的域名;
    index index.html index.htm;
    ssl_certificate   你的证书，要和域名匹配，阿里云上会提供下载 Nginx 版 .pem;
    ssl_certificate_key  你的证书，要和域名匹配，阿里云上会提供下载 Nginx 版 .key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_pass    https://127.0.0.1:4443;

    }

    access_log  /data/logs/nginx/gitlab/gitlab_ssl.log;

}
```

　　3、配置 GitLab

* 先把 SSL 证书 Copy 到 GitLab 数据目录

　　GitLab 配置时使用的是 `.crt`和`.key` 格式的证书文件，我们可以把原来的 `.pem` 先转换成 `.crt` 格式

```
[root@iZ28o12qifoZ gitlab]# cd /usr/local/nginx/ssl/gitlab/
[root@iZ28o12qifoZ gitlab]# openssl x509 -in 1532281108723.pem -out 1532281108723.crt
```

* 编辑 GitLab 配置文件

```
[root@iZ28o12qifoZ gitlab]# cp 1532281108723.crt /data/docker/volumes/gitlab/config/ssl/
[root@iZ28o12qifoZ gitlab]# cp 1532281108723.key /data/docker/volumes/gitlab/config/ssl/
[root@iZ28o12qifoZ nginx]# cd /data/docker/volumes/gitlab/
[root@iZ28o12qifoZ gitlab]# vim config/gitlab.rb
external_url 'https://你的域名'

nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/1532281108723.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/1532281108723.key"
```

* 最后重启容器

```
[root@iZ28o12qifoZ gitlab]# docker container restart gitlab
# 观察容器日志
[root@iZ28o12qifoZ gitlab]# docker logs -f gitlab
```

　　4、访问查看

![HTTPS 访问](http://img.lynchj.com/51acb72575db402d978cf415ea190c17.png)

> 　　为什么 Copy 到 `/data/docker/volumes/gitlab/config/ssl/`，因为我们在启动容器时，把宿主机 `/data/docker/volumes/gitlab/config` 目录和容器中 `/etc/gitlab` 目录进行映射，所以为了迎合 GitLab 配置文件中配置的路径，要 Copy 到`/data/docker/volumes/gitlab/config/ssl/`目录。

## 配置邮件服务

　　1、编辑 GitLab 配置文件

　　这里使用的是网易的免费企业邮箱

```
[root@iZ28o12qifoZ gitlab]# vim config/gitlab.rb

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.ym.163.com"
gitlab_rails['smtp_port'] = 994
gitlab_rails['smtp_user_name'] = "你的邮箱"
gitlab_rails['smtp_password'] = "你的密码"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = '你的邮箱'
gitlab_rails['smtp_domain'] = "ym.163.com"
```

　　2、重启 GitLab 容器

```
[root@iZ28o12qifoZ gitlab]# docker container restart gitlab
```

> 　　完毕
