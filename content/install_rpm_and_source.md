## RPM(Red Hat Package Manager)
- 系统中存在着一个关于RPM的数据库，它记录了安装包以及包与包之间的依赖关系。
- RPM包是预先在Linux机器上编译并打包的文件，安装非常快捷；但它也有一些缺点：
    - 安装环境必须与编译时的环境一致或者相当；
    - 包与包之间存在着相互依赖的情况下，卸载某个包时，需要先把系统里所有依赖该包的包进行卸载；虽然也可忽略依赖关系进行强制删除，但这样就会导致异常情况的发生。
- 安装RPM包使用命令`rpm -ivh filename`，其中：
    - `-i`，表示安装；
    - `-v`，表示可视化；
    - `-h`，表示显示安装进度；
- 升级RPM包使用命令`rpm -Uvh filename`，其中的`-U`就表示升级。
- 查询rpm包：
    - 查询是否已安装某个包使用命令`rpm -q packagename`，如`rpm -q zip`。另外，我们可以通过`rpm -qa`的命令来查询系统中所有已安装的包，并通过`grep`等方式进行二次搜索，如`rpm -qa | grep zip`。
    - 查询某个已安装的RPM包的详情：`rpm -qi packagename`，可得到版本号、安装时间、简介等信息。
- 卸载RPM包使用命令为`rpm -e packagename`。

## yum 工具
- Yum(Yellow dog Updater,Modified)是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。
- 列出所有可用的RPM包：`yum list`，由于数量众多，我们一般会进行二次搜索、筛选，如`yum list | grep zip | head -n 5`；此命令列出的信息里，主要有以下三列：
    - 第一列是包名，含平台信息。
    - 第二列是最新版本号。
    - 第三列是安装信息，如果已安装，则显示`@base`或`@anaconda`;如果未安装则显示`base`或`anaconda`；如果已安装但已有更新版本，则显示`updates`。
- 搜索RPM包的命令是`yum search str`，如`yun search zip`。
- 安装RPM包的命令是`yum install -y packagename`，如`yun install -y zip`，需要注意的是，虽然不加`-y`也是可以正常安装RPM包的，但是不加`-y`的话，如果该RPM包有依赖的包，就会一个一个轮流询问用户是否需要安装，那样子就太繁琐了，不如就加个`-y`全部默认安装，这也正是 yum 的一大特点嘛。
- 卸载RPM包的命令为`yum remove -y packagename`，加`-y`的原因同`yum install`。
- 升级RPM包的命令为`yum update -y packagename`，加`-y`的原因同`yum install`；另有`yum upgrade -y packagename`，作用与`yum update`类似都是更新本地系统里的该RPM包，不同在于`yum update`会先去更新软件支持列表（也称RPM源）。

## 安装源码包
安装源码包有3个主要步骤，分别是`./configure`、`make`、`make install`。

### 前置工作
安装源码包除了上述3个主要步骤，我们还需要前期的一些准备工作：
- 在官方站点下载源码包，并且基于约定俗成，把源码包放到`/usr/local/src`目录。
- 视源码包格式而定，挑选压缩工具进行解压。

### ./configure
这一步骤的主要作用就在于：
- 定制软件安装的功能/配置；
- 检查系统环境以及是否具有编译该源码包所需要的库；
- 生成 Makefile 文件；

关于软件可定制的功能/配置，我们可以通过命令`./configure --help`来进行查看，此时实际上并不会真的执行`./configure`，而是显示一个帮助文档。

最常用的可配置项莫过于`--prefix`，该配置项的意思是定义软件包的安装路径。

在确定好所有配置项后，我们可以执行形如以下的命令：`./configure --prefix=/usr/local/appache2`，此时就开始检测安装环境了，如果有问题，按照提示信息操作（如安装缺失了的库/包）即可。

如果执行成功，则可看到已生成了`Makefile`；另外也可以执行`echo $?`来验证操作结果，如果结果是0说明执行成功，否则就没有成功。

### make
生成`Makefile`后，需要进行编译，执行命令`make`，执行后，同样可用`echo $?`来验证操作结果。

### make install
通过`make`成功编译后，我们就可以执行安装了，命令为`make install`，执行后，同样可用`echo $?`来验证操作结果。

到此，该源码包便已安装完成了。 