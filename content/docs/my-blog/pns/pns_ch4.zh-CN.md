---
title: "CH4.项目的规划调度"
weight: 40
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Chapter 4 Project Planning and Scheduling

## 4.1 Introduction

本章的一些前提:
> This chapter focuses on the planning and scheduling of jobs that are subject
to precedence constraints. The setting may be regarded as a parallel machine
environment with an unlimited number of machines.

我们的优化目标:
> The objective is to minimize the makespan while adhering to the precedence constraints

两个限制:
* precedence constraints: 每个 job 的开始需要等待上一个 job 的完成.
* workforce constraints: 操作员可能短缺 (忙不过来)

job 先后顺序表示的两种方式:
![alt text](/img/pns/fig4_1.png)

对于前一种, 每个 arc 代表 job, 每个 node 代表一个 epoch.
对于后一种, 每个 node 代表 job, 每个 arc 代表 先后关系.



## 4.2 Critical Path Method (CPM)

首先声明必要的 assumptions:
* The processing time of job j is fixed and equal to $$p_j$$.
* unlimited number of machines in parallel.
* Besides a machine (and there is always one available) a job does not require any other resource.

在此前提下, 这个方法可以这样描述:

> Start at time zero with the processing of all jobs that have no predecessors. Every time a job completes its processing, start processing those jobs of which all the predecessors have been completed.

也就是说, 一个一个按顺序做.

但是为了更加正式的描述这个算法, 我们需要引入以下符号:
* $$C'_j$$: job j 的最早可能完成时间.
* $$S'_j$$: job j 的最早可能开始时间.
* $$p_j$$: job j 所需要的处理时间.
* {all k -> j}: job j 的全部前置.
  
因此, 我们可以拥有以下伪代码:

![alt text](/img/pns/a4_2_1.png)

这个方法, 我们称之为 **Forward Procedure**. 然而, 它虽然简单易懂, 但是它并不一定能带来最优的效率. 有的时候我们也需要适当地延迟某些 job 的开始, 以缓解机器生产的压力和库存成本之类的东西. 

因此, 我们接下来要介绍另一个 CPM 算法: **Backward Procedure**. 它在 Forward Procedure 的基础上将所有可能的开始时间全部推延, 同时保证总的的 makespan 不变. 这个算法利用了 Forward Procedure 的输出 $$C_max$$ 作为输入, 然后其它需要定义的符号如下:
* $$C^{\prime\prime}_j$$: job j 最晚可能完成时间. 
* $$S^{\prime\prime}_j$$: job j 最晚可能开始时间.
* {j -> all k}: job j 的全部后置.

![alt text](/img/pns/a4_2_2.png)

> So the forward procedure determines the earliest possible starting time and completion times as well as the minimum makespan.

后面又补充了几个概念:
* The diﬀerence between a job’s latest possible starting time and earliest possible starting time is the amount of slack, also referred to as **float**.
* A job of which the earliest starting time is equal to the latest starting time is referred to as a **critical job**.
* The set of critical jobs forms one or more **critical paths**. A critical path is a chain of non-slack jobs.


## 4.3 Program Evaluation and Review Technique (PERT)

与之前的章节不同, 这一节里, 每个 job 的处理时间现在是变量了. 相对的, 这些量的 **均值** \(u_j\), 和 **方差** \(\sigma_j\) 现在是已知的或者是可以被估计的. 而估计整的项目的 makespan, 我们可以用 **Program Evaluation and Review Technique (PERT)** 这一技术.

接着, 我们定义以下量:

\(p^a_j\) = the optimistic processing time of job j,

\(p^m_j\) = the most likely processing time of job j,

\(p^b_j\) = the pessimistic processing time of job j.

那么, 一个 job 的估计处理时间可以得到:

\[\hat{u}_j = \frac{p^a_j + 4 * p^m_j + p^b_j}{6}\]

基于这个估计处理时间, 我们就可以使用前面一节提到的 CPM. 

而 方差 则是:

\[\hat{\sigma}_j^2 = (\frac{p^a_j - p^b_j}{6})^2\]


## 4.4 Time/Cost Trade-Oﬀs: Linear Costs

在这一节里, 我们考虑成本的影响. 我们假设, 成本和处理时间有如下关系:

![alt text](/img/pns/fig4_6.png)

在进一步探讨相关算法之前, 我们需要先对模型进行化简. 

> The precedence graph can be extended by incorporating a **source** node (with zero processing time) that has an arc emanating to each node that represents a job without predecessors. In the same way each node that represents a job without successors has an arc emanating to a single **sink** node, which also has zero processing time. A **critical path** is a longest path from the source to the sink. 

我们记 \(G_{cp}\) 为一条或多条 Critical Path.

![alt text](/img/pns/fig4_7.png)

> A **cut set** in this subgraph is a set of nodes whose removal from the subgraph disconnects the source from the sink. A cut set is said to be **minimal** if putting any node back in the graph reestablishes the connection between the source and the sink (see Figure 4.7).


根据如上的表示, 我们可以有如下的启发式算法:

![alt text](/img/pns/a4_4_1.png)

example 4.4.2: 

![alt text](/img/pns/fig4_4.png)

> ps: Those jobs of which the earliest possible completion times are equal to the latest possible completion times are critical and constitute the critical path.

## 4.5 Time/Cost Trade-Oﬀs: Nonlinear Costs

## 4.6 Project Scheduling with Workforce Constraints

## 4.7 ROMAN: A Project Scheduling System for the Nuclear Power Industry

## 4.8 Discussion
