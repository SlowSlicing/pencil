[toc]

# 基本环境准备

## 环境变量

```
[root@android-package ~]# mkdir /usr/local/android

[root@android-package ~]# cat /etc/profile
# JAVA
export JAVA_HOME=/usr/local/jdk1.8.0_202
# ANDROID
export ANDROID_HOME=/usr/local/android
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/platform-tools:$JAVA_HOME/bin

[root@android-package ~]# source /etc/profile
```

## git

```
[root@android-package ~]# yum install -y git

# 克隆你的项目，如何 clone 这里就不说了
[root@android-package ~]# git clone ssh://git@git.chipparking.com:4422/Android/SmartParking.git
```

# 安装 sdk tools

　　官方[下载地址](https://developer.android.google.cn/studio)：

![](https://pencil.file.lynchj.com/blog/20190715133918.png)

```
[root@android-package ~]# wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
[root@android-package ~]# unzip sdk-tools-linux-4333796.zip
[root@android-package ~]# mv tools/ /usr/local/android/
[root@android-package ~]# source /etc/profile
[root@android-package ~]# sdkmanager --versio

# 下载 build-tools 和 platform
[root@android-package ~]# sdkmanager "build-tools;27.0.3" && sdkmanager "platform-tools" "platforms;android-25"
```

　　安装完成之后会在目录 `/usr/local/android/` 下有显示：

```
[root@android-package android]# ll
总用量 4
drwxr-xr-x 3 root root   20 7月  15 13:22 build-tools
drwxr-xr-x 2 root root   33 7月  15 13:21 licenses
drwxr-xr-x 3 root root   24 7月  15 13:22 platforms
drwxr-xr-x 5 root root 4096 7月  15 13:22 platform-tools
drwxr-xr-x 6 root root  205 7月  15 13:20 tools
```

# 安装 Gradle

　　Gradle 可以通过工具 [SDKMAN](https://sdkman.io/) 来安装，先安装 `SDKMAN`：

```
[root@android-package android]# curl -s "https://get.sdkman.io" | bash

                                -+syyyyyyys:
                            `/yho:`       -yd.
                         `/yh/`             +m.
                       .oho.                 hy                          .`
                     .sh/`                   :N`                `-/o`  `+dyyo:.
                   .yh:`                     `M-          `-/osysoym  :hs` `-+sys:      hhyssssssssy+
                 .sh:`                       `N:          ms/-``  yy.yh-      -hy.    `.N-````````+N.
               `od/`                         `N-       -/oM-      ddd+`     `sd:     hNNm        -N:
              :do`                           .M.       dMMM-     `ms.      /d+`     `NMMs       `do
            .yy-                             :N`    ```mMMM.      -      -hy.       /MMM:       yh
          `+d+`           `:/oo/`       `-/osyh/ossssssdNMM`           .sh:         yMMN`      /m.
         -dh-           :ymNMMMMy  `-/shmNm-`:N/-.``   `.sN            /N-         `NMMy      .m/
       `oNs`          -hysosmMMMMydmNmds+-.:ohm           :             sd`        :MMM/      yy
      .hN+           /d:    -MMMmhs/-.`   .MMMh   .ss+-                 `yy`       sMMN`     :N.
     :mN/           `N/     `o/-`         :MMMo   +MMMN-         .`      `ds       mMMh      do
    /NN/            `N+....--:/+oooosooo+:sMMM:   hMMMM:        `my       .m+     -MMM+     :N.
   /NMo              -+ooooo+/:-....`...:+hNMN.  `NMMMd`        .MM/       -m:    oMMN.     hs
  -NMd`                                    :mm   -MMMm- .s/     -MMm.       /m-   mMMd     -N.
 `mMM/                                      .-   /MMh. -dMo     -MMMy        od. .MMMs..---yh
 +MMM.                                           sNo`.sNMM+     :MMMM/        sh`+MMMNmNm+++-
 mMMM-                                           /--ohmMMM+     :MMMMm.       `hyymmmdddo
 MMMMh.                  ````                  `-+yy/`yMMM/     :MMMMMy       -sm:.``..-:-.`
 dMMMMmo-.``````..-:/osyhddddho.           `+shdh+.   hMMM:     :MmMMMM/   ./yy/` `:sys+/+sh/
 .dMMMMMMmdddddmmNMMMNNNNNMMMMMs           sNdo-      dMMM-  `-/yd/MMMMm-:sy+.   :hs-      /N`
  `/ymNNNNNNNmmdys+/::----/dMMm:          +m-         mMMM+ohmo/.` sMMMMdo-    .om:       `sh
     `.-----+/.`       `.-+hh/`         `od.          NMMNmds/     `mmy:`     +mMy      `:yy.
           /moyso+//+ossso:.           .yy`          `dy+:`         ..       :MMMN+---/oys:
         /+m:  `.-:::-`               /d+                                    +MMMMMMMNh:`
        +MN/                        -yh.                                     `+hddhy+.
       /MM+                       .sh:
      :NMo                      -sh/
     -NMs                    `/yy:
    .NMy                  `:sh+.
   `mMm`               ./yds-
  `dMMMmyo:-.````.-:oymNy:`
  +NMMMMMMMMMMMMMMMMms:`
    -+shmNMMMNmdy+:`


                                                                 Now attempting installation...


Looking for a previous installation of SDKMAN...
Looking for unzip...
Looking for zip...
Looking for curl...
Looking for sed...
Installing SDKMAN scripts...
Create distribution directories...
Getting available candidates...
Prime the config file...
Download script archive...
######################################################################## 100.0%
Extract script archive...
Install scripts...
Set version to 5.7.3+337 ...
Attempt update of interactive bash profile on regular UNIX...
Added sdkman init snippet to /root/.bashrc
Attempt update of zsh profile...
Updated existing /root/.zshrc



All done!


Please open a new terminal, or run the following in the existing one:

    source "/root/.sdkman/bin/sdkman-init.sh"

Then issue the following command:

    sdk help

Enjoy!!!
[root@android-package android]# source "$HOME/.sdkman/bin/sdkman-init.sh"
[root@android-package android]# sdk version
==== BROADCAST =================================================================
* 2019-07-13: Groovy 3.0.0-beta-2 released on SDKMAN! #groovylang
* 2019-07-11: Grails 4.0.0 released on SDKMAN! #grailsfw
* 2019-07-10: Gradle 5.5.1 released on SDKMAN! #gradle
================================================================================

SDKMAN 5.7.3+337
```

　　安装 Gradle：

```
[root@android-package android]# sdk install gradle 4.10.1

Downloading: gradle 4.10.1

In progress...

######################################################################## 100.0%

Installing: gradle 4.10.1
Done installing!


Setting gradle 4.10.1 as default.
[root@android-package android]# gradle -v

Welcome to Gradle 4.10.1!

Here are the highlights of this release:
 - Incremental Java compilation by default
 - Periodic Gradle caches cleanup
 - Gradle Kotlin DSL 1.0-RC6
 - Nested included builds
 - SNAPSHOT plugin versions in the `plugins {}` block

For more details see https://docs.gradle.org/4.10.1/release-notes.html


------------------------------------------------------------
Gradle 4.10.1
------------------------------------------------------------

Build time:   2018-09-12 11:33:27 UTC
Revision:     76c9179ea9bddc32810f9125ad97c3315c544919

Kotlin DSL:   1.0-rc-6
Kotlin:       1.2.61
Groovy:       2.4.15
Ant:          Apache Ant(TM) version 1.9.11 compiled on March 23 2018
JVM:          1.8.0_202 (Oracle Corporation 25.202-b08)
OS:           Linux 5.0.10-1.el7.elrepo.x86_64 amd64
```

# 打包 APK

```
[root@android-package SmartParking-master]# gradle clean
[root@android-package SmartParking-master]# gradle assembleRelease
```
