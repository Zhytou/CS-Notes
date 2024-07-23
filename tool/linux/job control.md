# 任务管理

- [任务管理](#任务管理)
  - [信号 Signal](#信号-signal)
  - [会话 Session](#会话-session)
  - [多路复用器 Multiplexor](#多路复用器-multiplexor)

在终端中，我们能够使用Shell执行一些命令，同时也能查看执行情况并且中止或暂停执行，即：进行任务管理。比如，我们可以使用Ctrl+C或Ctrl+Z快捷键中止或暂停（在后台挂起）一个正在运行的进程，也能使用jobs、fg、bg等Shell内置命令查看所有进程、在前台恢复某个暂停进程、在后台恢复某个暂停进程。那么其背后的原理是什么呢？如果我fork了一份计算任务，使用Ctrl+C会杀死哪个任务呢？为什么有些时候会出现无法结束进程的情况呢？

## 信号 Signal

简单来说，终端依靠信号完成任务管理。信号（Signal）是一种进程间的通信方式。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。信号本质上是软件模拟的中断请求。常见的信号包括：

- SIGINT:程序终止信号，通常通过Ctrl+C组合键发送。
- SIGTERM:另一种更优雅、更通用的程序终止信号，可以使用`kill`命令发出。
- SIGSTOP:程序暂停信号。
- SIGTSTP:另一种程序暂停信号，通常通过Ctrl+Z组合键发送（其中的T代表Terminal，即终端版本的暂停信号）。

此外，大家熟悉的SIGSEGV也是一种信号，它通常发生在程序出现段错误时，即访问到无效地址（野指针）。

因此，当我们在终端中输入Ctrl+C时，它会发送一个SIGINT信号到它连接会话（Session）中的前台进程组（Foreground Process Group）。而前台进程在收到该信号后则会中止。那么，什么是所谓的会话呢？终端中的会话概念和计算机网络中的会话概念有什么关系呢？

## 会话 Session

会话（Session）广义上是指两个程序建立连接并发送交互的状态。在计算机的世界中，会话这个概念十分常见。比如，在计算机网络领域，会话是一种在服务器端保存数据的机制。它常常和插件（Cookie）、令牌（Token）等机制比较，具体介绍可以查看我的[笔记](https://github.com/Zhytou/CS-Notes/blob/main/basical/computer%20networking/cookie%20%26%20session%20note.md)。

本文想讨论的主要是终端会话（Terminal Session）。它往往由多个进程组（Process Group）构成，并且一定由一个控制终端（Controlling Terminal）连接着（或者说，会话就是由这个终端开启的），具体如下图。

![session & terminal & process group](https://jyywiki.cn/pages/OS/img/tty-session.png)

除此之外，要补充的是所有fork出的进程都会继承进程组号。这也就能回答本章开头的问题了——如果我fork了一份计算任务，使用Ctrl+C会杀死哪个任务呢？答案是，所有fork出的进程都中止。

总结：一个终端对应这一个会话，也就是多个进程组。其中，有且只有一个前台进程组。当使用Ctrl+C或Ctrl+Z快捷键进行任务管理时，终端只会向前台进程组发送信号。这也就是为什么有些时候快捷键会失效的原因了。更详细的解释资料可以使用命令`man 2 setpgid`阅读，它解释了进程组、会话和终端之间的关系。

## 多路复用器 Multiplexor

尽管早期一个会话一定对应着一个终端窗口，但随着终端多路复用器如Tmux等工具的出现，会话逐渐变得可以和终端分离了。不仅如此，它甚至允许我们分离当前会话并在将来重新连接。常见的多路复用器包括：

- Windows Terminal：Windows平台上超好用的终端工具。
- Tmux：Liunx/Unix平台上好用的多路复用器。关于它的详细介绍可以查看[Tmux](https://github.com/Zhytou/CS-Notes/blob/main/tool/tmux/tmux.md)。
- iTerm：Mac OS平台上很热门的终端模拟器。功能非常强大，可自定义性很高。支持分屏、标签等。
- Xterm：终端模拟器开山鼻祖。功能非常基础，但配置简单。
