---
title: "CH1.规划调度: 角色与影响"
weight: 1
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---


## 1.1 Planning and Scheduling: Role and Impact

**规划** 与 **调度** 是 **决策制定** (decision-making) 一种形式. 

决策制定 在制造服务产业中 可以用来
* 采购与生产 (procurement and production)
* 运输与分发 (transportation and distribution)
* 信息处理与通讯 (information processing and communication)

等等.

我们希望我们的规划与调度算法可以做到:
> allocate limited **resources** to the **activities** to be done so that the company can optimize its **objectives** and achieve its goals.

这里的 资源 (resources) 一般可以是:
* 车间里的机器 (machines in a workshop)
* 飞机场的跑道 (runways at an airport)
* 工地的员工 (crews at a construction site)
* 计算机的处理单元 (processing units in a computing environment)

行动 (activities) 一般可以是:
* 车间里机器的运行处理行动 (operations in a workshop)
* 机场里飞机的起飞降落行动 (take-oﬀs and landings at an airport)
* 工地项目的一个阶段? (stages in a construction project)
* 计算机程序的执行 (computer programs that have to be executed)

目标 (objectives) 有很多种形式, 一般可以是:
* 减少完成所有 任务 的总时间
* 减少超时完成的 任务 的数量

然后书上例举了10个例子来说明这些概念 (略).

## 1.2 Planning and Scheduling Functions in an Enterprise

### Planning and Scheduling in Manufacturing.

在制造业中, 每一份 **订单** (order) 都需要被拆解成 拥有 **截止日期** (due date) 的 **任务** (jobs).

这些 任务 需要 机器 按照一定的顺序来进行处理.

然而, 处理可能会被延迟, 如果遇到某台机器很忙的情况. 因此, 为了处理 **高优先级任务**, 我们需要用 **抢占式调度** (preemptions).

有时, 某些突发事件也需要被考虑进来, 比如:
* 机器坏掉
* 超出预期的处理时间

在这样的场景下, 我们的 目标 就是高效率和可控生产.

接下来, 作者再次强调了 scheduling 和 planning 的区别:

> The shopfloor is not the only part of the organization that impacts the **scheduling** process. The scheduling process also interacts with the production **planning** process, which handles medium- to long-term planning for the entire organization. 

简言之, scheduling 不仅会收到车间本身状况的影响, 同时也会收到 planning的影响, 也就是你采用的规划算法. 

在这个场景下, 规划算法 的制定一般会基于以下考量:
* 仓储情况 (inventory levels)
* 需求预测 (demand forecasts)
* 资源需求 (resource requirements)

在了解上面这些概念后, 我们可以开始看一个更详细的生产系统的信息流图:

![alt text](/img/pns/fig1_1.png)

里面有很多新的概念, 我们可以一个个看. 首先, 我们的内部系统肯定是要与外部系统进行沟通和(对齐)的:

> In manufacturing, planning and scheduling has to interact with other decision making functions in the plant.

其中一个比较常见的系统是 **Material Requirements Planning** (MRP) system. 由于每个 调度 都需要工厂能够在制定时间时拥有足够的 资源 (resources) 和 原材料 (raw materials), 所以 MRP 系统会根据当前的仓储量来**决定**买入原材料的时间. MRP 目前已经有成熟的商业软件可以使用.

另一个常见的系统是 **Enterprise Resource Planning** (ERP) systems. 就是用于把各个私人电脑和数据中心的数据拉齐. 有时也会与供应商/客户的数据相连. 决策辅助系统一般需要与 ERP 相连.

### Planning and Scheduling in Services.

略

## 1.3 Outline of the Book

![alt text](/img/pns/fig1_3.png)





