# 深度学习基础

- [深度学习基础](#深度学习基础)
  - [概述](#概述)
  - [线性](#线性)
    - [多层感知机](#多层感知机)
  - [卷积神经网络](#卷积神经网络)

## 概述

**机器学习**：

> 机器学习（Machine Learning）：一种实现人工智能的方法

监督学习：数据有标签，一般为回归或分类等任务

- 线性回归：通过学习数据中输入与输出之间的关系，对新数据进行预测。
- 逻辑回归：在线性回归的基础上，通过sigmoid函数将结果转化为概率值，用于二分类问题。
- 决策树：通过不断划分数据集的方式，建立决策树，用于分类和回归问题。
- 支持向量机：通过寻找数据集中最大间隔的超平面来实现分类。
- 朴素贝叶斯：基于贝叶斯定理的分类方法，适用于离散数据。

无监督学习：数据无标签，一般为聚类或若干降维任务

- 聚类：将数据点自动分组，通常用于数据探索和结构发现。
- 降维：通过降低数据维度来简化数据，通常用于数据可视化和数据压缩。
- 异常检测：寻找数据集中的异常值或离群点。
- 关联规则学习：挖掘数据集中的

强化学习：序列数据决策学习，一般为与环境交互中学习

- Q-learning：这是一种基于值函数的强化学习方法，通过不断更新状态-动作对的价值来学习最优策略。
- SARSA：这是一种基于策略的强化学习方法，通过不断更新当前状态-动作对和下一个状态-动作对的关系来学习最优策略。
- DDPG：这是一种基于确定性策略梯度算法的强化学习方法，用于在连续动作空间中学习最优策略。
- A3C：这是一种基于异步优化策略梯度算法的强化学习方法，用于并行学习最优策略。
- PPO：这是一种基于策略改进的强化学习方法，用于改进和稳定策略学习过程。

深度学习是一种人工智能方法，用于教计算机以受人脑启发的方式处理数据。这是一种机器学习过程，称为深度学习（Deep Learning），它使用类似于人脑的分层结构中的互连节点或神经元。它可以创建自适应系统，计算机使用该系统来从错误中进行学习并不断改进。

- CNN 卷积神经网络
- RNN 循环神经网络
- GNN 图神经网络
- LSTM 长短期记忆网络
- Transformer 模型

```txt
机器学习是一般概念,包含了许多算法,包括决策树、SVM、 logistic回归等。

深度学习是机器学习的一个分支,利用深层神经网络学习高维非线性特征。

深度学习模型往往包含多个隐藏层,每一层可以看做是一个更简单的模型。

第一个隐藏层的模型可以看作是一个线性分类器或回归器,相当于 logistic回归或线性回归。

随着层数增加,模型可以学习更高阶和非线性的函数关系,提取更复杂的特征。

所以可以这么说:

每一层神经网络实质上是一个简单的机器学习模型,比如线性回归。

但通过多层堆叠,深度学习模型可以学习高阶非线性关系,弥补机器学习算法的不足。

深度学习利用前馈网络结构解决机器学习算法难以处理的大规模模式识别问题。

所以总的来说,深度学习可以看作是利用神经网络实现的一种机器学习方法。它通过多层堆叠简单模型来学习高维特征,弥补了机器学习算法的一些限制。
```

## 线性

### 多层感知机

## 卷积神经网络
