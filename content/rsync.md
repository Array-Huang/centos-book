## 备份工具rsync
rsync是Linux系统下最具代表性的数据备份工具，它具有以下特点：
- 不仅可以远程同步，还可以在本地进行同步。
- 增量更新，减少同步的流量。
- 可以在windows和mac下使用，能够做到跨平台使用。
- 可以很容易做到保持原来文件的权限、时间、软硬链接等等。
- 比较安全，可以使用scp、ssh等方式来传输文件，当然也可以通过直接的socket连接。

### rsync的命令格式
`rsync`命令的格式是：`rsync [OPTION]... SRC DEST`；其中，SRC和DEST既可以取本地目录/文件，也可以取远程目录/文件，如`rsync [OPTION]... SRC [USER@]HOST:DEST`或`rsync [OPTION]... SRC [USER@]HOST::DEST`。

SRC可以由多个文件组成，以此来实现一次性指定同步多个文件：`rsync [OPTION]... SRC1 SRC2 SRC3 DEST`。

### rsync的常用参数
- -a：归档模式，表示以递归方式传输文件，并保持所有属性，它等同于`-rlptgoD`，这是最常用的参数。另外，为灵活起见，`-a`后可跟一个`--no-OPTION`来表示关闭`-rlptgoD`中的任意一个参数，比如`-a--no-l`等同于`-rptgoD`。
- -r：表示以递归模式处理子目录。
- -v：表示打印同步的汇总结果信息。
- -l：表示保留软连接。
- -L：表示如果SRC中含有软连接文件，则取其指向的目标文件同步到DEST（当然软连接就不会同步过去了）。
- --delete：表示删除DST中SRC没有的文件。
- --exclude=PATTERN：表示指定排除不需要传输的文件。
- --progress：表示动态打印rsync同步过程中的状态以及最后的汇总结果信息（即包含了`-V`的效果）。
- -z：表示将会在同步传输过程中压缩。

### rsync远程同步
虽然rsync可以在本地的两个目录中进行同步，但其实rsync更常见的应用场景应该是在两台机器中进行远程同步（不论是通过局域网还是互联网）。

#### 通过ssh的方式进行远程同步
其命令格式(注意是只有一个**冒号**)如下：
- `rsync [OPTION]... SRC [USER@]HOST:DEST`
- `rsync [OPTION]... [USER@]HOST:SRC DEST`

在输入命令后，接下来的事情其实跟openssh的反馈十分类似：如没有通过密钥认证，则需要输入密码进行认证。认证通过后，接下来的rsync执行结果就跟在本地同步操作的没有两样了。

#### 通过rsync后台服务(rsyncd)的方式进行远程同步
这种方式的原理是在远程服务器上建立rsync服务器，并将本机看成是rsync客户端，这样我们就可以利用rsync客户端远程连接(via TCP)rsync服务器进行远程同步的操作了。

下面我们分**服务器端**和**客户端**两个部分来解释通过rsync后台服务(rsyncd)的方式进行远程同步的整个过程。

##### 服务器端

###### rsync配置文件
启动rsync后台服务rsyncd前，我们需要先准备一份rsyncd的配置文件——`/etc/rsyncd.conf`，该配置可控制rsync的权限、可操作范围、日志、可用模块等方方面面的内容。

配置文件由两个部分组成：全局变量和模块变量，但你也可以把模块变量写在全局变量里，作为模块变量的一份默认值。

下面列出一些常用的变量，具体的变量列表请查询[官方文档](https://download.samba.org/pub/rsync/rsyncd.conf.html)。

- 全局变量
    - pid file：指定pid文件路径(记录rsyncd的进程ID) 。
    - port：指定监听哪个端口，默认是873端口。
    - address：指定启动服务的IP，假如你的服务器有多个IP，那么可以指定其中的一个IP来启动rsyncd服务；如果不指定则默认在全部IP上启动
- 模块变量
    - path：指定该模块的根目录，客户端在访问该模块时不能超出该根目录的层级范围。另外，此变量可以达成某些骚操作，比如`path=/home/%RSYNC_USER_NAME%`，则当连接服务器端的rsync用户名为**test**时，该模块的根目录就自动设置为`/home/test`，但为了安全起见，不推荐这么干。
    - use chroot：值取**true**或**false**，表示在传输文件前，首先`chroot`到**path参数**所指定的目录下，这样做可以获得额外的安全防护；但是缺点是需要以root权限来启动rsyncd，并且，如果同步的内容里有软连接指向模块根目录以外的文件，则不能实现同步。
    - max connections：指定该模块最大的连接数，如果有超出此最大连接数的客户端试图连接，则会被告知**try later**；该值默认为0，即没有限制最大连接数。
    - log file：指定日志文件的路径，默认使用**syslog**。该日志文件意义非常重大，尤其是在架构rsync服务初期，可能会遇到相当多的问题，需要通过查看日志来定位问题所在。同时，即使为模块设置了**log file**，还是建议给全局设置一份，因为有些权限校验的异常，还是会被记录到全局日志文件里的。
    - read only：值取**true**或**false**，决定客户端是否能往服务器端上传/删除文件，默认为**false**。
    - write only：值取**true**或**false**，决定客户端是否能从服务器端下载文件。
    - list：值取**true**或**false**，决定客户端查询服务器端所有可用模块时是否显示，默认显示。
    - uid/gid：决定传输文件时以哪个身份（用户/组）进行传输，这关系到rsyncd是否有相应的文件系统权限来读取或修改文件。如果不设置此参数，则据此逻辑处理：如果rsyncd由超级用户启动，则以**nobody**这个系统用户的身份进行传输；否则，按启动rsyncd的用户身份进行传输。因此，如果想以root/root身份进行传输，则必须设置此参数为`uid=root;gid=root`。
    - filter/include from/include/exclude from/exclude：表示同步文件的白名单及黑名单；虽然在rsync客户端使用的`rsync`命令里也有对应的可选参数，但在服务器端设置的话，更具有强制性和隐蔽性（在客户端访问的时候，只会反馈"不存在"，而非"不允许访问"）。
    - auth users：表示身份验证的规则，不设置则表示所有rsync用户都可访问。指定对象的方式有两种：1. 指定用户名，此用户名并不一定要真实存在于Linux系统中；2. 指定用户组名（并在用户组名前加上`@`以区分开用户名），这个方式是依托于Linux系统的，因此客户端连接使用的用户名以及此处指定的用户组都必须是真实存在于Linux系统中。多个对象可用空格或逗号进行分隔（推荐使用逗号，因为如果使用空格来分隔，那么在用户名/组名中带有空格的情况下，会造成混淆）。另外可以使用参数`deny`/`ro`(read only)/`rw`(read write)来具体控制某个用户或某个组的访问权限，如：`auth users = , joe:deny, @Some Group:deny, admin:rw, @RO Group:ro`。
    - secrets file：指定一个声明`用户名:用户密码`或`@用户组名：用户组公共密码`的文件，该参数是`auth users`参数的配套，仅在已设置`auth users`参数的情况下生效，且没有默认值。另外，即使你在`auth users`参数中指定了用户组作为对象，也不一定要在本文件中声明`@用户组名：用户组公共密码`，完全可以罗列出用户组中所有用户的`用户名:用户密码`来满足需求。
    - strict modes：值取**true**或**false**，且默认值为**true**。如取**true**值，则`secrits file`参数所对应的文件，其文件系统权限必须设定为600。
    - hosts allow / hosts deny：表示rsync客户端的白名单及黑名单，可根据IP或IP段进行设置，且支持以空格为分隔符输入多个对象。

这里给出一份参考的rsyncd配置文件：
```
port=873
log file=/var/log/rsync.log
pid file=/var/run/rsyncd.pid

[test]
path=/root/rsync/test
use chroot=false
max connections=0 # no limitation
read only=false
list=true
uid=root
gid=root
auth users=rsyncuser
secrets file=/etc/rsyncd.passwd
```
###### 启动rsyncd
启动方式有以下两种：
- 独立的服务：`rsync --daemon --config=/etc/rsyncd.conf`。
- 由**inetd**在监听到请求时再行启动，这里不作阐述。

###### rsyncd自启
为保证rsyncd的可用性（比如在系统异常重启后依然可用），需要把rsyncd设置为开机自启，具体方法请查询`chkconfig`或`systemd`。

##### 客户端

##### 命令格式
在这种方式下的命令格式稍有区别：`rsync [OPTION]... RSYNCUSER@HOST::[MODULE[/SRC]] DEST`。多个对象的命令也有所区别：`rsync [OPTION]... RSYNCUSER@HOST::[MODULE1[/SRC1]] ::[MODULE2[/SRC2]] ::[MODULE3[/SRC3]] DEST`

##### 关于用户身份验证
若rsyncd设置了`auth users`，那么客户端在连接的时候就需要提供相应rsync用户的密码了：
- 直接用`rsync [OPTION]... RSYNCUSER@HOST::[MODULE[/SRC]] DEST`命令，接下来会提示需要输入密码。
- 指定存放rsync用户密码的文件路径：`rsync -avL --password-file=/etc/rsyncd.passwd RSYNCUSER@HOST::[MODULE[/SRC]]，此密码文件中，就只放密码：
```
# cat /etc/rsyncd.passwd
test123
```

### rsync进阶运用

#### rsync配合inotify实现文件目录实时同步
需求假设是这样的：
- 服务器A随着程序的执行，不断产生新的日志。
- 服务器B和C需要把服务器A上的日志拉取过来，并需要保证实时性。

如果我们用crond轮询来启动rsync发起同步，不仅实时性得不到保证，另外也很可能造成资源的浪费。

那么，就有了下面这套方案：
- 在服务器B和C上启动rsyncd。
- 在服务器A上利用inotify监控日志的产生，每逢有新日志产生inotify都会调用我们写好的shell脚本，这样我们就可以在脚本中，把服务器A作为rsync客户端，远程连接服务器B和C，达到把服务器A上新产生的日志推送到服务器B和C上的目的。

