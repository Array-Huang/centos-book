## Linux系统日志

### 核心系统日志文件——/var/log/messages
Linux的核心系统日志文件是`/var/log/messages`，它包含了以下内容：
- 系统启动时的引导消息
- I/O错误
- 网络错误
- 其它系统运行时发送的错误
- 单纯的操作记录

`/var/log/messages`是由**rsyslogd**这个守护进程生成的，如果**rsyslogd**被停止了，则系统将不会生成新的`/var/log/messages`。

### 安全日志

#### 使用`last`命令查看Linux系统的登录日志
```
# last | head
root     pts/0        192.168.80.1     Mon Feb  5 11:28   still logged in   
root     tty1                          Mon Feb  5 11:26   still logged in   
reboot   system boot  3.10.0-693.11.1. Mon Feb  5 11:26 - 11:28  (00:01)    
root     tty1                          Sun Feb  4 11:34 - crash  (23:51)   
```
上面的字段从左到右依次为账户名称、登录终端、登录客户端IP、登录时间段。

以上面这4条截取的日志来分析，它们是按时间从新到旧进行排列的，因此我们先从第四条开始分析。第四条日志其实是我在虚拟机以root账户登录后直接关闭虚拟机，因此我们可以发现，账户名称是root，登录终端是tty1（相当于本地登录了），没有客户端IP（因为是本地登录），登录时间段的末尾是**crash**（因为我不是正常的关机）。

根据上面的逻辑继续来分析就非常清晰了：
- 第3条是我重新开机虚拟机的日志；
- 第2条是开机成功后，我在本地使用root账户登录；
- 第1条是我在本物理机(windows系统)，开SecureCRT，同样使用root账户，进行远程登录（虽然两个系统在同一台物理机上，但虚拟机是相对独立的，因此其实看作这是两台机器，所以也就是远程登录而非本地登录了）。

另外再介绍一个与登录信息有关的日志文件——`/var/log/secure`：
```
Feb  5 11:28:14 localhost sshd[1462]: pam_unix(sshd:session): session opened for user root by (uid=0)
Feb  5 11:28:14 localhost sshd[1462]: Accepted password for root from 192.168.80.1 port 53703 ssh2
Feb  5 11:26:46 localhost login: ROOT LOGIN ON tty1
Feb  5 11:26:46 localhost login: pam_unix(login:session): session opened for user root by LOGIN(uid=0)
Feb  5 11:26:37 localhost sshd[1102]: Server listening on :: port 22.
Feb  5 11:26:37 localhost sshd[1102]: Server listening on 0.0.0.0 port 22.
Feb  5 11:26:35 localhost polkitd[535]: Acquired the name org.freedesktop.PolicyKit1 on the system bus
Feb  5 11:26:35 localhost polkitd[535]: Finished loading, compiling and executing 2 rules
Feb  5 11:26:35 localhost polkitd[535]: Loading rules from directory /usr/share/polkit-1/rules.d
Feb  5 11:26:35 localhost polkitd[535]: Loading rules from directory /etc/polkit-1/rules.d
Feb  4 21:36:42 localhost sshd[1434]: pam_unix(sshd:session): session closed for user root
```
对照着`last`命令的结果，`/var/log/secure`是不是挺容易看明白的呢？