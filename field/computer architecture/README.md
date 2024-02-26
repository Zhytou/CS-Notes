# 计算机体系结构

- [计算机体系结构](#计算机体系结构)
  - [指令集架构](#指令集架构)
  - [计算机组成](#计算机组成)
  - [微架构](#微架构)
  - [并行处理结构](#并行处理结构)
  - [参考](#参考)

![computer architecture](../img/computer_architecture.png)

## 指令集架构

指令集架构（Instruction Set Architecture）

- 发展历史
  - RISC\CISC
  - 存储管理
  - 运行权限
- 指令组成
  - 地址空间
  - 操作数
- 特权指令
- 软硬件协同
  - 应用程序二进制接口（Application Binary Interface，ABI）
  - 上下文切换

## 计算机组成

计算机组成（Computer Organization）

- 冯诺依曼结构：这是最常见的计算机体系结构,它将程序和数据统一存储在同一内存中。CPU可以同时执行和读取内存中的指令。这种结构具有程序可重入性。
- 哈佛结构：与冯诺依曼结构不同,它将程序和数据存储在两个独立的存储单元中。CPU只能执行存储在程序存储器中的指令,无法直接读取或写入数据存储器。代表产品为Harvard Mark I计算机。
- 菲林分类：根据指令集和数据单元进行分类:
  - SISD:单指令流单数据流,标量处理器
  - SIMD:单指令流多数据流,向量处理器
  - MISD:多指令流单数据流,很少见
  - MIMD:多指令流多数据流,像PC机一样工作
- 总线
- 计算机启动过程

## 微架构

微架构（Microarchitecture）

- 流水线设计
- 分支预测
- 高速缓存

## 并行处理结构

- 并发/并行/串行
- 并行计算机的内存架构
- 并行编程模型
- OpenMP/MPI

## 参考

- [国科大 胡伟武 计算机体系结构基础](https://foxsen.github.io/archbase/bookdown.pdf)
- [LLNL Introduction to Parallel Computing Tutorial](https://hpc.llnl.gov/documentation/tutorials/introduction-parallel-computing-tutorial##Abstract)
