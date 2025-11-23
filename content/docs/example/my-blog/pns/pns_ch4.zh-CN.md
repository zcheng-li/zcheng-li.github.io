---
title: "CH4.项目的规划调度"
weight: 10
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
* The processing time of job j is fixed and equal to $p_j$.
* unlimited number of machines in parallel.
* Besides a machine (and there is always one available) a job does not require any other resource.

在此前提下, 这个方法可以这样描述:

> Start at time zero with the processing of all jobs that have no predecessors. Every time a job completes its processing, start processing those jobs of which all the predecessors have been completed.

也就是说, 一个一个按顺序做.

但是为了更加正式的描述这个算法, 我们需要引入以下符号:
* $C^'_j$: job j 的最早可能完成时间.
* $S^'_j$: job j 的最早可能开始时间.
* $p_j$: job j 所需要的处理时间.
* {all k -> j}: 其余全部的 job 都是 j 的 前置.
  
因此, 我们可以拥有以下伪代码:

![alt text](/img/pns/a4_2_1.png)

这个方法, 我们称之为 **Forward Procedure**. 然而, 它虽然简单易懂, 但是它并不一定能带来最优的效率. 因为它有可能延迟某些 job 的开始. 

因此, 我们接下来要介绍另一个 CPM 算法: **Backward Procedure**. 这个算法利用了 Forward Procedure 的输出 $C_max$ 作为输入, 然后其它需要定义的符号如下:
* $C^{′′}_j$: job j 最晚可能完成时间. 
* $S^{′′}_j$: job j 最晚可能开始时间.
* ${j -> all k}$: job j 是所有其余 job 的前置.

![alt text](/img/pns/a4_2_2.png)

## 4.3 Program Evaluation and Review Technique (PERT)

## 4.4 Time/Cost Trade-Oﬀs: Linear Costs

## 4.5 Time/Cost Trade-Oﬀs: Nonlinear Costs

## 4.6 Project Scheduling with Workforce Constraints

## 4.7 ROMAN: A Project Scheduling System for the Nuclear Power Industry

## 4.8 Discussion
