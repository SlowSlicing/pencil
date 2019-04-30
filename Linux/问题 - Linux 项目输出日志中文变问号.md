>　　这是由于本地语言环境导致，学要修改语言环境。

* Step 1

```
$ vim /etc/locale.conf
LANG="zh_CN.UTF-8"
```

* Step 2

```
$ vim ~/.bashrc

# 追加
export LANG='UTF-8'
export LC_ALL='zh_CN.UTF-8'
export LC_CTYPE='zh_CN.UTF-8'
```

```
$ source ~/.bashrc
```

* Step 3

　　重启项目


