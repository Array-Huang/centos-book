# 第一章：CentOS的系统服务管理系统

## Linux系统服务管理
从CentOS7开始，CentOS的服务管理工具由SysV改为了systemd，但即使是在CentOS7里，也依然可以使用`chkconfig`这个原本出现在SysV里的命令。

Systemd的设计目标是，为系统的启动和管理提供一套完整的解决方案。

### chkconfig服务管理工具

#### 罗列chkconfig所管理的服务
使用`chkconfig --list`命令可以列出所有的服务及其在每个级别(run level)下的自启状态。
```
netconsole      0:off   1:off   2:off   3:off   4:off   5:off   6:off
network         0:off   1:off   2:on    3:on    4:on    5:on    6:off
```
这里我们只关心第3级和第5级：第3级表示完整的多用户模式，是标准的运行级，也即我们平常最常用的文字模式；第5级表示图形界面的管理模式。

需要注意的是，在CentOS7中，`chkconfig`只保留极少量的SysV服务，其它服务请使用systemd进行管理。

#### 使用chkconfig更改某服务在某级别下的自启状态
例如，使用`chkconfig --level 345 network off`即可关闭network这个服务在第3/4/5级中的自启；另外如果不传入参数`--level`，则默认针对级别2/3/4/5操作。

#### 为chkconfig添加/删除管理的服务项
简单例如：
```
# chkconfig --del network
# chkconfig --add network
```

### systemd服务管理工具

#### 罗列systemd所管理的服务
使用`systemctl list-units --all --type=service`：
```
# systemctl list-units --all --type=service
  UNIT                                   LOAD      ACTIVE   SUB     DESCRIPTION
  aegis.service                          loaded    active   running LSB: aegis update.
  agentwatch.service                     loaded    active   exited  SYSV: Starts and stops guest agent
  aliyun-util.service                    loaded    active   exited  Initial Aliyun Jobs
  aliyun.service                         loaded    active   running Aliyun Service Daemon
```

这些服务对应的启动脚本文件保存在`/usr/lib/systemd/system`。

#### systemd的基本概念
systemd把系统的各项资源（包括各个服务、设备等）都看作是**unit**，unit有许多种类，我们目前关心的是**service**和**target**。这里的service并不是什么新概念，因此只解释一下target：target是多个unit的组合，启动一个target也就相当于启动其中包含的所有unit；SysV中的run level在systemd里被target所取代，例如系统以多用户文字模式(runlevel 3)启动时，就会启动**multi-user.target**，而以图形界面模式(runlevel 5)启动时，则会启动**graphical.target**；target之间并非互斥的，因此可以同时启动多个target。

我们可以用`systemctl list-dependencies multi-user.target`来列举multi-user.target所包含的内容：
```
# systemctl list-dependencies multi-user.target
multi-user.target
● ├─aegis.service
● ├─agentwatch.service
● ├─aliyun-util.service
● ├─aliyun.service
● ├─brandbot.path
● ├─crond.service
● ├─dbus.service
● ├─network.service
● ├─ntpd.service
● ├─plymouth-quit-wait.service
● ├─plymouth-quit.service
● ├─rc-local.service
● ├─rsyslog.service
● ├─sshd.service
● ├─sysstat.service
● ├─systemd-ask-password-wall.path
● ├─systemd-logind.service
● ├─systemd-readahead-collect.service
● ├─systemd-readahead-replay.service
● ├─systemd-update-utmp-runlevel.service
● ├─systemd-user-sessions.service
● ├─basic.target
● │ ├─microcode.service
● │ ├─rhel-autorelabel-mark.service
● │ ├─rhel-autorelabel.service
● │ ├─rhel-configure.service
● │ ├─rhel-dmesg.service
● │ ├─rhel-loadmodules.service
● │ ├─paths.target
● │ ├─slices.target
● │ │ ├─-.slice
● │ │ └─system.slice
● │ ├─sockets.target
● │ │ ├─dbus.socket
● │ │ ├─systemd-initctl.socket
● │ │ ├─systemd-journald.socket
● │ │ ├─systemd-shutdownd.socket
● │ │ ├─systemd-udevd-control.socket
● │ │ └─systemd-udevd-kernel.socket
● │ ├─sysinit.target
● │ │ ├─dev-hugepages.mount
● │ │ ├─dev-mqueue.mount
● │ │ ├─kmod-static-nodes.service
● │ │ ├─ldconfig.service
● │ │ ├─plymouth-read-write.service
● │ │ ├─plymouth-start.service
```

可以看出这其中就包含了不少target，比如**basic.target**，因此target是可以嵌套的。

#### systemd常用命令
```
# systemctl enable crond.service // 让某个服务开机自启(.service可以省略)
# systemctl disable crond // 不让开机自启
# systemctl status crond // 查看服务状态
# systemctl start crond // 启动某个服务
# systemctl stop crond // 停止某个服务
# systemctl restart crond //重启某个服务
# systemctl reload * # 重新加载服务配置文件
# systemctl is-enabled crond // 查询服务是否开机启动
```

## systemd功能介绍
Systemd 是 Linux 的系统工具，用来启动守护进程，已成为大多数发行版的标准配置。

它的设计目标是，为系统的启动和管理提供一套完整的解决方案。
根据 Linux 惯例，字母d是守护进程（daemon）的缩写。 Systemd 这个名字的含义，就是它要守护整个系统。

从CentOS7开始，CentOS的服务管理工具由SysV改为了systemd，但即使是在CentOS7里，也依然可以使用`chkconfig`这个原本出现在SysV里的命令。