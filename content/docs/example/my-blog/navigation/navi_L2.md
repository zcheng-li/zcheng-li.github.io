---
title: "L2.导航数学基础"
weight: 20
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# The Mathematics of Navigation

## Coordinate Systems

为了实现 navigation position fix, 我们需要先确定一个**参考系** (frame-of-reference), \(\mathscr{F}\).

一个**直角坐标系** (Cartesian coordinate frame) 包含了三个相互正交 (mutually-orthogonal) 的基向量 \((x, y, z)\).

例如:

![alt text](/img/navi/L2_p3.png)

这个图中, \(p\) 在坐标系 \(\mathscr{F^{\beta}}\) 中, 因此我们可以把其记作 \(\underline{p^{\beta}}\). 

其中, 下划线表示其是一个矢量, 上标 \(\beta\) 表示其是在 \(\mathscr{F^{\beta}}\) 进行的测量.

## Navigation Frames

接下来, 我们需要定义几个常用的参考系用于导航.

### Earth Centred Inertial

**地心惯性坐标系** (Earth-Centered Inertial), 简称 ECI, 常用 \(i\)表示.

> An inertial coordinate frame is one that does not accelerate or rotate with respect to the rest of the Universe.

ECI 坐标系 \(\mathscr{F^i}\) 的原点处于地球球心 \(O_e\). 其 z 轴指向北极, x, y轴 处于赤道平面, 但是不和地球一起旋转. 其中, x 轴在**春分时**正好指向太阳, y 轴负责满足 RHS (right-handed set).

### Earth Centred - Earth Fixed

**地心固定坐标系** (Earth Centred, Earth Fixed), 简称 ECEF, 常用 \(e\) 表示.

ECEF 坐标系 (\(\mathscr{F^e}\)) 也是以地球球心为原点, 但是它的轴是固定在地球上的. 具体来说, z 轴指向北极, x 轴指向**格林威治子午线** (Greenwich merdian) 与赤道平面的交点, y 轴负责满足 RHS. 

ECEF 坐标系相对于 ECI 这个惯性系 以 \(\Omega_{ie}\) 绕着 \(Oz^i\) 轴旋转.

![alt text](/img/navi/L2_p9.png)

### Local Navigation Frame

**本地导航坐标系** (Local Navigation Frame), 简称 NED, 记作 \(n\) (native ??), i.e. \(\mathscr{F^n}\).

其原点固定在需要的地方, x 轴朝北 (N), y 轴朝东 (E), z 轴朝下 (Down). 所以叫它 NED.

![alt text](/img/navi/L2_p11.png)

### Vehicle (body) Frame

对象参考系, 和 NED 参考系很相似, 但是这个参考系的**姿态** (attitude) 会随着时间变换.

![alt text](/img/navi/L2_p12.png)

## Mathematical Notation
现在, 我们需要更详细的定义一下坐标系:

\[\underline{x}^{\gamma}_{\beta\alpha}\]

其中:
* \(\alpha\) 是 **对象系** (object frame)
* \(\beta\) 是 **参考系** (reference frame)
* \(\gamma\) 是 **解析系** (resolving frame)

例如:

![alt text](/img/navi/L2_p15.png)

> The red vector shows the position of \(\mathscr{F}^{\alpha}\) wrt \(\mathscr{F}^{\beta}\). The black dotted lines show the coordinates of \(\underline{r}^{\beta}_{\beta\alpha}\) and the blue dotted lines \(\underline{r}^{\alpha}_{\beta\alpha}\).

## Transforming Between Coordinate Frames

### Coordinate Transformation Matrix

![alt text](/img/navi/L2_p19.png)

这里的矩阵称之为 **方向余弦矩阵** (Direction Cosine), 其是围绕着 \(z^{\beta}\) 轴进行的旋转. 

这样的旋转矩阵是坐标变换矩阵的一种, 记作 \(C^{\gamma}_{\beta}\). 下标 \(\beta\) 表示 from, 上标 \(\alpha\) 表示 to. 这点与前面的标记不太一致.

我们可以这样使用这个矩阵:

\[\underline{x}^{\alpha}_{\delta\gamma} = C^{\alpha}_{\beta}\underline{x}^{\beta}_{\delta\gamma}\]

这个方向余弦矩阵有以下性质:

* 正交 (orthogonal)

\[C^{\alpha}_{\beta} = (C^{\beta}_{\alpha})^{-1} = (C^{\beta}_{\alpha})^T\]

* 可相乘

\[C^{\gamma}_{\beta} = C^{\gamma}_{\alpha}C^{\alpha}_{\beta}\]

### Euler Angles

为了定义**绝对**的姿态, 我们需要**欧拉角** (Euler angle).

然后, 整个完整的坐标系变换可以分成以下三个部分:

* YAW (\(\psi\)), 围绕初始的 z 轴旋转.
* PITCH (\(\theta\)), 围绕第一步旋转完成后的 y 轴旋转.
* ROLL (\(\phi\)), 围绕第二步旋转完成后的 x 轴旋转

以及每一步对应的变换矩阵:
![alt text](/img/navi/L2_p24.png)

最终的 Euler Direction Cosine matrix (\(C^{\alpha}_{\beta} = C^{\alpha}_{\prime\prime}C^{\prime\prime}_{\prime}C^{\prime}_{\beta}\)):

![alt text](/img/navi/L2_p24b.png)

## Cartesian Coordinates

### Velocity

参考系\(\mathscr{F}^{\alpha}\) 对于 参考系\(\mathscr{F}^{\beta}\) 的速度, 在 \(\mathscr{F}^{\gamma}\) 下的解析解是: 

\[\underline{v}^{\gamma}_{\beta\alpha} = C^{\gamma}_{\beta}\dot{\underline{r}}^{\beta}_{\beta\alpha}\]

注意: 这个和直接对 \(\underline{r}^{\gamma}_{\beta\alpha}\) 关于时间求导的结果**不一样**. 这是因为, 当存在坐标系之间的旋转时, 我们有:

\[\underline{\dot{r}}^{\gamma}_{\beta\alpha} 
= \dot{C}^{\gamma}_{\beta}\underline{r}^{\beta}_{\beta\alpha} + C^{\gamma}_{\beta}\underline{\dot{r}}^{\beta}_{\beta\alpha}
= \dot{C}^{\gamma}_{\beta}\underline{r}^{\beta}_{\beta\alpha} + \underline{v}^{\gamma}_{\beta\alpha}\]

一个重要结论: 如果 解析系 和 参考系 有旋转, 那么对 时间求导 不能直接等于速度/加速度.

### Angular Velocity

角速度 \(\underline{\omega}^{\gamma}_{\beta\alpha}\) 是 参考系 \(\mathscr{F}^{\alpha}\) 相对于坐标系 \(\mathscr{F}^{\beta}\) 的旋转速度, 在坐标系 \(\mathscr{F}^{\gamma}\) 下解析.

其可表示为:

\[\underline{\omega}^{\gamma}_{\beta\alpha} = (p^{\gamma}_{\beta\alpha}, q^{\gamma}_{\beta\alpha}, r^{\gamma}_{\beta\alpha})\]

另一个常见的表达形式是**反对称矩阵** (skew-symmetric matrix):

![alt text](/img/navi/L2_p30.png)

与这个矩阵相乘, 等效于与 \(\underline{\omega}^{\gamma}_{\beta\alpha}\) 做叉乘.

#### Transformation of Matrix

那么, 结下来我们可以来思考一下如何变换 变换矩阵 的解析系 (好像有点点绕...).

由于这里变换的是矩阵, 这点与前面的向量不太一样, 所以我们需要重新推导:

![alt text](/img/navi/L2_p31.png)

![alt text](/img/navi/L2_p32.png)

#### Strapdown Equation

在我们定义了角速度后, 现在我们可以开始定义 \(\dot{C}^{\gamma}_{\beta}\).

![alt text](/img/navi/L2_p33.png)

![alt text](/img/navi/L2_p34.png)

![alt text](/img/navi/L2_p35.png)

因此, 我们可以得到 **捷联方程** (Strapdown Equation):

fixed-to-rotating:
\[\dot{C}^{\gamma}_{\beta}(t) = - \Omega^{\gamma}_{\beta\gamma}C^{\gamma}_{\beta}\]

rotating-to-fixed:
\[\dot{C}^{\beta}_{\gamma}(t) = C^{\beta}_{\gamma}\Omega^{\gamma}_{\beta\gamma}\]

其他的几种形式:

![alt text](/img/navi/L2_p36.png)

### Acceleration

**加速度**的定义就是对于位置的二阶导.

当然, 在变换加速的解析系时, 我们也会遇到相同的难题:

![alt text](/img/navi/L2_p42.png)

其中,

* \[\ddot{C^{\gamma}_{\beta}}\]
* \[\dot{\underline{r}}^{\beta}_{\beta\alpha}\]
  
都是待续解决的问题.

对于第一个:

![alt text](/img/navi/L2_p43.png)

对于第二个:

![alt text](/img/navi/L2_p44.png)

代入后可以得到:

![alt text](/img/navi/L2_p45.png)

## Earth Surface and Gravity Models

### Earth Surface Model

![alt text](/img/navi/L2_p51.png)

![alt text](/img/navi/L2_p53.png)

![alt text](/img/navi/L2_p61.png)

### Gravity Models

**重力模型** (Gravity Models) 有两个关键的分量:
* 比力 (Specific Force) \(\underline{f}^{\gamma}_{ib}\): 单位质量受到的非万有引力的力.
* 万有引力 (Gravitation) \(\underline{\gamma}^{\gamma}_{ib}\): 对于单位质量的万有引力.

ps: Gravitation 指的是纯粹的万有引力，不包含由于地球自转产生的向心加速度。

因此我们有:

\[ \underline{f}^{\gamma}_{ib} = \underline{a}^{\gamma}_{ib} - \underline{\gamma}^{\gamma}_{ib}\]

在实际使用这个方程时, 我们会发现, 由于传感器是固定在 body 上的, 所以我们测量的比力是 \(\underline{f}^{b}_{ib}\). 

再有,

\[\underline{a}^{e}_{ib} = \underline{\Omega}^{e}_{ib}\underline{\Omega}^{e}_{ib}\underline{r}^{e}_{ib} + 2\underline{\Omega}^{e}_{ib}\underline{v}^{e}_{ib} + \underline{a}^{e}_{ib}\]

ps: 我们不考虑地球的自转加速度.

当物体在地球上静止不动时, 我们有:

\[\underline{a}^{e}_{ib} = \underline{\Omega}^{e}_{ib}\underline{\Omega}^{e}_{ib}\underline{r}^{e}_{ib}\]

代入可得:

\[ \underline{f}^{\gamma}_{ib} = \underline{\Omega}^{e}_{ib}\underline{\Omega}^{e}_{ib}\underline{r}^{e}_{ib} - \underline{\gamma}^{\gamma}_{ib}\]

由于我们认为在 ECEF 系中, 该物体只受到 重力 (Gravity) 和 非重力外力, 我们有:

\[ \underline{g}^{e}_{b} = -\underline{f}^{e}_{ib} = \underline{\gamma}^{\gamma}_{ib} - \underline{\Omega}^{e}_{ib}\underline{\Omega}^{e}_{ib}\underline{r}^{e}_{ib}\]

我们因此得到了重力.

## Navigation Frames and Transposition











