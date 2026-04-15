---
title: "L3.惯性导航"
weight: 30
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Inertial Navigation

## Mechanisation of Inertial Navigation Systems

惯性导航是一种**航位推测法** (dead reckoning). 如果我们知道一个物体的加速度, 那么我们就可以通过一次积分获得其速度修正, 两次积分以获得其位置.

不过, 正如上一节所看到的, 如果参考系和解析系不是同一个系, 那么就要耗费大量的运算资源去求解.

在过去, 由于算力有限, 人们需要使用 **Stabilised Inertial Platform**.

而现在, 由于算力不是大问题了, 人们直接讲传感器固定在对象上, 也就是使用 **Strapdown INS configuration**.

### Stabilised Inertial Platform

**稳定惯性平台** (Stabilised Inertial Platform) 大致如下:

![alt text](/img/navi/L3_p3.png)

里面的陀螺仪 (gyro) 会测量平台转到的角度. 而这些测量的角度又会被当成反馈信号传递给轴上的马达, 来调整姿态.

然而, 它有如下缺点:
* mechanically complex
* costly to manufacture/maintain
* larger volume/weight
* cannot be used directly for airframe stabilisation

### Strapdown System

因此, 对于新的 INS 系统, 人们通常喜欢使用 **Strapdown Configuration**. 

与 Stablised Inertial Platform 不一样, 它的 gyro 是直接固定在测量对象上的. 也就是说, 我们需要花费一定的算力, 去将 gyro 的测量信号从 body 系转换到所需的系中.

## The Navigation Equations

> The Navigation Equations form the core of all strapdown inertial navigation systems. 

对于**导航方程**我们有如下更 general 的表达形式:

\[\underline{a}^{\gamma}_{ib} = \underline{f}^{\gamma}_{ib} + \underline{\gamma}^{\gamma}_{ib}\]

接下来, 我们要求解两个不同的导航方程.

### Earth-Frame Navigation Equations

ECEF 系通常被卫星系统 (i.e. GNSS) 用于参考系和解析系. 因此, 我们也希望我们车子的导航可以在相同的系下进行解析.

我们可以重写 Navigation Equation 在 \(\mathscr{F}^e\):

\[\underline{a}^{e}_{ib} = \underline{f}^{e}_{ib} + \underline{e}^{\gamma}_{ib}\]

在实际使用这个方程时, 我们会发现, 由于传感器是固定在 body 上的, 所以我们测量的比力是 \(\underline{f}^{b}_{ib}\). 

因此:

\[\underline{a}^{e}_{ib} = C^{e}_{b}\underline{f}^{b}_{ib} + \underline{e}^{\gamma}_{ib}\]

那么, 我们要如何从这个式子里求解出我们感兴趣的 \(\underline{v}^{e}_{eb}\)? (其参考系是 ECEF)

![alt text](/img/navi/L3_p8.png)

![alt text](/img/navi/L3_p9.png)

![alt text](/img/navi/L3_p10.png)

![alt text](/img/navi/L3_p11.png)

然而, 这还没有结束. 在实际使用过程中, 我们需要实时更新 \(C^e_b\), 通过下面公式:

\[\dot{C^e_b} = C^e_b\Omega^b_{eb}\]

然而, 我们无法得到 \(\Omega^b_{eb}\), 因为我们的 gyro 只能测量惯性系的量, 也就是 \(\underline{\omega}^b_{ib}\).

因此, 我们需要通过一下方法得到:

![alt text](/img/navi/L3_p12.png)

然而, 问题依旧没有结束!!!! 

因为它还是用到了在 body 系解析的地球的转动速度 \(\Omega^b_ie\), 然而我们实际知道的是 ECEF 系解析的转动速度 \(\Omega^e_ie\).所以, 我们还得这样:

![alt text](/img/navi/L3_p13.png)

终于, 我们可以开心地更新这个坐标变换矩阵了!!!!

其总的流程如下图所示:

![alt text](/img/navi/L3_p14.png)

### Local Navigation Frame Equations

在这个解析系下, 求解\(\underline{v}^{n}_{eb}\)过程如下:

![alt text](/img/navi/L3_p19.png)

![alt text](/img/navi/L3_p20.png)

在更新姿态时, 也有一些不同:

![alt text](/img/navi/L3_p16.png)

可以发现, 上述式子中有三个\(\Omega\), 其中, 第一个是 gyro 测量的.

第二个与地球的自转有关:

![alt text](/img/navi/L3_p17.png)

第三个被称作 Transport Rate, 也就是和 NED 系相对于 ECEF 系的旋转有关.

![alt text](/img/navi/L3_p18.png)

总的流程如下:

![alt text](/img/navi/L3_p21.png)

### INS Update Steps

* Update attitude
* Transform specific force frame
* Update velocity
* Update position

## Inertial Sensors

### Accelerometers

**加速度仪**主要有以下两种类别:

* conventional mechanical
* MEMS (Micro-Electro Mechanical Machines).
  * piezoelectric devices (更精确, 但也带来了更多的 size/weight/volume/unit cost)
  * capacitive devices (most common)

#### conventional mechanical

![alt text](/img/navi/L3_p28.png)

#### MEMS Capacitive Accelerometer

### Gyroscopes

gyro 可以分成如下几类:

* Mechanical rate gyroscopes: use the **conservation of angular momentum** to measure angular rate
* MEMS gyroscopes: use the **Coriolis effect** to measure angular rate
* Optical gyroscopes: use the **Sagnac effect** to achieve the same measurement.

#### Mechanical Gyroscopes

![alt text](/img/navi/L3_p40.png)

#### Vibrating Structure Gyroscopes (VSG)

#### Laser Gyroscopes


## Sensor Error Characteristics

### Bias Error

acc 和 gyro 都有.

Bias 会变, 这一性质被称为 bias instability. 一般 MEMS 的 BS 要比机械的高.

### Scale Factor Errors

一般传感器的输入输出会有一个比值, 我们称之为 scale factor (SF).

![alt text](/img/navi/L3_p65.png)

### Cross-Coupling Errors

其经常与 SF error 一起用如下矩阵描述:

![alt text](/img/navi/L3_p66.png)

### Random Noise

Thermo-Mechanical White Noise 

### Error Models

最后, 几个误差全部包含进来后, 总的输出是:

![alt text](/img/navi/L3_p70.png)

对应的误差也可以得到:

![alt text](/img/navi/L3_p70b.png)

这些误差也可以被修正:

![alt text](/img/navi/L3_p71.png)
