# linux命令

## netstat

netstat是控制台命令,是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息，并且可以按IP、TCP、UDP或ICMP协议进行统计。

常见的用法包括：

- netstat -a显示所有连接和侦听端口。
- netstat -n以数字形式显示地址和端口号。
- netstat -t或--tcp 显示TCP传输协议的连线状况。
- netstat -u或--udp 显示UDP传输协议的连线状况。

## awk

awk是一种文本处理工具，它可以从文件或标准输入中逐行读取数据，并根据指定的规则进行处理和输出。awk支持强大的文本处理和数据分析功能，可以用于提取、过滤、计算和格式化文本数据。

## grep

grep命令用于在文件或标准输入中搜索指定的文本模式，并输出匹配的行。它可以用于快速查找文件中包含特定字符串的行，以及进行模式匹配和筛选。

## tar

tar命令(tape archive)主要用于文件的打包压缩及解压。由于其最初的设计目的是为了将文件备份到磁带上，因而得名tar。它最常用的两个参数分别是-czvf和-xzvf，它们的例子如下：

```bash
# 压缩：将文件file1.txt, file2.txt, file3.txt压缩打包成file.tar.gz压缩包
tar -czvf file.tar.gz file1.txt file2.txt file3.txt
# 解压：命令将压缩包file.tar.gz文件解压缩
tar -xzvf file.tar.gz
```

其中，-c和-x分别代表create和extract，即创建和提取，分别对应了创建和解包的操作。至于其他三个共有的参数作用则分别如下：

- -f指定压缩文件名；
- -z指定使用gzip进行压缩；
- -v指定显示详细信息。

## chmod

在Linux/Unix中，文件权限可分为rwx，即读、写和执行。此外，同一个文件可针对文件所有者(Owner)、用户组(Group)和其它用户(Other Users)这三类不同的用户设置不同权限。其中，同组用户就是和文件所有者属于同一用户组的用户。搞清楚这些概念，我们就可以理解下面这些权限的含义了。比如：

``` txt
444 r--r--r--
600 rw-------
644 rw-r--r--
666 rw-rw-rw-
700 rwx------
744 rwxr--r--
755 rwxr-xr-x
777 rwxrwxrwx
```

chmod命令(chaneg mode)用于修改文件或目录对特定用户的权限。它可以使用字符或者数字对文件权限进行修改，比如：

```bash
# 给a.c文件的同组用户去掉写权限。
chmod  g-w a.c
# 将a.c文件的权限设置为属主、同组用户、其他用户都可读可写可执行。
chmod 777 a.c
```
