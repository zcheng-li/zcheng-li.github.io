---
title: "L5.集成导航"
weight: 50
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Integrated Navigation

## Principles of Sensor Fusion

引入了一个小的知识点:
> to fuse the information from two pdfs we can multiply them together

## The Kalman Filter

**卡尔曼滤波** (Kalman Filter) 的工作步骤:

* A Prediction Step and
* A Measurement Update Step

### Filter Equations

![alt text](/img/navi/L5_p14.png)

![alt text](/img/navi/L5_p15.png)

> We use ’−’ to differentiate between ´a priori and ´a posteriori estimates, i.e. estimates from the prediction step (´a priori) and those modified by the measurement step (´a posteriori)

* a priori estimate（先验估计 / 预测估计）
* a posteriori estimate（后验估计 / 更新后的估计）

![alt text](/img/navi/L5_p16.png)

![alt text](/img/navi/L5_p22.png)

这张图展示的是 标准离散卡尔曼滤波器的预测（Time Update）和更新（Measurement Update）方程。

⸻

🟥 Time Update（预测 / 先验）

1.  $$\hat{x}_k^{-}$$

先验状态估计（prediction）
即：根据系统模型预测得到的状态，但还没有用测量修正。

\[\hat{x}_k^{-} = \Phi_{k-1} \hat{x}_{k-1}\]

⸻

2.  $$\Phi_{k-1}$$

状态转移矩阵（State Transition Matrix）

它描述系统状态在离散时间步之间是如何演化的。

例如位置速度系统：

\[x_k = x_{k-1} + v_{k-1} dt\]

—

3.  $$P_k^{-}$$

先验协方差矩阵（prediction covariance）
代表状态预测的不确定性。

\[P_k^{-} = \Phi_{k-1} P_{k-1} \Phi_{k-1}^T + Q_k\]

⸻

4.  $$Q_k$$

过程噪声协方差（Process noise covariance）

代表系统模型的不确定性，例如：
* 加速度模型不准
* 外界干扰
* IMU 噪声

$$Q_k$$会 使预测的不确定性变大。

⸻

🟩 Measurement Update（修正 / 后验）

5.  $$K_k$$

卡尔曼增益（Kalman Gain）

\[K_k = P_k^{-} H_k^T (H_k P_k^{-} H_k^T + R_k)^{-1}\]

直观意义：

决定信任测量多少 vs 信任预测多少

* 若 $$R_k$$（测量噪声）大 → 信预测
* 若 $$P_k⁻$$（模型不确定）大 → 信测量

⸻

6.  $$z_k$$

测量向量（Measurement）

来自传感器的实际测量值
如 GPS 位置、速度计读数、IMU 测量等。

⸻

7.  $$H_k$$

测量矩阵（Measurement Matrix）

描述状态如何映射到测量：

\[z_k = H_k x_k + v_k\]

例子：
若测量只观测到位置：

\[H_k = [1 \; 0]\]

⸻

8.  $$\hat{x}_k$$

后验状态估计（修正后的状态）

\[\hat{x}_k = \hat{x}_k^{-} + K_k(z_k - H_k\hat{x}_k^{-})\]

括号里的项叫 innovation（创新 / 残差）

\[z_k - H_k \hat{x}_k^{-}\]

表示测量与预测之间的差值。

⸻

9.  $$P_k$$

后验协方差（修正后的不确定性）

\[P_k = (I - K_k H_k) P_k^{-}\]
* 后验不确定性会比先验小
* 表示状态估计变得更精确

⸻

10.  $$R_k$$

测量噪声协方差（Measurement noise covariance）

代表传感器噪声的大小。

例如：
* GPS 噪声可能是 3 m，$$R_k$$ 较大
* 激光雷达噪声可能是 0.1 m，$$R_k$$ 较小

⸻


### Analysis of Filter

### 2D Navigation Example

## Integrated INS/GNSS Navigation

### Integration Architecture

### INS/GNSS Kalman Filter

### Integrated Navigation Example