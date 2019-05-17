# 预备

* 对准时间

```
$ yum install -y ntpdate
$ yes | cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ ntpdate us.pool.ntp.org
```

* 安装依赖包

```
$ sudo yum install -y -q openssh-server openssh-clients gcc make autoconf automake libtool pam-devel
```

* 获取 google 两步验证工具源码

```
$ git clone https://github.com/google/google-authenticator-libpam.git
```

# 安装

　　这里给安装到 `/usr/local/googleAuthenticator` 下。

```
$ cd google-authenticator-libpam/
$ ./bootstrap.sh
$ ./configure --prefix=/usr/local/googleAuthenticator
$ make -j 4 && sudo make install
make  all-am
make[1]: 进入目录“/root/google-authenticator-libpam”
  CC       src/pam_google_authenticator_la-pam_google_authenticator.lo
  CC       src/pam_google_authenticator_la-util.lo
  CC       src/pam_google_authenticator_la-base32.lo
  CC       src/pam_google_authenticator_la-hmac.lo
  CC       src/pam_google_authenticator_la-sha1.lo
  CC       src/google-authenticator.o
  CC       src/util.o
  CC       src/base32.o
  CC       src/hmac.o
  CC       src/sha1.o
  CC       src/base32_prog.o
  CCLD     base32
  CCLD     google-authenticator
  CCLD     pam_google_authenticator.la
make[1]: 离开目录“/root/google-authenticator-libpam”
make[1]: 进入目录“/root/google-authenticator-libpam”
 /usr/bin/mkdir -p '/usr/local/googleAuthenticator/bin'
  /bin/sh ./libtool   --mode=install /usr/bin/install -c google-authenticator '/usr/local/googleAuthenticator/bin'
libtool: install: /usr/bin/install -c google-authenticator /usr/local/googleAuthenticator/bin/google-authenticator
 /usr/bin/mkdir -p '/usr/local/googleAuthenticator/share/doc/google-authenticator'
 /usr/bin/install -c -m 644 FILEFORMAT README.md '/usr/local/googleAuthenticator/share/doc/google-authenticator'
 /usr/bin/mkdir -p '/usr/local/googleAuthenticator/share/doc/google-authenticator'
 /usr/bin/install -c -m 644 totp.html '/usr/local/googleAuthenticator/share/doc/google-authenticator'
 /usr/bin/mkdir -p '/usr/local/googleAuthenticator/share/man/man1'
 /usr/bin/install -c -m 644 man/google-authenticator.1 '/usr/local/googleAuthenticator/share/man/man1'
 /usr/bin/mkdir -p '/usr/local/googleAuthenticator/share/man/man8'
 /usr/bin/install -c -m 644 man/pam_google_authenticator.8 '/usr/local/googleAuthenticator/share/man/man8'
 /usr/bin/mkdir -p '/usr/local/googleAuthenticator/lib/security'
 /bin/sh ./libtool   --mode=install /usr/bin/install -c   pam_google_authenticator.la '/usr/local/googleAuthenticator/lib/security'
libtool: install: /usr/bin/install -c .libs/pam_google_authenticator.so /usr/local/googleAuthenticator/lib/security/pam_google_authenticator.so
libtool: install: /usr/bin/install -c .libs/pam_google_authenticator.lai /usr/local/googleAuthenticator/lib/security/pam_google_authenticator.la
libtool: finish: PATH="/sbin:/bin:/usr/sbin:/usr/bin:/sbin" ldconfig -n /usr/local/googleAuthenticator/lib/security
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/googleAuthenticator/lib/security

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
make[1]: 离开目录“/root/google-authenticator-libpam”
```

　　有一个需要用到的 PAM 文件 `pam_google_authenticator.so` 安装于目录：

```
/usr/local/googleAuthenticator/lib/security
# OR
/usr/local/googleAuthenticator/lib64/security
```

| 路径 | OS |
| --- | --- |
| /usr/local/googleAuthenticator/lib/security | Debian/Ubuntu/CentOS/RHEL/SLES |
| /usr/local/googleAuthenticator/lib64/security | OpenSUSE |

## 为可执行文件配置环境变量、导出库文件

```
$ lib_dir=${lib_dir:-'/usr/local/googleAuthenticator/lib'} && [[ -d '/usr/local/googleAuthenticator/lib64' ]] && lib_dir='/usr/local/googleAuthenticator/lib64' && bin_dir=${bin_dir:-'/usr/local/googleAuthenticator/bin'}
# 导出库文件并让系统重新生成缓存
$ sudo bash -c 'echo '"${lib_dir}"' > /etc/ld.so.conf.d/googleAuthenticator.conf' && sudo ldconfig -v

# 添加环境变量
$ sudo bash -c 'echo "export PATH=\$PATH:'"${bin_dir}"'" > /etc/profile.d/googleAuthenticator.sh'
$ source /etc/profile.d/googleAuthenticator.sh
```

## 生成并配置

```
$ google-authenticator

###  是否使用基于时间的认证令牌
Do you want authentication tokens to be time-based (y/n) y
Warning: pasting the following URL into your browser exposes the OTP secret to Google:
### 生成的二维码地址
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/root@localhost.localdomain%3Fsecret%3DVTOK7VV7RNOGBXN6BDKWGGM6OU%26issuer%3Dlocalhost.localdomain

### 这里有一个二维码，手机上的两步验证设备可扫描使用

### 安全码，在手機端Google Authenticator APP中的`Enter provided key`中用到
Your new secret key is: VTOK7VV7RNOGBXN6BDKWGGM6OU
### 认证码，扫描上方二维码之后会有
Enter code from app (-1 to skip): 092159
Code confirmed
### 5 个紧急验证码，用于当手机丢失后找回正确验证码使用
Your emergency scratch codes are:
  84327870
  86989146
  10941741
  53313537
  84108481

### 是否更新文件 ~/.google_authenticator，该文件默认不存在，会保留一些关键信息
Do you want me to update your "/root/.google_authenticator" file? (y/n) y

### 是否允许同一认证令牌用于多种用途
Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

### 基于时间登录，每个令牌默认的有效时间是30秒，足够抵消客户端到服务器之间的时间延迟
By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) y

### 单位时间内显示登录尝试的次数，以防暴力破解，默认每30秒内不能超过3次
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```

　　`~/.google_authenticator` 文件：

```
$ cat ~/.google_authenticator
VTOK7VV7RNOGBXN6BDKWGGM6OU
" RATE_LIMIT 3 30
" WINDOW_SIZE 17
" DISALLOW_REUSE
" TOTP_AUTH
84327870
86989146
10941741
53313537
84108481
```

# 配置 PAM

* 编辑文件 `/etc/pam.d/sshd`

```
$ vim /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       required     pam_google_authenticator.so
auth       include      postlogin
```

　　这里使用到了 `pam_google_authenticator.so`，是针对于目录 `/lib64/security/` 来说的，但是现在 `/lib64/security/` 下是没有此文件的，使用符号链接给转过去：

```
$ sudo ln -fs /usr/local/googleAuthenticator/lib/security/pam_google_authenticator.so /lib64/security/
```

✨因是原来是通过密码进行远程连接，请务必保证行 `auth substack password-auth` 不被注释，否则只验证 Google Authenticator 的 token 即可登录，不建议。

* 文件 `/etc/ssh/sshd_config` 的配置

```
$ vim /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
UsePAM yes
```

* 重启 sshd

```
$ sudo systemctl restart sshd
```

# 测试

```
Password:
Verification code:
Last failed login: Mon May 13 18:21:13 CST 2019 from 10.211.55.2 on ssh:notty
There was 1 failed login attempt since the last successful login.
Last login: Mon May 13 18:20:50 2019 from 10.211.55.2
```

* 问题 1
　　如果系統启用用了 `SELinux`，SSH 远程登录时会反覆出现要求输入认证码的情况，系统日志输出类似内容 sshd(pam_google_authenticator)[13445]: Failed to update secret file “/root/.google_authenticator”: Permission denied error: PAM: Authentication failure for root from $IP

　　可通过在 `auth required pam_google_authenticator.so` 后添加 secret 指令解決。

　　默认情况下，命令 `google-authenticator` 生成的文件路径在 `~/.google_authenticator`，为了能够让 PAM 读取含有所需 key 的文件 `~/.google_authenticator`，可先将其复制到 `~/.ssh/` 目录中（同时更改文件owner），再通过设置指令 `secret=${HOME}/.ssh/.google_authenticator`。这样即便已经启用了 SELinux，PAM 仍可以读取所需的文件内容，而不会被 SELinux 拦截。完整指令如下：

```
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator
# or
auth required pam_google_authenticator.so secret=${HOME}/.ssh/.google_authenticator [authtok_prompt=Verification code: ]
```

