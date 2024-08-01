# linux命令

- [linux命令](#linux命令)
  - [netstat](#netstat)
  - [awk](#awk)
  - [grep](#grep)
  - [tar](#tar)
  - [chmod](#chmod)
  - [crontab](#crontab)

## netstat

netstat是控制台命令,是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息，并且可以按IP、TCP、UDP或ICMP协议进行统计。

常见的用法包括：

- netstat -a显示所有连接和侦听端口。
- netstat -n以数字形式显示地址和端口号。
- netstat -t或--tcp 显示TCP传输协议的连线状况。
- netstat -u或--udp 显示UDP传输协议的连线状况。

## awk

awk命令(Awkward/Weinberger/Kernighan)是一种文本处理工具。它可以从文件或标准输入中逐行读取数据，并根据指定的规则进行处理和输出。awk支持强大的文本处理和数据分析功能，可以用于提取、过滤、计算和格式化文本数据。

一般来说，它的语法为下`awk [options] 'pattern {action}' file`。其中，pattern是匹配输入的模式；action是匹配后执行的动作；options是一些选项，包括：

- -f：指定包含awk脚本的文件。
- -V：显示awk的版本信息。
- -F：指定输入字段的分隔符。

## grep

grep命令(global regular expression)用于在文件或标准输入中搜索指定的文本模式，并输出匹配的行。它可以用于快速查找文件中包含特定字符串的行，以及进行模式匹配和筛选。

一般来说，一个grep命令的用法如下：`grep [options] pattern [file...]`。其中，pattern是用于指定正则表达式搜索模式；file是要搜索的文件，可以指定多个文件搜索，也可以使用通配符进行搜索；而options则可以使用以下选项：

- -i：忽略大小写，不区分大小写进行匹配。
- -v：反向匹配，输出不匹配的行。
- -r：递归搜索，搜索指定目录及其子目录下的文件。
- -l：只输出包含匹配模式的文件名，不输出匹配的具体行。
- -n：显示匹配行的行号。
- -c：统计匹配到的行号。

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

chmod命令(change mode)用于修改文件或目录对特定用户的权限。它可以使用字符或者数字对文件权限进行修改，比如：

```bash
# 给a.c文件的同组用户去掉写权限。
chmod  g-w a.c
# 将a.c文件的权限设置为属主、同组用户、其他用户都可读可写可执行。
chmod 777 a.c
```

## crontab

crontab命令(chrono table)用于设置周期性被执行的指令的命令。它可以给特定用户创建、编辑或删除其时间表，比如：

```bash
# -u user是指设定指定user的时程表，这个前提是你必须要有其权限(比如说是 root)才能够指定他人的时程表。如果不使用-u user的话，就是表示设定自己的时程表。
crontab [ -u user ] {-l | -e | -r}

# 查看当前用户的crontab时间表
crontab -l
# 编辑当前用户的crontab时间表
crontab -e
# 删除当前用户的crontab时间表
crontab -r
```

此外，还需要补充的是crontab时间表的格式。一般来说，它的格式如下：

```bash
f1 f2 f3 f4 f5 command
```

其中，f1是表示分钟；f2表示小时；f3表示一个月份中的第几日；f4表示月份；f5表示一个星期中的第几天；而command则表示要执行的程序。

具体来说，这些时间的设置方法包括：

- 当f1为*时表示每分钟都要执行program，其它时间也可以以此类推。
- 当f1为a-b时表示从第a分钟到第b分钟这段时间内要执行，其它时间也可以以此类推。
- 当f1为*/n时表示每n分钟个时间间隔执行一次，其它时间也可以以此类推。
- 当f1为a, b, c,... 时表示第a, b, c,... 分钟要执行，其它时间也可以以此类推。

最后，再来看几个具体的例子。

```bash
0 */2 * * * /sbin/service httpd restart  意思是每两个小时重启一次apache 

50 7 * * * /sbin/service sshd start  意思是每天7：50开启ssh服务 

50 22 * * * /sbin/service sshd stop  意思是每天22：50关闭ssh服务 

0 0 1,15 * * fsck /home  每月1号和15号检查/home 磁盘 

1 * * * * /home/bruce/backup  每小时的第一分执行 /home/bruce/backup这个文件 

00 03 * * 1-5 find /home "*.xxx" -mtime +4 -exec rm {} \;  每周一至周五3点钟，在目录/home中，查找文件名为*.xxx的文件，并删除4天前的文件。

30 6 */10 * * ls  意思是每月的1、11、21、31日是的6：30执行一次ls命令
```
