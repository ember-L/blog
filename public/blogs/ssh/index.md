> **写在前面**：这篇文章主要介绍的是我做实验时候的工具，我日常做实验都使用的工具是Ubuntu20.4版本(VMWare虚拟机)+Windows10/11的**vscode**，这两个操作系统使用的是SSH进行连接。
>
> 为什么会使用**虚拟机+windows的vscode**的方式，这样不显得麻烦吗？这一点很好解释，主要是我对于vim/nvim不熟悉(没有go语言插件的自动补全，写go程序时也比较痛苦)，其次是在虚拟机中使用vscode配置过后会编码体验并不是很好，体验下来还是这种方式最好。
>
> 但是在xv6的实验中会涉及gdb的使用，所以在我的实验截图中，你会发现仍然会使用虚拟机的操作。

# SSH简介

> **SSH**（Secure Shell）远程登录协议

实现Linux和Windows这两个操作系统的**数据通信、文件共享**。只需要知道我们可以通过这个协议(像是一种连接的应用)，就可以实现在windows操作系统中对本地虚拟机内的linux文件进行编码了。

经常会使用的SSH工具就是Xshell和vscode，对于Xshell的话我看大部分博主都是用来操作云主机的，我使用的也比较少。

与Xshell同一家的还有一个软件叫做**Xftp**，主要功能就是通过ftp协议跨不同的操作系统传输文件，这个软件是肥肠实用的操作也简单，感兴趣的可以自行查看如何使用。

# SSH配置

接着就是在虚拟机中安装SSH软件了，具体命令如下所示。

```shell
# 1.更新资料列表
sudo apt-get update

# 2.安装openssh-server
sudo apt-get install openssh-server

# 3.查看ssh服务是否启动
sudo ps -e | grep ssh

# 4.如果没有启动，启动ssh服务
sudo service ssh start

# 5.查看IP地址
sudo ifconfig
inet addr:192.168.252.128
```



通过`ifconfig`得到的ip地址便是虚拟机的IP地址，这个地址会在后续用于ssh的通信。



# vscode配置

1. 打开VScode，我们在扩展中搜索SSH，安装`Remote - SSH`与`Remote - SSH：Editing Configuration Files`

![ssh1](https://ember-img.oss-cn-chengdu.aliyuncs.com/ssh1.png)

2. 安装之后，我们在左侧栏中打开远程连接的图标，将鼠标移到SSH TARGETS处点击“+”号进行添加

![ssh2](https://ember-img.oss-cn-chengdu.aliyuncs.com/ssh2.png)

或者是使用`ctrl+shift+p`命令输入SSH选择`Remote：Add New SSH Host...`

![image-20230525230128782](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20230525230128782.png)

3. 在SSH窗口中输入`ssh 用户名@虚拟机的ip地址` 这样的格式回车后会要求你**输入用户密码**这样便可以通过ssh连接到虚拟机了，

![image-20230525230254282](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20230525230254282.png)

选择`.ssh\config`将ssh信息记录到文件中保存，以后可以直接在ssh扩展处打开连接到虚拟机。

![image-20230526012425263](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20230526012425263.png)

像在windows一样我们可以在vscode中选择文件夹，再次输入用户密码，就将文件夹加载到工作区了。

![image-20230525231923745](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20230525231923745.png)

> 示例

以我自己连接为例输入`ssh lyj@192.169.19.128`命令，回车后选择打开linux系统的文件夹就可以愉快的进行编程辣！

![image-20230525230705147](https://ember-img.oss-cn-chengdu.aliyuncs.com/image-20230525230705147.png)

> 连接到linux系统后，我们可以继续打开扩展，安装扩展到虚拟机中，windows下已经安装的扩展是对linux主机下文件不生效的。安装后的插件存在于~/.vscode-server中

# 结果图

如下图我们之后连接好虚拟机后，就可以在上半区编写代码，下半区使用终端运行linux命令，编译运行文件了。

![ssh7](https://ember-img.oss-cn-chengdu.aliyuncs.com/ssh7.png)

## 虚拟机SSH拒绝连接问题

> 修改配置文件

```sh
vim /etc/ssh/sshd_config 
```

> sshd_config开启认证

```
# Authentication:
LoginGraceTime 2m
PermitRootLogin yes
StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```





## 番外：SSH免密登陆（multipass）

我们每次登录虚拟机时都得用`multipass shell hostname`命令，然后再输入密码，有点麻烦。

做点工作省略这些步骤。

先用`multipass shell hostname`登录到虚拟机中。

然后为 ubuntu 用户添加一个密码`sudo passwd ubuntu`（123456 就好）

对，没错，为了免密登录我们得先配置密码，出于安全考虑

然后修改 SSH 配置`sudo vi /etc/ssh/sshd_config`，把下面这三个都改为 yes

```powershell
PubkeyAuthentication yes
PasswordAuthentication yes
KbdInteractiveAuthentication yes
```

编辑好 SSH 配置文件之后要重启 SSH 服务`sudo service ssh restart`

做完上面这些工作之后就可以用 SSH 来登录虚拟机了，执行`ssh ubuntu@ip`命令后输入密码登录

### 配置免密登录（SSH）

先了解一下原理——非对称加密，我们用自己的电脑生成一对公钥和私钥，私钥保存好，公钥发给各个虚拟机，这样就能通过匹配公钥和私钥进行登录而不用输入密码

![](https://ember-img.oss-cn-chengdu.aliyuncs.com/ssh_pub.jpeg)

先用 ssh 生成一对公钥和私钥`ssh-keygen -t rsa -b 4096`，回车之后会提示为文件设置密码，建议直接回车不设密码

> gen：generate
>
> -t：type 密钥类型
>
> -b：bit 位数

生成密钥后到当前用户下的 `.ssh` 目录下 `ls` 查看公钥和私钥的文件。

我们需要把公钥复制到虚拟机中，Linux 和 Mac直接执行`ssh-copy-id username@remote_host`就可以自动把本地的公钥复制到虚拟机中，然鹅，Windows 系统不支持这个命令，有两个办法可以解决。

`ssh-copy-id` 命令用于将本地计算机上的公钥添加到远程主机的 `~/.ssh/authorized_keys` 文件中

1. 既然 Windows 不行那我们就找个 Linux，想想有Windows 上有什么是 轻量级 Linux 系统？GitBash，用 GitBash 进入到  `.ssh` 文件夹下，然后执行命令。需要你输入前面设置的密码。
2. `ssh-copy-id` 命令的本质是把本地的公钥添加到远程虚拟机的`~/.ssh/authorized_keys`文件里，PowerShell 行不通那就自己复制，把 `.ssh/id_rsa.pub` 的内容复制出来，到虚拟机中写入就好了。

### 配置命令别名

每次都要输入`ssh hostname@ip`也是有点麻烦，我们给这个命令起个别名更方便一些，如果你使用的macos或是linux可以使用`alias`设置别名

```sh
#macos
$ echo  alias master='ssh ubuntu@ip'  >> ~/.zshrc
#linux(以bash为例)
$ echo  alias master='ssh ubuntu@ip'  >> ~/.bashrc
```

