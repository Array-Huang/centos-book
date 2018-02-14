## SSH远程登录
Linux系统通过sshd(ssh daemon)服务实现远程登录的功能，其默认端口是22，此服务为Linux系统预装，并预设开机自启，因此不需要额外设置便能够实现Linux远程登录。

### Linux系统上的ssh客户端——openssh
Windosw系统上有许多软件可以实现ssh远程登录，比如说putty、SecureCRT、Xshell等，那么，我们在Linux系统上，应该使用哪个ssh客户端呢？这里推荐使用**openssh-clients**(简称**openssh**)。

### ssh的校验登录方式
ssh支持两种方法进行远程校验登录：使用密码登录和使用密钥登录。

#### 使用密码登录
使用命令`ssh user@host`，如`ssh root@192.168.80.128`，系统会进一步提示输入密码，正确输入后，你已经成功登录到远程服务器上了，你的一切操作都跟本地操作无异。

#### 使用密钥认证登录
由于密码登录存在易泄露、易破解的问题，因此一般主张使用ssh远程登录时，仅使用密钥进行登录，并关闭密码远程登录的权限。

所谓密钥认证，实际上是使用了一对加密的字符串：公钥(public key)和私钥(private key)，其一般使用**RSA**算法生成。公钥用于加密，任何人都可以看到其内容；而私钥用于解密，只有拥有者才能看到其内容；那么显而易见，即使坏人拿到公钥，也无法解读加密后的内容，因此可以保证内容的保密性。

下面介绍具体的操作：
1. 为本地本用户账号生成一个密钥对：`ssh-keygen`，系统会进一步提示你输入密钥存放的目标目录，按回车键保持默认即可（默认为`~/.ssh`）；接下来会提示你输入密钥的密码，一般留空即可（直接按回车键）；此时会告知你密钥对已成功生成。
2. 查看刚生成的公钥内容：`cat ~/.ssh/id_rsa.pub`。
3. 方法有两个：
    - 使用命令`ssh-copy-id user@host`即可把本地的公钥复制到远程服务器的**authorized_keys**文件上，并给相应的目录和文件设置好合适的权限，详情请看这里[SSH-COPY-ID](https://www.ssh.com/ssh/copy-id)。
    - 复制公钥的全部内容，粘贴到远程服务器的`~/.ssh/authorized_keys`里。远程服务器上如果不存在这个**authorized_keys**文件，则直接新建一个；如果已存在，则注意需要另起一行，加到此文件末尾。这一个过程也可使用以下命令进行操作：`$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`。需要注意，使用此手动方法，需要相应的设置好**authorized_keys**文件的权限，推荐为**600**。
4. 大功告成，从此你使用ssh远程登录，就不需要再输入密码了。

### 使用ssh远程执行命令并获得执行结果

#### 例子1：复制公钥
上述的`$ ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`就是一个很好的例子：
1. `ssh user@host`，表示登录远程主机；
2. 单引号中的`mkdir .ssh && cat >> .ssh/authorized_keys`，表示登录后在远程shell上执行的命令；
3. `mkdir -p .ssh`的作用是：如果用户主目录中的.ssh目录不存在，就创建一个；
4. `cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub`的作用是：将本地的公钥文件`~/.ssh/id_rsa.pub`，重定向追加到远程文件**authorized_keys**的末尾。

#### 例子2：查看远程主机是否运行进程httpd
```
$ ssh user@host 'ps ax | grep [h]ttpd'
```

### 退出当前ssh会话
使用`exit`命令或`logout`命令即可。