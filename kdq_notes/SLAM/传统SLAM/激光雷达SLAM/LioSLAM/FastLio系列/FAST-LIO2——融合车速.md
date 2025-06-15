---
author: KDQ
tags:
  - SLAM
  - fastlio2
  - multi-sensor-fusion
  - 个人总结
date: 2025/06/05
email: kdq1290@gmail.com
---

---

## 1.背景

为了解决隧道等在前进方向上没有静态物体的场景导致fast-lio2退化问题，决定引入车速信号。因为fast-lio2之前使用的是IESKF（迭代误差Kalman Filter），其状态时误差状态，需要推导这部分的量测更新模型，同时需要考虑车速信号在哪一步进行更新。

## 2.相关定义

定义**名义状态**：
![[Pasted image 20250605095636.png]]
定义**误差状态**：
![[Pasted image 20250605100134.png]]
名义状态和误差状态维度不一致，误差状态是真实状态的自由度，用误差状态做滤波状态的好处有二：1）误差状态自由度是最小最完整的，不会引入额外的约束；2）误差状态在0附近所以线性化的时候可以直接省略二阶以上的，线性化程度高

## 3.误差状态的预测

误差状态的预测包括误差本身的预测和其协方差的预测，这部分的误差源主要来自于IMU数据，具体预测如下：

**误差状态的递推**：
![[Pasted image 20250605101322.png]]
**误差状态的协方差递推：**

![[Pasted image 20250605101557.png]]

$F_{\widetilde{x}}$ 和 $F_w$ 是相对于误差状态和噪声的Jacbians矩阵，具体推导可以参照：
[[FAST-LIO 论文解读.pdf#page=11&selection=111,0,122,9|FAST-LIO 论文解读, 页面 11]] 

## 4.车速量测更新

**量测模型：**
为了简化难度，认为没有必要使用迭代的方式去做状态更新，这里直接使用ESKF来做车速更新。因为我们从can拿到的车速是车辆纵向的轮速，具体它和fast-lio2中定义车速关系如下：

$$
^Cs = M \circ R_{CI}\circ R_{GI}^T \circ {{^G}V}+ n   (1)
$$
定义： 
 $^Cs$: 定义为采集到的车速(m/s)
  M: $\begin{matrix}[1 & 0 & 0]\end{matrix}$
  $R_{GI}= \overline{R}_{GI}\oplus\delta\theta$
  ${^G}V={^G}\overline{V}+\delta{V}$ 
  $R_{CI}$: imu到车身的旋转矩阵外参
  n: 是量测误差，符合高斯分布
  
**量测Jacbian推导：**
由公式(1)我们可以得： 

					$^Cs \cong M \circ R_{CI} \circ \overline{R}_{GI}^T \circ {{^G}\overline{V}}+ H_{\delta x} \delta{x}+ n$
这里需推导一下量测模型相对于误差状态得Jacbian H矩阵，H矩阵只和误差得速度和误差角度有关，具体如下： 

$$
\begin{align}
H_{\delta{\theta}} = \frac{\partial ^Cs}{\partial \delta{\theta}} 
&= \frac{\partial M \circ  R_{CI} \circ (\overline{R}_{GI}\oplus\delta\theta)^T \circ {^G}\overline{V} }{\partial \delta{\theta}} \\ 
&=\frac{\partial M\circ  R_{CI} \circ (I - [\delta\theta )]_{\times})\overline{R}^T_{GI}\circ  {^G}\overline{V}}{\partial \delta{\theta}}\\ 
&=-\frac{\partial M\circ  R_{CI} \circ [\delta\theta]_{\times}\overline{R}^T_{GI}\circ  {^G}\overline{V}}{\partial \delta{\theta}}\\
&=\frac{\partial M\circ  R_{CI} \circ [\overline{R}^T_{GI}\circ  {^G}\overline{V}]_\times \delta\theta}{\partial\delta\theta} \\
&=M \circ R_{CI} \circ [\overline{R}^T_{GI}\circ  {^G}\overline{V}]_\times 
\end{align}
$$
$$
\begin{align}
H_{\delta{V}} = \frac{\partial ^Cs}{\partial \delta{V}} 
&= \frac{\partial M \circ  R_{CI} \circ \overline{R}_{GI}^T \circ ({^G}\overline{V} + \delta V ) }{\partial \delta{V}} \\ 
&=M\circ  R_{CI} \circ \overline{R}_{GI}^T
\end{align}
$$
**状态更新**:

因为我们只使用ESKF，所以只要一次更新即可，具体查看如下部分：
[[Notes on Kalman Filter(KF, EKF, ESKF, IEKF, IESKF).pdf#page=9&selection=433,0,433,24|Notes on Kalman Filter(KF, EKF, ESKF, IEKF, IESKF), 页面 9]]
更新步骤为：
![[Pasted image 20250605133535.png]]
最后一步因为要重置误差状态所以对其协方差也有更新，但是因为这里面都是误差量这个G矩阵很小接近单位阵，所以这一步的P可以不操作，具体参照[[Quaternion kinematics for the error-state Kalman filter.pdf#page=65&selection=160,0,174,33|Quaternion kinematics for the error-state Kalman filter, 页面 65]]。

