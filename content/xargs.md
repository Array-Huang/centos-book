# 神奇的xargs命令

## xargs命令：将stdin转换成传入其它命令的参数
`xargs`命令的作用在于给别的命令传递参数，其一般配合管道符`|`来使用，把前一命令的stdout作为自己的stdin，再转换成`command line`形式的参数传给其它命令。

### xargs命令的语法
其一般出现的形式如下：
```
OtherCommand [options] | xargs [options] [TargetCommand [options]]
```
如：
```
find /tmp -name "*.log" -type f -print | xargs /bin/rm -f
```
上面这是`xargs`命令的常用场景，配合`find`命令，找到`/tmp`目录下所有日志文件并予以删除。

### xargs命令的意义
- 虽然管道能把别的命令的stdout作为下个命令的stdin传入，但毕竟并非所有的命令都接受stdin的，如`ls`；比较常见接受stdin的命令有`cat`、`less`；而`xargs`命令能转化stdin的命令正好弥补了这些不接受stdin的命令的不足。
- 对于大数据量的操作来说，如上面的例子，一次性删除大量文件，若直接使用`rm -f /tmp/*.log`，很可能会报错`/bin/rm Argument list too long`，而如果我们用上`xargs`命令，`xargs`会帮我们把待删的文件分批交给`rm`命令来执行。
- 某些命令针对`xargs`调用的方式进行了优化，达到更进一步的效果，如：
```bash
# ls | xargs ls
file1 file2 file3

dir1:
file4

dir2:
file5 file6 file7
```

### xargs命令的工作原理
想了解`xargs`命令的工作原理，其实很简单；`xargs`命令在不指定目标命令时，其默认目标命令实际上是`echo`：
```
# ls ./ | xargs echo
file1 file2 file3
# ls ./ | xargs
file1 file2 file3
# ls ./
file1   file2   file3
```
从以上命令的执行结果我们可以看到，`xargs`命令实际上就是**将所有空格、制表符和分行符都替换为空格并压缩到一行上显示，这一整行将作为一个字符串传入到目标命令中**。

以下两个命令实际上是等价的：
```bash
# ls ./ | xargs echo
file1 file2 file3
# echo 'file1 file2 file3'
file1 file2 file3
```

明白了`xargs`命令的工作原理，那么其实它所支持的options也很好理解，实际上就是如何将stdin传来的结果转化成不同的字符串，如处理分隔符的问题、转化成多个字符串分批执行的问题。
