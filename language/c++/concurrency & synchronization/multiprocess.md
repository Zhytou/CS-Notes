# 多进程编程

- [多进程编程](#多进程编程)
  - [进程 Process](#进程-process)
  - [进程的API](#进程的api)
  - [进程间通信](#进程间通信)

## 进程 Process

进程和程序组成了一对相依的概念。一个运行中的程序被称为一个进程；反过来讲，进程就是程序的一个实例。更准确的说，一个正在执行、且拥有独立的内存空间和资源的程序就是进程。

尽管我们说一个正在执行的程序是进程，但其实进程不只有正在运行一个状态，还包括：

- 新建状态（Created/New）：等待资源分配
- 就绪状态（Ready）：等待CPU资源
- 阻塞状态（Waiting）：等待其他资源，如I/O等
- 中止状态（Terminated）：进程执行结束

## 进程的API

**fork()**：做一份状态机的完整复制（内存、寄存器现场）。

`pid_t fork(void)`

在fork函数返回后，会有两个进程同时运行，其中一个是父进程，另一个是新创建的子进程。

在子进程创建成功后，它会复制父进程的代码和数据段，并且开始执行从fork函数返回的位置。也就是说，子进程会从fork函数返回的地方开始执行，而不是从程序的起始位置开始执行。

需要注意的是，在子进程中，fork函数的返回值是0，而在父进程中，fork函数的返回值是新创建的子进程的进程ID。因此，在父进程中可以通过判断fork函数的返回值来区分当前进程是父进程还是子进程。

**execve()**：将当前运行的状态机重置成另一个程序的初始状态。

`int execve(const char *pathname, char *const argv[], char *const envp[]);`

- pathname：指定要执行的程序的路径，可以是绝对路径或者相对路径。如果路径中不包含目录分隔符 /，则会从环境变量 PATH 中查找可执行文件。
- argv：一个以 NULL 结尾的字符串数组，用于指定新程序的命令行参数。第一个元素通常是程序的名称，后面的元素是程序的参数。注意，第一个元素是程序的名称，而非路径名。
- envp：一个以 NULL 结尾的字符串数组，用于指定新程序的环境变量。每个字符串都是一个形如 key=value 的键值对，表示一个环境变量的值。

execve函数执行成功时，不会返回到调用进程，而是直接将当前进程替换为新程序的进程。如果执行失败，则会返回-1，并设置相应的错误码。

**waitpid()**: 使父进程等待某个子进程结束后再继续运行。

`pid_t waitpid(pid_t pid, int *stat_loc, int options);`

- pid:指定等待的子进程ID。如果pid为-1,则等待任何子进程;如果pid为0,则等待本进程组中的任何子进程。
- stat_loc:用于存放子进程退出状态的指针。
- options:配置选项,如WNOHANG可以非阻塞等待。

waitpid函数让父进程进入阻塞状态，等待指定子进程结束，然后根据stat_loc的内容确定子进程的退出状态。

使用上述三个api，编写的多进程程序框架往往类似如下：

```c++
int main() {
    int pid = fork();
    if (pid == -1) {
      // 子进程创建失败
      fprintf(stderr, "fork error: %s\n", strerror(errno));
      exit(EXIT_FAILURE);
    } else if (pid == 0) {
      // 子进程execve
      char *exec_argv[] = {}
      // ... 
      execve("/usr/bin/xxx", exec_argv, NULL);
      // execve()仅执行失败返回
      perror("execve");   
      exit(EXIT_FAILURE);
    } else {
      // 父进程等待子进程结束
      int status;
      if (waitpid(pid, &status, 0) == -1) {
        fprintf(stderr, "waitpid error: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
      }
      if (WEXITSTATUS(status) != 0) {
        // 子进程执行失败
        printf("sub process error\n");
        continue;
      }
    }
}
```

## 进程间通信

在多进程程序中，各个程序的协作通常依靠`进程间通讯IPC(Interprocess Communication)`来完成。Linux系统中，IPC被分成三大类：

- 基于数据的IPC方法
  - 匿名 & FIFO 管道
  - 消息队列
  - 共享内存
  - 套接字
- 基于信号的IPC方法
  - IPC中唯一一种异步的通讯方法
- 基于同步的IPC方法
  - 信号量
  - 互斥量（一种特殊的信号量，只有锁定和非锁定两种状态）
