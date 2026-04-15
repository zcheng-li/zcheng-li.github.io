---
title: "CH1. Planning and Scheduling: Roles and Impact"
weight: 1
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## 1.1 Planning and Scheduling: Role and Impact

**Planning** and **scheduling** are both forms of **decision-making**.

Decision-making in manufacturing and service industries can be applied to:
* procurement and production
* transportation and distribution
* information processing and communication

and many other areas.

In general, we expect planning and scheduling algorithms to be able to:

> allocate limited **resources** to the **activities** to be done so that the company can optimize its **objectives** and achieve its goals.

Here, **resources** usually refer to things such as:
* machines in a workshop
* runways at an airport
* crews at a construction site
* processing units in a computing environment

**Activities** typically include:
* machine operations in a workshop
* take-offs and landings at an airport
* stages of a construction project
* execution of computer programs

**Objectives** can take many forms, for example:
* minimizing the total completion time of all jobs
* minimizing the number of tardy jobs

The textbook then provides ten examples to illustrate these concepts (omitted here).

---

## 1.2 Planning and Scheduling Functions in an Enterprise

### Planning and Scheduling in Manufacturing

In manufacturing, each **order** is decomposed into **jobs**, each of which has a **due date**.

These jobs must be processed by machines in a specific order.

However, processing may be delayed if certain machines are heavily loaded. Therefore, to handle **high-priority jobs**, we may need to use **preemptive scheduling**.

In addition, unexpected events must often be taken into account, such as:
* machine breakdowns
* processing times exceeding expectations

Under such conditions, the overall objective is to achieve **efficient and controllable production**.

Next, the author emphasizes again the distinction between *scheduling* and *planning*:

> The shopfloor is not the only part of the organization that impacts the **scheduling** process. The scheduling process also interacts with the production **planning** process, which handles medium- to long-term planning for the entire organization.

In short, scheduling is influenced not only by the current state of the shop floor, but also by planning decisions—that is, the planning algorithms being used.

In this context, planning decisions are usually based on considerations such as:
* inventory levels
* demand forecasts
* resource requirements

After understanding these concepts, we can examine a more detailed information flow diagram of a production system:

![alt text](/img/pns/fig1_1.png)

This diagram introduces many new concepts, which can be examined one by one. First of all, internal systems must interact and align with external systems:

> In manufacturing, planning and scheduling has to interact with other decision-making functions in the plant.

One commonly used system is the **Material Requirements Planning (MRP)** system. Since each schedule requires sufficient **resources** and **raw materials** to be available at the scheduled time, the MRP system determines *when* raw materials should be purchased based on current inventory levels. Today, mature commercial MRP software solutions are widely available.

Another common system is the **Enterprise Resource Planning (ERP)** system. ERP systems are used to integrate data across individual computers and data centers, and may also be connected with suppliers and customers. Decision-support systems are typically integrated with ERP platforms.

### Planning and Scheduling in Services

(Omitted)

---

## 1.3 Outline of the Book

![alt text](/img/pns/fig1_3.png)