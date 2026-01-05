---
title: "CH2. Manufacturing Models"
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

This chapter introduces common **manufacturing models** used in planning and scheduling, focusing on how jobs, machines, and constraints are represented in a formal way.

---

## 2.2 Jobs, Machines, and Facilities

Some basic definitions:

> The number of jobs is denoted by *n* and the number of machines by *m*.  
> The subscripts *j* and *k* refer to jobs *j* and *k*.  
> The subscripts *h* and *i* refer to machines *h* and *i*.  
> The following data pertain to job *j*.

That is:
* Number of jobs: *n*
* Number of machines: *m*

In addition, we define:
* **Processing time** ($$p_{ij}$$): the time job *j* has to spend on machine *i*
* **Due date** ($$d_j$$)
* **Weight** ($$w_j$$): the priority of job *j*

> The weight may represent the cost of keeping job *j* in the system for one unit of time.  
> It can be a holding or inventory cost, or it can represent the amount of value already added to the job.

These four parameters are referred to as **static data**, since they do not depend on the scheduling algorithm itself.  
In contrast, data that cannot be determined in advance and depend on the scheduling decisions are called **dynamic data**.

Important dynamic data include:
* **Starting time** $$S_{ij}$$: the time at which job *j* starts processing on machine *i*
* **Completion time** $$C_{ij}$$: the time at which processing is completed

A key characteristic of a scheduling model is its **machine configuration**, which can be classified as follows:
* **Single Machine Models**: each job can only be processed by one machine
* **Parallel Machine Models**: a job can be processed by one of several machines
* **Flow Shop Models**: all jobs follow the same processing route
* **Job Shop Models**: different jobs may follow different routes
* **Supply Chain Models**

> A flow shop is a job shop in which every job has exactly the same route.

![alt text](/img/pns/fig2_1.png)

![alt text](/img/pns/fig2_2.png)

---

## 2.3 Processing Characteristics and Constraints

Job processing is subject to various characteristics and constraints:

* **Precedence constraints**: a job must complete one operation before starting the next
* **Machine eligibility constraints**: when machines are not identical, some machines may not be capable of processing certain jobs
* **Workforce constraints**
* **Routing constraints**
* **Material handling constraints**
* **Sequence-dependent setup times and costs**
* **Storage space and waiting time constraints**
* **Make-to-stock and make-to-order**
* **Preemptions**
* **Transportation constraints**

---

## 2.4 Performance Measures and Objectives

Several key performance measures and objectives are commonly used:

* **Throughput and makespan objectives**
  
  Throughput = output rate  
  Makespan = total time required to complete all jobs  

  $$C_{\max} = \max(C_1, \ldots, C_n)$$

* **Due-date-related objectives**
  * One objective is minimizing the **maximum lateness**

    Lateness of job *j*:
    $$L_j = C_j - d_j$$

    $$L_{\max} = \max(L_1, \ldots, L_n)$$

  * Another objective is minimizing the **number (or weighted sum) of tardy jobs**

    Tardiness of job *j*:
    $$T_j = \max(C_j - d_j, 0)$$

    $$\sum_{j=1}^{n} w_j T_j$$

* **Setup costs**
* **Work-in-process (WIP) inventory costs**
* **Finished goods inventory costs**
* **Transportation costs**