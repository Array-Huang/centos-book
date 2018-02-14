# 第二章：CentOS的文件系统

## 修改文件的特殊属性
`chattr`，可修改文件的多种特殊属性：
- `a`，增加该属性后，只能追加不能删除，非root用户不能设定该属性
- `c`，自动压缩该文件，读取时会自动解压；
- `i`，增加后，使文件不能被删除、重命名、设定链接接、写入、新增数据

`lsattr`，该命令用来读取文件或者目录的特殊权限

## 在linux下搜一个文件
- `which`，找命令。
- `locate`，针对已生成的全局文件树索引对文件名进行搜索，但使用前需要先安装`mlocate`且执行`updatedb`来生成文件树索引；该命令仅支持按文件名进行搜索。
- `find`，遍历查找指定目录（不指定就针对整个系统进行查找）；该命令支持多种筛选条件（可按`与或否`的逻辑关系进行串联）进行查找，如：
  - 文件名，通过`-name`和`-iname`参数传入，支持通配符。
  - 所属用户，通过`-user`参数传入。
  - 所属组，通过`-group`参数传入。
  - 文件时间戳的相关属性，通过`-atime`(Access time)/`-ctime`(Change time)/`-mtime`(Modify time)参数传入，其中`-mtime`参数比较常用。
  - 文件类型，通过`-type`参数传入。
  - 文件大小，通过`-size`参数传入。

## 如何动态显示一个不停增加内容的文件
- 使用`tail -f`可实时追踪一个或多个文档的所有更新，这个功能在调试程序时非常好用：
```
tail -f /var/log/mail.log /var/log/apache/error_log
```

## 查看文件/目录占用磁盘大小
`du -sh filename`，解释：
- `-s`，表示只列出目录本身的数据。
- `-h`，系统自动调节单位。

## 压缩和解压缩
### gzip 压缩工具
- linux下压缩工具有多种，但最常用的是gzip，其它的使用起来也差不多。
- gzip只支持文件的压缩，若要压缩目录，则需要使用下述的`tar`打包工具。
- 压缩直接用`gzip sourcefile`，解压则用`gzip -d zipfile`。
- 使用 gzip 压缩的文件后缀一般为`.gz`。

### tar 打包工具
- tar 本身是一个打包工具，并不具有压缩功能，但可以配合压缩工具，一次性完成打包和压缩的任务；通常情况下我们也不会只打包不压缩，所以我们直接记住“一次性打包压缩”的参数即可：
    - `tar -czvf distfile sourcedir`，压缩打包sourcedir到disfile。
    - `tar -zxvf sourcefile`，解压解包fourcefile到当前目录。
- 解释一下上面命令用到的参数：
    - `-z`表示使用 gzip 压缩工具；其实还可使用其它压缩工具（如 bzip2 和 xz），但毕竟最常用的还是 gzip。
    - `-c`(`c` for compress)表示压缩打包，`-x`表示解压解包。
    - `-v`表示可视化。
    - `-f`后面跟文件名（即`-f filename`），表示压缩后的文件名为 filename，或当期需要解压文件 filename。
- tar 除了可以打包目录，还可以指定多个文件打包到一起：`tar -czvf files.tar.gz file1 file2 file3`。
- tar 命令支持查看（但不解压）压缩文件的内容，其参数为`-t`，但需要注意的是必须与`-f`同用，其用法为：`tar -tf file.tar.gz`。

### zip 压缩工具
- 对比起上述介绍的 gzip 和 tar，zip 的功能更为强大，它可以压缩(解压)文件和目录。
- 由于 zip 在 windows 系统上比较常用，因此如需与 windows 系统交换文件，可通过 zip 进行压缩，这样两边都可以识别。
- CentOS 默认不带 zip 命令，需要通过`yum install -y zip`进行安装。
- 压缩文件用`zip distfile sourcefile`，压缩目录则用`zip distfile sourcedir`。
- 需要注意的是，当压缩目录下还有二级目录甚至更多级目录时，zip 命令仅仅是把二级目录本身压缩而已，如果想要一并压缩二级目录下的文件及更多级目录，则必须加上`-r`，如`zip -r distfile sourcedir`。
- 解压文件并不用 zip 命令，而是用`unzip`命令，如`unzip file.zip`。
- 除了基本的压缩/解压功能外，zip 还提供更多进阶功能，如：使用密码进行加密；设置压缩级别；添加注释，等等。