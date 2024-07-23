# Shell

- [Shell](#shell)
  - [Shell and Terminal](#shell-and-terminal)
    - [Shell Basic Functions](#shell-basic-functions)
    - [命令执行 Commands Excution](#命令执行-commands-excution)
    - [环境变量 Environment Variable](#环境变量-environment-variable)
    - [重定向 Redirection](#重定向-redirection)
    - [管道 Pipeline](#管道-pipeline)
    - [组合命令 Command Composition](#组合命令-command-composition)
    - [别名 Alias](#别名-alias)
  - [Shell Programming](#shell-programming)
    - [变量 Variable](#变量-variable)
    - [参数 Parameter](#参数-parameter)
    - [运算符 Operator](#运算符-operator)
    - [流控制 Flow Control](#流控制-flow-control)
    - [Shell脚本例子](#shell脚本例子)
  - [Remote Shell(SSH)](#remote-shellssh)
    - [Asymmetric Cryptography](#asymmetric-cryptography)
    - [SSH登录例子](#ssh登录例子)

## Shell and Terminal

终端（Terminal）早期是指一种允许工程师向计算机发送指令并观察其运行结果的硬件设备，但现在我们所说的终端其实都是指终端模拟器（Terminal Emulator），即使用软件模拟文本输入输出的界面。换句话说，现代终端就是一种帮助人机交互的软件。那么，终端和同为交互软件的Shell的差别又是什么呢？

事实上，终端作为一个早期代指硬件的概念，现在和Shell已经逐步分不清了。但我们仍可以将终端当作只提供人与计算机之间的文本界面，允许用户输入命令并显示输出的一种软件，而把Shell当作一种命令解析器。当打开一个终端窗口时，默认启动Shell程序，并且将Shell的输入和输出都显示在终端上。总之，两者概念上的区别已经模糊了。

### Shell Basic Functions

Shell提供了很多功能和特性，下面将着重介绍其中最重要和常见的几个。

### 命令执行 Commands Excution

首先，Shell最基础也是最重要的功能——执行命令。一般来说，Shell中可执行命令分成Shell函数、Shell内置命令和普通可执行程序三类。

其中，最常见的就是普通可执行程序，以我们常用的ls、grep、mv等来自[GNU Coreutils](https://www.gnu.org/software/coreutils/)核心工具集的命令为例，它们实际上就是一组默认安装在Linux系统的程序。更多有关GNU Tools的介绍可以阅读我的另一篇博客[一文了解GNU Tools](https://zhytou.github.io/post/2023-6-27/gnu-tools/)。

除此之外，我们自己编写的可执行程序也可以使用Shell来执行，方法就是使用绝对路径或相对路径调用该程序。此外，以cd、pwd、alias等为代表的Shell内置命令也能够在Shell中被执行。

值得注意的是，有时候使用Shell执行命令也会出现错误。其中，除了命令名称或参数调用输入错误之外，最常见的就是未赋予命令可执行权限和无法查找到指定命令。前者可以使用chmod命令赋予相应权限，后者则涉及到Shell执行命令的流程，也就是:

- 将用户输入的命令解析为命令名称和参数。
- 查找命令名称对应的可执行文件名。
- 根据找到的可执行文件名，使用相应的路径执行命令。

因此，无法查找到指定命令涉及到Shell的又一个重要特点，即：环境变量。

### 环境变量 Environment Variable

Shell环境变量是一些可用于Shell脚本和命令执行中的信息。比如，某路径没有添加到PATH中，那么该路径下的命令就必须使用绝对路径或相对路径调用。除了代表可执行文件的搜索路径的PATH之外，常见的Shell环境变量还有：

- HOME:用户主目录
- SHELL:当前使用的Shell程序
- USER:当前用户名称

此外，我们可以使用export或env命令查看或设置当前Shell的环境变量。值得注意的是，使用export设置Shell环境变量只会对当前Shell进程有效。若希望长期修改某环境变量，则应该去修改相应Shell配置文件，或者修改操作系统的环境变量。因为，Shell在初始化时会继承操作系统的所有环境变量。

### 重定向 Redirection

Shell重定向是Shell的又一种强大特性，它可以将一个命令的输出重定向到另一个文件或者另一个命令的输入。比如：

```bash
# 将cmd的标准输出写入file.txt
cmd > file.txt

# 将cmd的标准输出追加到file.txt
cmd >> file.txt

# 将cmd的错误输出写入file.txt
cmd 2> file.txt

# 将file.txt作为cmd的输入
cmd < file.txt
```

另外，补充一个常用的重定向目的地`/dev/null`。当命令输出被重定向到它，那么所有输出都会被丢弃；而尝试将其作为输入时，则什么都读取不到。此外，下面还列出Linux的三大类I/O流，包括：

- 标准输出（stdout）是程序默认输出的流，它通常指向控制台或屏幕，在程序运行时会将输出信息打印到屏幕上。
- 标准输入（英文缩写为stdin）是程序默认从中读取输入的流，它通常指向键盘，在程序运行时会等待用户输入数据。
- 错误输出（英文缩写为stderr）是程序默认输出错误信息的流，它也通常指向控制台或屏幕，在程序运行时会将错误信息打印到屏幕上，以便程序员进行调试和故障排除。

### 管道 Pipeline

Shell管道是一种特殊的重定向，它可以将一个命令的输出重定向到另一个命令的输入。

```bash
# 将cmd1的输出作为cmd2的输入
cmd1 | cmd2
```

值得注意的是，Shell管道其实就是依靠操作系统中进程通信方式之一的管道实现的。

### 组合命令 Command Composition

除了使用重定向或者管道将两个命令组合起来，Shell中还提供了一些其他方法，包括：

- `cmd1 && cmd2`:如果前一个命令成功，则执行后一个命令。
- `cmd1 || cmd2`:如果前一个命令失败，则执行后一个命令。
- `cmd1 ; cmd2`:使用分号连接，直接将两个命令按顺序执行。
- `cmd1 & cmd2`:将两个命令同时放在在后台执行。

### 别名 Alias

Shell别名允许我们使用更简短或者更容易记忆的命令代替完整的命令。比如，在bash中我们可以使用`ll`命令代替`ls -alF`，因为在.bashrc中默认有一行`alias ll='ls -alF'`。

```bash
# 将cmd命名为new_cmd
alias new_cmd='cmd'
```

此外，还有两种与Shell别名类似的命令我想特别强调一下。首先是使用mv命令重命名文件，它和alias的不同之处在于mv是真正改变文件的名字，而alias不会修改原文件。其次是使用ln命令符号链接文件，它和alias的不同之处在于ln 创建了一个实际的链接文件，其它程序都可以读取和使用，而alias定义的别名只对Shell有效。

## Shell Programming

Shell作为一门编程语言，也和其他语言一样提供了运算符、参数、变量、函数、流控制等功能。下面介绍一些Shell作为一门编程语言的相关特性。

### 变量 Variable

在Shell脚本中，我们可以使用`VARIABLE_NAME=value`来定义变量、使用$加变量名来引用变量的值。比如，如果执行`for file in $(ls)`，Shell首先将执行ls命令，然后遍历得到的这些返回值。

此外，Shell还可以定义数组。比如：

```bash
# 使用空格分隔的列表
my_array=("apple" "banana" "orange")
# 使用索引分配值
my_array[0]="apple"
my_array[1]="banana"
my_array[2]="orange"
# 使用连续的索引进行赋值
my_array=([0]="apple" [1]="banana" [2]="orange")

# 使用数组中的值
fruits=("apple" "banana" "orange")
echo "First fruit: ${fruits[0]}"
echo "Second fruit: ${fruits[1]}"
echo "Third fruit: ${fruits[2]}"
```

### 参数 Parameter

我们可以在执行Shell脚本时，向脚本传递参数。脚本内获取参数的格式为：$n。其中，n代表一个数字，1为执行脚本的第一个参数，2为执行脚本的第二个参数，以此类推。此外，还有一些特殊字符代表一些特殊含义，具体如下：

- `$#`：参数个数
- `$$`：当前脚本进程号
- `$@`：所有参数

上述参数符号同样在Makefile中也很常用。

### 运算符 Operator

类似其他编程语言，Shell脚本语言也支持多种运算符，包括：

- 算数运算符
  - `+`加法
  - `-`减法
  - `*`乘法
  - `/`除法
  - `%`取余
  - `=`赋值
- 关系运算符
  - `-eq`判断是否相等
  - `-ne`判断是否不相等
  - `-gt`判断是否大于
  - `-lt`判断是否小于
  - `-ge`判断是否大于等于
  - `-le`判断是否小于等于
- 逻辑运算符
  - `&&`逻辑与
  - `||`逻辑或
- 字符串运算符
- 文件测试运算符

### 流控制 Flow Control

```bash
# if else fi
if condition1; then {
    command1
} elif condition1; then {
    command2
}
else {
    command3
} fi

# while
while condition; do { 
    command
} done
```

在此处再补充一下Shell中括号的区别，具体如下：

- `[]` 测试命令退出状态
- `()` 执行命令，获取输出
- `[[` 进行字符串和数学测试
- `((` 进行数学计算

其中，前两者是Shell最基本的测试方法,可以兼容大部分Shell。而后两者则只是Bash的扩展，只在Bash中可用。更多详细介绍可以使用指令`man sh`来阅读相关手册。

### Shell脚本例子

下面我会写一些Shell脚本来演示上述功能。

```bash
# 批量压缩文件夹
for file in /home/username/*; 
do 
    zip -r ${file}.zip $file
done

# 判断数据库是否启动
ps -ef | grep mysql
if [ $? -eq 0 ]; then
    echo "MySQL is running."
else
    echo "MySQL is not running!"
fi
```

## Remote Shell(SSH)

除了在本地使用Shell进行项目开发和调试，我们还常常需要在远程服务器上进行项目部署和运维。远程Shell（Remote Shell）也应运而生。而SSH(Secure Shell)正是一种实现安全远程Shell的方式。SSH是一种能够提供安全的远程登录和命令执行功能的安全协议，它的出现代替了Telnet以及其他非安全远程Shell。

### Asymmetric Cryptography

SSH使用非对称加密（Asymmetric Cryptography）来保证安全连接，与之相对的则是对称加密（Symmetric Cryptography）。二者的区别在于，前者加密和解密使用的密钥不同，加密使用公钥，解密则使用私钥；而后者则是加密解密使用同一密钥。

除了SSH使用了非对称加密技术之外，还有用于实现互联网的HTTPS和电子邮件的SMTPS等安全协议也基于该加密技术。

**Login Steps**：

- 登录者向主机发送一个登录请求，附带登录者自己的公钥。
- 主机收到请求后，会检查登录者的公钥是否在主机的authorized_keys文件中。
- 如果在authorized_keys中，则登录成功；反之，则登录失败。

**Password Login vs Public Key Login**：

使用SSH协议登录远程主机一般分成口令登录（Password Login）和公钥登录（Public Key Login）。其中，前者依靠口令，也就是密码验证登录者的身份；后者则是利用登录者公钥验证身份。

具体来说，公钥登录就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录，不再要求密码。

对于口令登录来说，它的麻烦之处就是需要每次输入密码。特别是，当你第一次登录远程主机时，系统甚至会给出下面的提示：

```txt
　　The authenticity of host 'host (12.18.429.21)' can't be established.

　　RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.

　　Are you sure you want to continue connecting (yes/no)?
```

这段话的意思是，无法确认host主机的真实性，只知道它的公钥指纹，问你还想继续连接吗？

当你成功连接一次之后，这个警告就不再出现了。因为这台远程主机的信息已经被保存在了文件$HOME/.ssh/known_hosts之中。下次再连接这台主机，系统就会认出它的公钥已经保存在本地了，从而跳过警告部分，直接提示输入密码。

### SSH登录例子

```bash
# 生成密钥对
# -t rsa/dsa 指定要创建的密钥类型。
# -C comment 提供注释
ssh-keygen -t rsa -C "xxx@gmail.com"

# 将公钥传送到远程主机上
ssh-copy-id username@remote_host

# 登录
ssh username@remote_host
```

除了可以使用Linux中的ssh命令来实现远程Shell之外，还可以使用一些SSH客户端，比如：PuTTY、WinSCP等。值得一提的是，我们可以使用SSH连接github服务器代替HTTPS连接来进行更快的克隆、推送和拉取分支操作，具体操作可以参考我的另一篇博客[Git Workflow&Tips](https://zhytou.github.io/post/2023-6-4/how-to-use-git/)中`use ssh instead of http`的部分。
