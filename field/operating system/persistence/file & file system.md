# 文件 & 文件系统

- [文件 \& 文件系统](#文件--文件系统)
  - [文件和目录](#文件和目录)
    - [文件系统API](#文件系统api)
    - [创建和挂载文件系统](#创建和挂载文件系统)
  - [文件系统](#文件系统)
    - [元数据](#元数据)
    - [目录组织](#目录组织)
    - [空闲空间管理](#空闲空间管理)
    - [读取和写入](#读取和写入)
    - [缓存和缓冲](#缓存和缓冲)
  - [总结](#总结)
  - [参考](#参考)

## 文件和目录

### 文件系统API

### 创建和挂载文件系统

## 文件系统

文件系统主要关注两方面的问题：

- 第一个方面是文件系统的数据结构（Data Structure）。换言之，文件系统在磁盘上使用哪些类型的结构来组织其数据和元数据？
- 文件系统的第二个方面是访问方法（Access Method）。如何将进程发出的调用，如 open()、read()、write()等，映射到它的结构上？在执行特定系统调用期间读取哪些结构？改写哪些结构？所有这些步骤的执行效率如何？

### 元数据

文件系统最重要的磁盘结构之一是Inode（Index Node），几乎所有的文件系统都有类似的结构。它用于描述保存给定文件的所有信息，包括：文件类型（例如，常规文件、目录等）、大小、分配给它的块数、保护信息（如谁拥有该文件以及谁可以访问它）、一些时间信息（包括文件创建、修改或上次访问的时间文件下），以及有关其数据块驻留在磁盘上的位置的信息（如某种类型的指针）。我们也称这些信息为元数据（Meta Data）。实际上，文件系统中除了纯粹的用户数据外，其他任何信息通常都称为元数据。

设计 inode 时，最重要的决定之一是它如何引用数据块的位置。一种简单的方法是在inode 中有一个或多个直接指针（磁盘地址）。每个指针指向属于该文件的一个磁盘块。但这种方法难以支持寻址一个非常大的文件（例如，大于块的大小乘以直接指针数）。

**多级索引**：

为了支持更大的文件，人们提出了一种名为多级索引的方法。它的思路是在Inode中引入一种特殊的结构，即间接指针（Indirect Pointer）。间接指针它不是指向包含用户数据的块，而是指向包含更多指针的块，每个指针指向用户数据。因此，Inode可以有一些固定数量（例如12个）的直接指针和一个间接指针。如果文件变得足够大，则会分配一个间接块（来自磁盘的数据块区域），并将Inode的间接指针设置为指向它。

类似的，如果想要支持更大的文件，则可以使用双重间接指针，乃至三重间接指针。而这种不平衡树则被称为指向文件块的多级索引（Multi-level Index）方法。

许多文件系统使用多级索引，包括常用的文件系统，如 Linux ext2 [P09]和 ext3，NetApp的 WAFL，以及原始的 UNIX 文件系统。其

### 目录组织

### 空闲空间管理

### 读取和写入

### 缓存和缓冲

## 总结

文件系统是基于 block I/O 封装的数据结构。换句话说，文件系统是建立在块，即磁盘扇区或簇输入/输出模型之上的一个数据结构。

文件系统通过映射、索引等机制把文件和目录对应到底层的磁盘块，然后通过块I/O访问实际的数据。

## 参考

- [Operating Systems: Three Easy Pieces: Files and Directories](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/39.pdf)
- [Operating Systems: Three Easy Pieces: File System Implementation](https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/40.pdf)