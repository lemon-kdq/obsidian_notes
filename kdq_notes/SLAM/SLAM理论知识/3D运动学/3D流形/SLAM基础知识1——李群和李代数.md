---
author: KDQ
tags:
  - SLAM
  - 李群
  - 李代数
  - 流形
email: kdq1290@gmail.com
date: 2025/06/13
---
## 一. 背景
### 1. SLAM中为什么要引入李群和李代数的概念
#### 1.1 线性空间和流形空间的特性
为什么在SLAM中定义状态时候要引入李群和李代数的作为姿态变量的参数化方法，主要原因就是姿态变量不在线性空间，而是在流形空间。
对于线性空间$\boldsymbol{R}^n$满足：
	1）加法封闭性：$a+ b \in\boldsymbol{R}^n$
	2)	 交换律：$a+b = b+a$
但是对于流形空间（局部线性，全局非线性的空间），完全不存在线性空间的上述两条性质。比如我们用旋转矩阵R来表示姿态SO(3)(特殊正交群)，则有: 
	$SO(3) = \{{R\in \boldsymbol{R}^{3\times3}|R^T R=\boldsymbol{I},det(R) = 1}\}$
很明显： $R_1 + R_2 \notin SO(3)$
在比如用四元数来表示旋转，则有： 
	$\boldsymbol{S^3} = \{{ q\in \boldsymbol{R}^4 | \left \|q\right\|=1} \}$
很明显：$q_1 + q_2 \notin \boldsymbol{S^3}$
这就带来一个问题，我们在进行状态更新$x_{k} = x_{k-1} + \delta x$ ，我们无法保证姿态变量还能在流形空间（即表示正确的旋转姿态），也无法像线性空间这样$F=\frac{\partial f(x)}{\partial x}=\frac{f(x+\delta x)-f(x)}{\delta x}$ 去定义Jacbian矩阵，都是因为流形空间不存在类似线性空间具备封闭性的加法。
#### 1.2 使用四元数、欧拉角参数化的弊端
##### 1.2.1四元数参数化
旋转姿态实际自由度仅为3，我们使用单位四元数虽然有四个数但是自由度也是3，每次更新的时候都需要归一化四元数:
$$
g(q) = \frac{q}{\|q\|} \tag{1}
$$
理论上我们应该对协方差矩阵进行更新，即：
$$
\overline P = J_{g}PJ_{g}^T \tag{2}
$$
如果不更新协方差矩阵，则该协方差并不能准确的描述姿态的不确定度。归一化操作是一个高度非线性的操作，本身描述四元数的协方差矩阵是4个维度的，归一化操作则强制将其压缩到3个维度上，这将导致Kalman Filter中的高斯误差在归一化造作后不再是高斯分布，也即是协方差矩阵将病态化，协方差矩阵有可能行列式接近为0（降维）数值求解将很不稳定。
##### 1.2.2 欧拉角参数化
姿态用欧拉角做参数化则更为不合适，首先我们回顾一下欧拉角的定义，现在按照x-y-z顺序来定义欧拉角$\phi,\theta,\psi$ ，其欧拉角的微分方程为：
$$
\begin{bmatrix}
\dot{\phi} \\
\dot{\theta} \\
\dot{\psi}
\end{bmatrix}
=
\begin{bmatrix}
1 & \sin\phi \tan\theta & \cos\phi \tan\theta \\
0 & \cos\phi             & -\sin\phi \\
0 & \sin\phi / \cos\theta & \cos\phi / \cos\theta
\end{bmatrix}
\begin{bmatrix}
\omega_x \\
\omega_y \\
\omega_z
\end{bmatrix} \tag{3}
$$
从这个微分方程就能发现欧拉角表示姿态很大问题：
**奇异性：** 微分方程中当$\theta$为90°时，将会导致微分矩阵异常，导致丧失一个自由度，欧拉角不具有全局一致的自由度，在Jacbian在90°附近很不稳定，导致滤波器崩溃
**耦合性：** 每一个角度的导数都不是独立的，都会受到其他角度的影响，即便是你按照一个方向旋转也会因为耦合导致其他角度发生很大的变化，所以即便是很小的误差扰动，积分误差都会很大，说到底欧拉角也不处于线性空间，不能随意的加减
**高度非线性：** 同样只要是$\theta$接近90°，可以发现$tan\theta,sec\theta$都非常的大，造成角速度映射的严重变形，jacbian矩阵将变得机器不稳定，难以求导
下面给一个简单的欧拉角扰动的例子：
	import numpy as np
	from scipy.spatial.transform import Rotation as R
	from numpy.linalg import svd, cond
	def compute_jacobian(euler):
	    # Small angle perturbation to compute finite differences
	    delta = 1e-6
	    J = np.zeros((9, 3))  # 3x3 rotation matrix flattened → shape (9,)
	    base_R = R.from_euler('zyx', euler).as_matrix().flatten()
	    for i in range(3):
	        euler_perturbed = np.array(euler,dtype=float)
	        euler_perturbed[i] += delta
	        perturbed_R = R.from_euler('zyx', euler_perturbed).as_matrix().flatten()
	        J[:, i] = (perturbed_R - base_R) / delta
	    return J
	J1 = compute_jacobian([0, 0, 0])
	print("Condition number (pitch = 0):", cond(J1))
	J2 = compute_jacobian([0, np.deg2rad(89.9), 0])
	print("Condition number (pitch ≈ 90°):", cond(J2))
	U, S, V = svd(J2)
	print("Min singular value near singularity:", np.min(S))	
所以，我们需要找到更好的姿态参数化方法，要求其满足欧式空间的可加性，自由度为3，在姿态所有空间都保持这些条件得以满足。而李代数就可以满足上述的要求，我们通过指数运算将李代数和李群映射起来，通过李代数的加减法来定义状态量相对于姿态的Jacbian，姿态的误差直接通过李代数来传递和修正。

## 二. 李群和李代数的定义

关于李群的定义在高翔的《视觉SLAM十四讲》中已经有了介绍，这里只给简单的定义。首先群的定义是**一种集合+一种运算的数据结构** ，此运算放在集合中输出的结果依然在该集合里，具有封闭性。比如整数和加法就构成一个集合，旋转矩阵和乘法也构成一个集合，但是旋转矩阵的加法就不具备封闭性，就不是群。**李群**在群的基础上还拥有流形的特征，即使在连续的和光滑的空间，比如用于描述刚体在3D空间的旋转和位移就是连续的变化，所以旋转和位移和它们乘法运算构成了李群：
$$
R_1 R_2 \in SO(3), T_1 T_2 \in SE(3)
$$
### 1. SO3和so3
#### 1.1 旋转矩阵微分方程
关于旋转矩阵，我们知道其遵循： 
$$
RR^T = I \tag {4}
$$
因为在3D空间旋转的时候时关于时间的函数用$R(t)$来表示不同时刻物体的姿态，然后求关于时间的导数：
$$
\begin{align}
\dot {R(t)R(t)^T} &= \dot{R(t)} R(t)^T + R(t)\dot{R(t)}^T = 0 \\
&=\dot{R(t)} R(t)^T + (\dot{R(t)} R(t)^T)^T = 0 \\
&=> \dot{R(t)} R(t)^T = -(\dot{R(t)} R(t)^T)^T
\end{align} \tag{5}
$$
说明$\dot{R(t)} R(t)^T$ 是个反对称矩阵，我们用$[\phi]_\times$ ($\phi \in R^3$)来表示，则有：
$$
\dot R(t) = [\phi]_\times R(t)
$$
如果调换顺序，还可以得到： 
$$
\dot R(t) = R(t)[\phi]_\times \tag{6}
$$
只不过这里面对应的$\phi$含义是不一样的，一个左乘的反对称矩阵定义在SO3原点处的切空间上，而右乘的反对称矩阵则定义在R(t)所在流形位置的切空间。
求解上式微分方程：
$$
\begin{align}
\dot{R_{t}} &= \frac{dR}{dt}=[\phi]_\times R \\
&=> \frac{dR}{R}=[\phi]_\times dt \\
&=> \int \frac{1}{R}dR = \int [\phi]_\times dt \\
&=> ln(R) = [\phi]_\times t + C\\
&=> R(t) = e^C exp([\phi]_\times t) \\
&=> R(t) = C_1 exp([\phi]_\times t)

\end{align} \tag{7}
$$
定义在$t=0,有R(0)=I$, 则有：
$$
R(t) = exp([\phi]_\times t) \tag {8}
$$
公式(8)展示了一个在线性空间$R^3$上的向量可以通过指数运算映射到SO3上。但是其中的指数是$R^{3*3}$的矩阵，如何计算这样的指数运算呢？
#### 1.2 罗德里格斯公式的证明
下面对公式(8)求解，这里直接用$\boldsymbol{\phi} = \phi t$ 对公式进行泰勒展开： 
$$
exp([\boldsymbol\phi]_\times) = \underset{n=0}{\overset{\infty}{\sum}} \frac{[\boldsymbol{\phi}]_\times^n}{n!} \tag{9}
$$
同时令$\boldsymbol \phi = \theta \boldsymbol u,其中\theta = \|\boldsymbol \phi\|,\boldsymbol u = \frac{\boldsymbol \phi}{\theta} = [x,y,z]^T$ ，则有： 
$$
[\boldsymbol u]_\times ^ 2 = \begin{bmatrix} 0 & -z & y \\ z & 0&-x\\-y & x&0  \end{bmatrix}\begin{bmatrix} 0 & -z & y \\ z & 0&-x\\-y & x&0  \end{bmatrix}=\begin{bmatrix} x^2-1 & xy & xz \\ yx & y^2-1&zx\\zx & zy&z^2-1  \end{bmatrix}=\boldsymbol{u}\boldsymbol{u^T} - \boldsymbol{I} \\ \tag{10}
$$
$$
[\boldsymbol u]_\times ^ 3 = \begin{bmatrix} x^2-1 & xy & xz \\ yx & y^2-1&zx\\zx & zy&z^2-1  \end{bmatrix}\begin{bmatrix} 0 & -z & y \\ z & 0&-x\\-y & x&0  \end{bmatrix}=\begin{bmatrix} 0 & z & -y \\ -z & 0&x\\y & -x&0  \end{bmatrix}=-[\boldsymbol{u}]_\times\tag{11} 
$$
讲公式(10)和(11)带入公式(9)得： 
$$
exp([\boldsymbol{\phi}]_\times)=\boldsymbol I + \theta [\boldsymbol{u}]_\times + \frac{\theta ^ 2}{2!}[\boldsymbol u]_\times ^ 2+  \frac{\theta ^ 3}{3!}[\boldsymbol u]_\times ^ 3+ \frac{\theta ^ 4}{4!}[\boldsymbol u]_\times ^ 4+ ... 
$$



