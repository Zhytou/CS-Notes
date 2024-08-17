# tmux

- [tmux](#tmux)
  - [简介](#简介)
  - [使用](#使用)
    - [会话管理](#会话管理)
    - [窗格操作](#窗格操作)
    - [窗口管理](#窗口管理)
  - [参考](#参考)

## 简介

**概述**：

Tmux是一个终端复用软件（Terminal Multiplexor），它可以在一个终端窗口中同时运行多个终端会话，并提供了多个窗口、面板、会话等功能，方便用户在终端窗口中执行多个任务。

**概念**：

会话（Session）广义上是指两个程序建立连接并发送交互的状态。在计算机的世界中，会话这个概念十分常见。比如，在计算机网络领域，会话是一种在服务器端保存数据的机制。它常常和插件（Cookie）、令牌（Token）等机制比较，具体介绍可以查看我的[笔记](https://github.com/Zhytou/CS-Notes/blob/main/basical/computer%20networking/cookie%20%26%20session%20note.md)。

本文想讨论的主要是终端会话（Terminal Session）。它往往由多个进程组（Process Group）构成，并且一定由一个控制终端（Controlling Terminal）连接着（或者说，会话就是由这个终端开启的），具体如下图。

![session & terminal & process group](https://jyywiki.cn/pages/OS/img/tty-session.png)

总结：一个终端对应这一个会话，也就是多个进程组。其中，有且只有一个前台进程组。当使用Ctrl+C或Ctrl+Z快捷键进行任务管理时，终端只会向前台进程组发送信号。这也就是为什么有些时候快捷键会失效的原因了。

更详细的解释资料可以使用命令`man 2 setpgid`阅读，它解释了进程组、会话和终端之间的关系。

Tmux的出现使得会话和终端的分离成为可能。它允许我们在一个终端中进行多个会话，并让每个会话对应终端中的一个窗口（Window）。此外，它还支持窗口任意的垂直和水平拆分，即形成多个窗格（Pane）。

![Session vs Window vs Pane](https://img-blog.csdnimg.cn/20191108150228720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

**功能**：

- 它允许在单个窗口中，同时访问多个会话。这对于同时运行多个命令行程序很有用。
- 它可以让新窗口"接入"已经存在的会话。
- 它允许每个会话有多个连接窗口，因此可以多人实时共享会话。
- 它还支持窗口任意的垂直和水平拆分。

## 使用

**安装**：

```bash
# 可以使用 apt | yum | brew 包管理器直接下载
sudo apt install tmux

sudo yum install tmux

brew install tmux
```

**启动与退出**：

```bash
# 键入tmux命令，打开一个新Tmux窗口并打开一个新会话。
tmux
# 按下Ctrl+d或者显式输入exit命令，就可以退出Tmux窗口。
exit
```

### 会话管理

**新建会话**：

在Tmux中，窗口可以使用编号和名称进行区分。其中，第一个启动的窗口编号默认是0，第二个窗口的编号是1，以此类推。

一般来说，我们更倾向于使用名称新建会话，因为编号区分会话不太直观。

```bash
# 新建一个自定义名称的会话。
tmux new -s <session-name>
```

此外，新建Tmux窗口不可以在Tmux中进行，即Tmux默认不允许嵌套。

**分离会话**：

```bash
# 在 Tmux 窗口中，按下Ctrl+b d或者输入tmux detach命令，就会将当前会话与窗口分离。
tmux detach
```

上面命令执行后，就会退出当前 Tmux 窗口，但是会话和里面的进程仍然在后台运行。退出Tmux窗口后，我们可以查看所有Tmux会话。

```bash
# 查看所有Tmux会话
tmux ls
```

**接入会话**：

```bash
# 使用会话编号
tmux attach -t 0

# 使用会话名称
tmux attach -t <session-name>
```

**杀死会话**：

```bash
# 使用会话编号
tmux kill-session -t 0

# 使用会话名称
tmux kill-session -t <session-name>
```

**切换会话**：

```bash
# 使用会话编号
tmux switch -t 0

# 使用会话名称
tmux switch -t <session-name>
```

### 窗格操作

Tmux可以将窗口分成多个窗格，每个窗格运行不同的命令。

**划分窗格**：

```bash
# 划分上下两个窗格
tmux split-window

# 划分左右两个窗格
tmux split-window -h
```

**移动光标**：

```bash
# 光标切换到上方窗格
tmux select-pane -U

# 光标切换到下方窗格
tmux select-pane -D

# 光标切换到左边窗格
tmux select-pane -L

# 光标切换到右边窗格
tmux select-pane -R
```

### 窗口管理

除了将一个窗口分成多个窗格之外，Tmux还允许使用多个窗口连接一个会话。

**新建窗口**：

```bash
tmux new-window

# 新建一个指定名称的窗口
tmux new-window -n <window-name>
```

**切换窗口**：

```bash
# 切换到指定编号的窗口
tmux select-window -t <window-number>

# 切换到指定名称的窗口
tmux select-window -t <window-name>
```

## 参考

[Tmux 使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)
