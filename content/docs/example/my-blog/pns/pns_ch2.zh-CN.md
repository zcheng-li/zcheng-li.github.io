---
title: "CH2.工业生产模型"
weight: 20
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---


# Chapter 2 Manufacturing Models

## 2.1 Introduction

## 2.2 Jobs, Machines, and Facilities

一些基本概念:

> The number of jobs is denoted by n and the number of machines by m. The subscripts j and k refer to jobs j and k. The subscripts h and i refer to machines h and i. The following data pertain to job j.

也就是:
* job 数: n.
* 机器数: m.

除此外, 还有:
* 处理时间: ($$p_{ij}$$): represents the time job j has to spend on machine i.
* 截止日期 ($$d_{j}$$).
* 权重 ($$w_j$$). job 的优先级 
 > It may represent the cost of keeping job j in the system for one time unit. The weight can be a holding or inventory cost, or it can be the amount of value already added to the job.

以上 4 个数据并称为 **静态数据** (static data). 因为它们不依赖于调度算法. 相反, 那些不能提前确定并且依赖于调度算法的数据称之为 **动态数据** (dynamic data). 比较重要的动态数据如下:

* Starting time $$S_{ij}$$: job j 开始在 机器i 上处理的时间点.
* Completion time $$C_{ij}$$: 完成时间.

调度模型的一个重要特征是它的 **机器配置** (machine configuration), 如下:
* Single Machine Models: job 只能去一个地方处理
* Parallel Machine Models: job 可以选择多台机器中的其中一个
* Flow Shop Models: 所有的 job 都会经过相同的处理
* Job Shop Models: job 通常有不同的处理
* Supply Chain Models

> a flow shop is a job shop in which each and every job
has the same route

![alt text](/img/pns/fig2_1.png)

![alt text](/img/pns/fig2_2.png)

## 2.3 Processing Characteristics and Constraints

job 的处理也有特征和对应的限制. 

* Precedence Constraints: 前后限制, 一个 job 必须完成一个处理个再进行下一个处理.
* Machine Eligibility Constraints: 当机器不完全一样时, 有些机器可能没有资格完成对特点 job 的处理.
* Workforce Constraints.
* Routing Constraints.
* Material Handling Constraints.
* Sequence Dependent Setup Times and Costs. 
* Storage Space and Waiting Time Constraints. 
* Make-to-Stockand Make-to-Order.
* Preemptions.
* Transportation Constraints.

## 2.4 Performance Measures and Objectives
一些关键的描述性能的 测量值 和 目标:

* Throughput and Makespan Objectives. 
  Throughput = output rate
  Makespan = 所有工作全部完成所需的总时间
  $$Cmax = max(C_1,...,C_n)$$
* Due Date Related Objectives. 
  * one objective: minimizing the maximum lateness
    Lateness: $$L_j= C_j−d_j$$
   $$ L_{max} = max(L_1,...,L_n)$$
  * anather objective: minimizing number of tardy jobs
    tardiness of job j: $$T_j = max(C_j−d_j,0)$$
    $$\sum^n_{j=1}\omega_jT_j$$
* Setup Costs.
* Work-In-Process Inventory Costs. (WIP)
* Finished Goods Inventory Costs. 
* Transportation Costs. 



