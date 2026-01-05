---
title: "CH4. Project Planning and Scheduling"
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

Some basic assumptions for this chapter:

> This chapter focuses on the planning and scheduling of jobs that are subject  
> to precedence constraints. The setting may be regarded as a parallel machine  
> environment with an unlimited number of machines.

Our optimization objective is:

> The objective is to minimize the makespan while adhering to the precedence constraints.

Two main constraints are considered:
* **Precedence constraints**: the start of a job must wait until its predecessor job has been completed.
* **Workforce constraints**: operators may be limited or unavailable (i.e., not enough manpower).

There are two common ways to represent job precedence relationships:

![alt text](/img/pns/fig4_1.png)

For the first representation, each **arc** represents a job, and each **node** represents an epoch.  
For the second representation, each **node** represents a job, and each **arc** represents a precedence relationship.

---

## 4.2 Critical Path Method (CPM)

We first state the necessary assumptions:
* The processing time of job *j* is fixed and equal to $$p_j$$.
* There is an unlimited number of machines operating in parallel.
* Besides a machine (which is always available), a job does not require any other resource.

Under these assumptions, the method can be described as follows:

> Start at time zero with the processing of all jobs that have no predecessors.  
> Every time a job completes its processing, start processing those jobs for which all predecessors have been completed.

In simple terms, jobs are processed as soon as they become available.

To describe this algorithm more formally, we introduce the following notation:
* $$C'_j$$: the earliest possible completion time of job *j*
* $$S'_j$$: the earliest possible starting time of job *j*
* $$p_j$$: the processing time required by job *j*
* {all *k* → *j*}: all predecessors of job *j*

Based on these definitions, we obtain the following pseudocode:

![alt text](/img/pns/a4_2_1.png)

This procedure is called the **Forward Procedure**.  
Although it is simple and intuitive, it does not always lead to optimal operational efficiency. In some cases, it may be beneficial to deliberately delay the start of certain jobs in order to reduce machine congestion or inventory-related costs.

Therefore, we introduce another CPM algorithm: the **Backward Procedure**.  
Based on the results of the Forward Procedure, this method postpones all possible starting times while keeping the overall makespan unchanged.

The Backward Procedure uses the output $$C_{\max}$$ from the Forward Procedure as input, and introduces the following notation:
* $$C^{\prime\prime}_j$$: the latest possible completion time of job *j*
* $$S^{\prime\prime}_j$$: the latest possible starting time of job *j*
* { *j* → all *k* }: all successors of job *j*

![alt text](/img/pns/a4_2_2.png)

> Thus, the forward procedure determines the earliest possible starting and completion times, as well as the minimum makespan.

Several additional concepts are then introduced:
* The difference between a job’s latest possible starting time and its earliest possible starting time is called **slack**, also referred to as **float**.
* A job whose earliest starting time is equal to its latest starting time is referred to as a **critical job**.
* The set of critical jobs forms one or more **critical paths**. A critical path is a chain of jobs with zero slack.

---

## 4.3 Program Evaluation and Review Technique (PERT)

Unlike the previous section, the processing time of each job is now treated as a random variable.  
Instead, the **mean** \(u_j\) and **variance** \(\sigma_j\) of the processing time are assumed to be known or estimable.

To estimate the overall project makespan under uncertainty, we use the **Program Evaluation and Review Technique (PERT)**.

We define the following quantities:
* \(p^a_j\): optimistic processing time of job *j*
* \(p^m_j\): most likely processing time of job *j*
* \(p^b_j\): pessimistic processing time of job *j*

The estimated processing time of job *j* is then given by:

\[
\hat{u}_j = \frac{p^a_j + 4p^m_j + p^b_j}{6}
\]

Based on this estimated processing time, the CPM procedure introduced earlier can be applied.

The variance of the processing time is given by:

\[
\hat{\sigma}_j^2 = \left(\frac{p^a_j - p^b_j}{6}\right)^2
\]

---

## 4.4 Time/Cost Trade-Offs: Linear Costs

In this section, the impact of costs is taken into account.  
We assume that cost and processing time are related as follows:

![alt text](/img/pns/fig4_6.png)

Before discussing the corresponding algorithms, the model is simplified as follows:

> The precedence graph can be extended by incorporating a **source** node (with zero processing time) that has an arc to each job without predecessors.  
> Similarly, each job without successors has an arc to a single **sink** node, which also has zero processing time.  
> A **critical path** is the longest path from the source to the sink.

We denote by \(G_{cp}\) one or more critical paths.

![alt text](/img/pns/fig4_7.png)

> A **cut set** in this subgraph is a set of nodes whose removal disconnects the source from the sink.  
> A cut set is said to be **minimal** if adding back any node restores the connection between the source and the sink (see Figure 4.7).

Based on this representation, the following heuristic algorithm can be applied:

![alt text](/img/pns/a4_4_1.png)

Example 4.4.2:

![alt text](/img/pns/fig4_4.png)

> ps: Jobs whose earliest possible completion times are equal to their latest possible completion times are critical and constitute the critical path.

---

## 4.5 Time/Cost Trade-Offs: Nonlinear Costs

---

## 4.6 Project Scheduling with Workforce Constraints

---

## 4.7 ROMAN: A Project Scheduling System for the Nuclear Power Industry

---

## 4.8 Discussion