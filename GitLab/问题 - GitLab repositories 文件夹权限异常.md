　　关键错误信息如下：

```
Failed asserting that mode permissions on "/var/opt/gitlab/git-data/repositories" is 2770
```

　　解决办法：

```
chmod 2770  /var/opt/gitlab/git-data/repositories
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```
