# 第七章：shell相关

## 区分登录/非登录shell，以及交互式/非交互式shell
### 登录/非登录shell
- 登录shell(login shell)指的是需要用户输入用户名和密码才能进去的shell，这是比较常用的。
- 非登录shell(non-login shell)则相反，不需要用户输入用户名和密码，例如：
    - 直接命令`bash`(不带`--login`参数)就是打开一个新的非登录shell。
    - 在图形界面（如Gnome或KDE）中打开一个“终端”(terminal)窗口程序也是属于打开了一个非登录shell。

### 交互式/非交互式shell
- 交互式shell(interactive shell)指的是shell等待自然人用户在终端上的输入，并且按照用户输入的命令立即执行且返回执行结果，这种操作模式被称为“交互式模式”，是我们最直观、最常用的操作方式。
- 非交互式shell(non-interactive shell)针对的并非自然人用户，它通常是以 shell script （即以 shell 的语法所写成的程序代码段）的方式执行。
    - 运行shell脚本程序时，系统将创建一个子shell。此时，系统中将有两个shell，一个是登录时系统启动的shell，另一个是系统为运行脚本程序创建的shell。
    - 当一个脚本程序运行完毕，它的脚本shell将终止，可以返回到执行该脚本之前的shell。
    - 从这种意义上来说，用户可以有许多 shell，每个shell都是由某个shell（称为父shell）派生的。

### 不同类型shell初始化时所执行的startup脚本不一样
> 对于Bash来说，登录shell（包括交互式登录shell和使用`–-login`选项的非交互shell），它会首先读取和执行`/etc/profile`全局配置文件中的命令，然后依次查找`~/.bash_profile`、`~/.bash_login` 和 `~/.profile`这三个配置文件，读取和执行这三个中的第一个存在且可读的文件中命令。除非被`–noprofile`选项禁止了。另外，由于`~/.bash_profile`一般也会带有执行`~/.bashrc`的代码段，因此`~/.bashrc`的内容也会被执行。

> 在非登录shell里，只读取 `~/.bashrc` （和 `/etc/bash.bashrc`、`/etc/bashrc` ）文件，不同的发行版里面可能有所不同。

从上可得，无论是登录shell还是非登录shell，最终都会执行`~/.bashrc`。

## 如何让变量突破shell脚本程序的上下文限制：`export`与`source`
在子 shell 中定义的变量只在该子 shell 内有效。如果在一个 shell 脚本程序中定义了一个变量，当该脚本程序运行时，这个定义的变量只是该脚本程序内的一个局部变量，其他的 shell 不能引用它。

有两种方法可以使子 shell 中的变量突破上下文限制，在其它 shell 中也能使用：
- 在脚本代码中使用`export`语句可以对已定义的变量进行输出，如下：
```
a=2333
export a
```
`export`命令将使系统在创建每一个新的子 shell 时定义这个变量的一个拷贝。这个过程称之为变量输出。
- 在交互式 shell 中使用`source`命令来执行脚本程序，系统将不会生成新的子 shell 来执行脚本程序，而是直接在当前的交互式 shell 中执行；那么，脚本程序中所有定义（或改变）的变量，都将在当前 shell 中起效。


## 如何配置Linux用户的工作环境，并在下次登录依然保持有效
当我们直接在命令行里执行配置工作环境的命令，如`alias`，赋值自定义变量等操作时，这些配置仅适用于当前登录的会话，若当前用户退出系统，则这些配置均失效。

首先需要明确的是，既然针对的是“Linux用户的工作环境”，那么实际上我们执行的是**登录交互式shell**；因此，若想要这些工作环境的配置在下次登录依然保持有效，可以这样做：


- “全局配置”，指的是为当前系统的每一个用户都设置一致的工作环境配置。全局配置的方式是：在`/etc/profile`文件中进行编辑或添加命令语句。
- “每个用户私有配置”，指的是针对Linux中具体某个用户进行特定的工作环境配置，其方式为：在`~/.bash_profile`中进行编辑或添加命令语句。

## shell变量规则
- 赋值变量的格式为`a=b`，其中a为变量名，b为赋给变量的值，需要注意的是，等号两边不能有空格。
- 变量名只能由字母、数字以及下划线组成，而且不能以数字开头。
- 当变量内容带有特殊字符（如空格）时，需要加上单引号，如`myname='array huang'`；如果变量内容中本来就含有单引号，此时就需要加双引号了，如`myname="'array huang'"`。
- 如变量中需要用到linux命令的执行结果，可以使用反引号，如
```
pwdResult=`pwd`
```
- 拼接变量的方法如下：`myname="$firstName"Huang`

## `grep`命令和`egrep`命令不一样的地方
`egrep`是`grep`的扩展版本，因此在功能上前者比后者更为强大，下面介绍它们不同的地方：
- `egrep`支持更多的正则表达式用法：
    - `?`(表示0个或1个指定的字符)；
    - `+`(表示至少1个指定的字符)；
    - `|`(逻辑或连接符)，如`'a|b'`，指的是匹配含有 a **或** b 的字符串；
    - `()`表示把多个字符合为1个整体来进行表达，通常搭配其它正则表达式符号来使用，如`'r(oo|at)o'`，指的是匹配含有 rooo 或 rato 的字符串；
- 在`{}`的用法上，`grep`的使用比较麻烦，需要加上转义字符`\`，如`'o\{2\}'`(表示匹配含有 oo 的字符串)；而`egrep`的使用则自然得多，同样的正则表达式，直接使用`'o{2}'`即可。

## shell脚本

### shell脚本的创建和执行
- shell脚本应以`#! /bin/bash`开头来表明该脚本使用的是 bash 语法，如果不写，也能正常运行，但就不符合编码规范了。
- shell脚本的执行有两种方法：
    - 执行命令`sh ./file.sh`，并且可以考虑加上`-x`参数，这样就能查看脚本每一步执行的命令与结果了，非常利于调试程序。
    - 直接执行`./file.sh`，这种方法使用的前提是用户拥有脚本的执行权限(即**x**权限)，需要注意的是新建脚本文件时一般是没有执行权限的，需要通过`chmod`进行修改。
    
因此一般我们直接用第一种方法即可。

### 脚本常用命令及语法
在 shell 脚本中，命令可看作是一般程序语言中的预设函数。

#### 时间
`date`，时间格式化和简单的计算（如“一天前”、“一小时后”等）。
- 命令(函数)的结果可以通过反引号来赋值给变量，如
```
nowDate=`date +"%Y-%m-%d"`
```

#### 数学运算
- 数学运算有特别的语法：
    - 用`$[]`给括起来(只支持整数运算)，如：
```
a=1
b=2
c=$[$a+$b+3]
echo $c // 结果是6
```
-
    - 使用`let`(只支持整数运算)：

```
var=1
let "var+=1"
echo $var
```
-
    - 使用`(())`(只支持整数运算)：
```
   var=1
   ((var+=1))
   echo $var
```
-
    - 使用bc(可以进行浮点数计算)：
```
   var=1
   var=`echo "$var+1"|bc`
   echo $var
```
其原理是：bc是linux下的一个简单计算器，支持浮点数计算，在命令行下输入bc即进入计算器程序，而我们想在程序中直接进行浮点数计算时，利用一个简单的管道即可解决问题。

#### 和用户交互
使用`read`命令可达到与用户交互的目的，它会把用户输入的字符串作为变量值，如：

```
read -p "please input a number:" x # 执行后等待用户输入并按回车确认
echo "$x"
```

#### 执行shell脚本时传参
在执行脚本时，我们可以通过对脚本传参（当然前提是脚本被设计成可以接受参数），来改变脚本的具体行为，以及得到以此计算出来的结果。

脚本内获取参数的格式为：$n。（n代表一个数字，，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……）。

举例说明，当前有一个名为**test.sh**的脚本：

```
#!/bin/bash

echo "Shell 输出脚本名称及参数";
echo "执行的脚本名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```

运行输出：

```
$ ./test.sh 1 2 3

Shell 传递参数实例！
执行的文件名：./test.sh
第一个参数为：1
第二个参数为：2
第三个参数为：3
```

#### 逻辑判断
共有以下3种语法：
- 不带`else`：
```
if [判断语句]; then
    command
fi
```
- 带`else`:
```
if [判断语句]; then
    command
else
    command
fi
```
- 带`elif`：
```
if [判断语句]; then
    command
elif [判断语句]; then
    command
else
    command
fi
```

##### if语句
###### 字符串判断

- str1 = str2　　　　　　当两个串有相同内容、长度时为真 
- str1 != str2　　　　　 当串str1和str2不等时为真 
- -n str1　　　　　　　 当串的长度大于0时为真(串非空) 
- -z str1　　　　　　　 当串的长度为0时为真(空串) 
- str1　　　　　　　　   当串str1为非空时为真

###### 数字的判断

- int1 -eq int2　　　　两数相等为真
- int1 -ne int2　　　　两数不等为真 
- int1 -gt int2　　　　int1大于int2为真 
- int1 -ge int2　　　　int1大于等于int2为真 
- int1 -lt int2　　　　int1小于int2为真 
- int1 -le int2　　　　int1小于等于int2为真

###### 文件的判断

- -r file　　　　　用户可读为真 
- -w file　　　　　用户可写为真 
- -x file　　　　　用户可执行为真 
- -f file　　　　　文件为正规文件为真 
- -d file　　　　　文件为目录为真 
- -c file　　　　　文件为字符特殊文件为真 
- -b file　　　　　文件为块特殊文件为真 
- -s file　　　　　文件大小非0时为真 
- -t file　　　　　当文件描述符(默认为1)指定的设备为终端时为真

###### 复杂逻辑判断

- && 　 　　　　　 与 
- ||　　　　　　　 或 
- !　　　　　　　　非

举例：
```
a=10
if [$a -lt 1] || [$a -gt 5]; then
    echo ok; # 输出ok
fi
```

#### case语句
语法如下：
```bash
case 变量 in
value1)
    command
    ;;
value2)
    command
    ;;
*)
    command
    ;;
esac
```
结合执行shell脚本时传入的参数，可以实现功能的切换：
```bash
#!/bin/bash
case $1 in
    start|s)      ## |表示or，在这里表示匹配start或s均可
    echo service is running
    ;;
    stop)
    echo service is stoped
    ;;
    reload)
    echo service is reload
    ;;
    *)
    echo xxxxx
    ;;
esac
```

#### for循环
语法如下：
```
for 变量名 in 循环的条件; do
    command
done
```

##### 循环条件
- 数字递进：`((i=1;i<=10;i++))`
- 列举一组字符串或数字，以空格分隔：`1 2 3 aaa bbb`
- 命令执行的结果：
```bash
for i in `ls`; do 
    echo $i is file name\!
done
```
- 变量的值：
```bash
list="rootfs usr data data2"
for i in $list; do
    echo $i is appoint
done
```
- 执行脚本时传入的参数：
```bash
#!/bin/bash

echo "number of arguments is $#" # $#表示参数的个数

echo "What you input is: "

for argument in "$@"; do # $@表示参数列表，此处也可以使用$*代替，$*则把所有的参数当作一个字符串
    echo "$argument"
done
```

#### while循环
语法：
```
while [循环(判断)的条件]; do
    command
done
```

注意此处的***循环(判断)的条件***与`if`语句中的判断条件是一样的。

#### 函数
##### 函数定义语法：
```bash
function 函数名() {
    command
    [return] # 可return也可不return
}
```

##### 函数的调用方法是：
```bash
#!/bin/bash
function show() {
    echo "hello , you are calling the function"
}
echo "first time call the function"
show
echo "second time call the function"
show
```

##### 如函数需要传入参数，则使用$1、$2……等取用：
```bash
#!/bin/bash
function show() {
    echo "hello , you are calling the function $1"
}
echo "first time call the function"
show first # 输出hello , you are calling the function first
echo "second time call the function"
show second # 输出hello , you are calling the function second
```

##### 函数中的关键字“return”可以放到函数体的任意位置，通常用于返回某些值，Shell在执行到return之后，就停止往下执行，返回到主程序的调用行，return的返回值只能是0~256之间的一个整数，返回值将保存到变量“$?”中。
```
#!/bin/bash
function test() {
    return 0
}

test
echo "$?" # 输出0
```

##### 如果函数在另外一个文件中，我们该怎么调用它呢？
我们可以使用`source`命令，比如 show 函数写在了function.sh里面了：
```bash
source function.sh
show
```

##### 函数的变量作用域
默认情况下，变量具有全局作用域，如果想把它设置为局部作用域，可以在其前加入local

例如：
```bash
local a=hello
```
使用局部变量，使得函数在执行完毕后，自动释放变量所占用的内存空间，从而减少系统资源的消耗，在运行大型的程序时，定义和使用局部变量尤为重要。

##### 函数的嵌套
函数可以进行嵌套，实例：
```bash
#!/bin/bash
function first() {
    function second() {
        function third() {
            echo "------this is third"
        }
        echo "this is the second"
        third
    }
    echo "this is the first"
    second
}
 
echo "start..."
first
```

#### shell脚本中的中断和继续

##### break
与其它程序语言一样，`break`是用来跳出当前的循环的，`break`后可加数字，如`break 2`表示跳出两层的循环。

##### continue
与其它程序语言一样。

##### exit
退出脚本，或者更准确地说，是退出当前运行脚本的shell（既然运行环境关闭了，那么脚本当然也不能继续执行下去了）。

在函数中使用`exit`，是会退出整个脚本的，因此若只是想退出函数，请使用`return`。

`exit`后可接数字作为状态码，如`exit 0`；一般来说，**0**表示脚本执行成功，其它状态码则表示各种可能的异常情况

## 别名
我们可以通过`alias`命令把一个常用的并且很长的指令另取名为一个简单易记的指令；如果不想用了，还可以使用`unalias`来取消。

- 使用`alias`命令，可以列出系统当前所有的别名。
- 使用`alias [命令别名]=['具体的命令']`可以自定义别名，例如：`alias aming='pwd'`。
- 使用`unalias`可以取消别名，例如`unalias aming`。

## 重定向
重定向分为“输入重定向”和“输出重定向”，重定向一般通过在命令间插入特定的符号来实现。

- `command1 > file1`，这个命令执行command1然后将执行后输出的内容存入file1。注意任何file1内的已经存在的内容将被新内容替代。如果要将新内容添加在文件末尾，请使用>>操作符。
- `command1 < file1`，执行command1，使用file1作为用来替代键盘的输入源。
- `command1 < infile > outfile`，同时替换输入和输出，执行command1，从文件infile读取内容，然后将输出写入到outfile中。

另外，上述的“输出内容”仅指命令执行的结果，若命令执行中出现错误，则错误信息需要另行处理：

- `command1 2> file1`，执行command1，然后将标准错误输出重定向到文件file1。
- 另外一个很有用的功能是将标准错误输出融合到标准输出中去，这样错误信息可以和其他普通的输出信息一起处理。例如：`command > file 2>&1`，这表示将command执行后的结果与错误信息均写入到file里。

## 管道符
管道符的标识为`|`，它用于将前一个指令的输出作为后一个指令的输入，格式如下：`command1 | command2`，举例说明：`yum list | grep zip | head -n 5`。