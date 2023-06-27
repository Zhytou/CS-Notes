# GDB

- [GDB](#gdb)
  - [简介](#简介)
  - [使用](#使用)
  - [参考](#参考)

## 简介

**概述**：

GDB（GNU Debugger）是一个在Linux和其他类Unix系统上使用的常见调试工具。

**安装 & 配置**：

``` bash
# 安装gdb
sudo apt install gdb
# 编译保留调试信息
gcc -g test.c -o test
# 添加可能会用到的环境变量
export LD_LIBRARY_PATH=..
```

## 使用

``` bash
# 导入需要调试的可执行程序
file xxx

# 运行
run | r
# 单步步过执行
next | n
# 单步步入执行
step | s
# 单步汇编指令执行
step instruction | si
# 运行到下一断点
continue

# 添加断点
break func1 | break main.c:10
b func1 | b main.c:10
# 删除断点
delete 3

# 查看当前执行位置代码
list
# 查看当前执行位置汇编指令
disassemble | disas
# 查看当前调用链
backtrace | bt

# 查看所有断点信息
info break
# 查看当前寄存器信息
info registers
# 查看当前堆栈
info stack
# 查看当前调用堆栈帧帧
info frame
# 查看当前局部变量
info locals

# 打印某地址值
examine /[length][format] [address] | x /[length][format] [address]
x /1c  0x7ffff7d44060
# 打印某变量值
print xxx | p xxx
```

## 参考

- [Debugging Under Unix: gdb Tutorial](https://www.cs.cmu.edu/~gilpin/tutorial/)
- [GDB Tutorial: Advanced Debugging Tips For C/C++ Programmers](https://www.techbeamers.com/how-to-use-gdb-top-debugging-tips/)
- [Give me 15 minutes & I'll change your view of GDB [VIDEO]](https://www.youtube.com/watch?v=PorfLSr3DDI)
