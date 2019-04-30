### 运行级别概念的区分

| System V init 运行级别 | systemd 目标名称 | 作用 |
| :-: | :-: | :-: |
| 0 |  `runlevel0.target, poweroff.target` | 关机 |
| 1 |  `runlevel1.target, rescue.target` | 单用户模式 |
| 2 |  `runlevel2.target, multi-user.target` | 等同于级别 3 |
| 3 |  `runlevel3.target, multi-user.target` | 多用户的文本界面 |
| 4 |  `runlevel4.target, multi-user.target` | 等同于级别 3 |
| 5 |  `runlevel5.target, graphical.target` | 多用户的图形界面 |
| 6 |  `runlevel6.target, reboot.target` | 重启 |
| emergency |  `emergency.target` | 紧急 Shell |

### systemctl 管理服务的启动、重启、停止、重载、查看状态等常用命令区分

| System V init 命令（RHEL 6） | systemctl 命令（RHEL 7） | 作用 |
| --- | --- | --- |
| `service foo start` |  `systemctl start foo.service` | 启动服务 |
| `service foo restart` |  `systemctl restart foo.service` | 重启服务 |
| `service foo stop` |  `systemctl stop foo.service` | 停止服务 |
| `service foo reload` | `systemctl reload foo.service` | 重新加载配置文件（不终止服务）|
| `service foo status` | `systemctl status foo.service` | 查看服务状态 |

### systemctl 设置服务开机启动、不启动、查看各级别下服务启动状态等常用命令

| System V init 命令（RHEL 6） | systemctl 命令（RHEL 7） | 作用 |
| --- | --- | --- |
| `chkconfig foo on` | `systemctl enable foo.service` | 开机自动启动 |
| `chkconfig  foo off` | `systemctl disable foo.service` | 开机不自动启动 |
| `chkconfig foo` | `systemctl is-enabled foo.service` | 查看特定服务是否为开机自动启动 |
| `chkconfig --list` | `systemctl  list-unit-files --type=service` | 查看各个级别下服务的启动与禁用情况 |
