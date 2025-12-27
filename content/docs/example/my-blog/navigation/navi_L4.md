---
title: "L4.卫星导航"
weight: 40
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Satellite Navigation

## Radio Navigation and Trilateration

雷达导航的基础是电磁波在真空中以光速传播. 所以只需要知道光传播的时间, 就可以测算出对应的距离.

三个雷达可以确定一个具体的位置, 这就叫三点定位法 (Trilateration)

它的计算步骤十分繁琐! 但是适合电脑. 流程如下:

![alt text](/img/navi/L4_p12.png)

其中, dr 是 true r - predicted r.

还有, 里面提到的 H, 是**雅各布矩阵** (Jacobian Matrix):

![alt text](/img/navi/L4_p11.png)

里面的 df / dx 可以用如下方式计算:

![alt text](/img/navi/L4_p10.png)

如果是 3 个 beacon 的话, 那么 \(H^{-1}\) 需要通过**Moore-Penrose pseudo inverse** 的方法来实现:

![alt text](/img/navi/L4_p21.png)

## GNSS Architecture

**全球卫星导航系统** (Global Navigation Satellite System, GNSS), 一般包含三个部分:

* Space segment
* Ground control segment
* User segment
  
### Space segment

其中, 我们感兴趣的部分有:

* Directed antenna array pointed at the earth 
* Atomic clock. 

> The space segment consists of a constellation of **24** satellites. 
> Each satellite broadcasts its **position** and a very accurate measurement of the **broadcast time** provided by an on-board caesium or rubidium **atomic clock**. 
> There are two broadcast signals, each modulated by a binary phase shift keying digital process encoding a **pseudo-random signal and a data signal**.

### Ground control segment

> The control segment monitors the **position and status** of each satellite and issues **course correction** commands, **clock adjustment** commands, satellite almanac information and ionospheric correction factors.

### User segment

> The user segment consists of an **antenna** to receive the broadcast signal from each satellite in view, a **receiver** segment to **demodulate** each satellite signal and **recover** both the pseudorange measurement and data message and a **processing** segment to **correct** the pseudorange measurements and compute a position, velocity, time navigation solution using **trilateration**.

## Sources of Error

> 8 principle sources of error in a GNSS system

* Satellite clock errors:
* Receiver clock error: 
* Ionospheric refraction: 离子层折射
* Tropospheric refraction: 对流层折射
* Satellite geometry: Dilution of precision.
* Relativity effects:Due to special and general relativity, the clocks on-board the satellites run faster than identical clocks on earth.
* Multipath errors: landscape surrounding the receiver.
* Receiver tracking errors: Thermal noise and interference eﬀects

### Geometric - Dilution of Precision

## Single-Point Navigation Solution

