

## 1.相关材料
- **代码：** [Lidar_IMU_Init](https://github.com/hku-mars/LiDAR_IMU_Init?tab=readme-ov-file)
- **文章：** [[Robust Real-time LiDAR-inertial Initialization.pdf]]
- **功能：** 标定出Lidar和IMU的6DOF外参和时间常值offset，同时估计出系统的重力向量，IMU Bias等信息，可以直接用于Fast-LIO2的系统初始化，且该代码无缝嵌入了Fast-LIO2
- **方法：** 通过假定的恒速，恒角速度的模型实现纯Lidar Odometry，然后先通过构造如下约束一步步求解待标定的变量：
	1. 根据角速度模值变换粗对齐IMU和Lidar的时间offset（index级别）；
	2. 根据IMU和Lidar三轴角速度和它们外参、IMU陀螺仪bias以及dt的关系重新构造约束方程，优化出这些变量；
	3. 最后根据IMU和Lidar Odom的加速度关系构造出关于二者位置外参、加速度bias和重力向量的约束，优化求解出它们；
- **场景：** 标定时候需要先静止一下设备，让设备有一个准确的静态Map，然后再重分激烈三轴IMU，这样才能准确的估计出系统待标定的参数。
## 2. 算法结构

![[Pasted image 20240918155430.png]]
算法大体分两个模块：**Lidar Odometry** 和 **Lidar-IMU 初始化** ，前者用恒速和恒角速度模型的IEKF算法估计出Lidar的里程计信息，并根据微分出线和角的加速度信息。后者则是利用前者的输出信息将Lidar Odometry和IMU的加速度和角速度进行对齐，采用信号无延迟滤波降噪和ceres优化的方法将待优化的变量逐个的标定出来。
## 3. 重点分析
### 3.1 Lidar Odometry
#### 3.1.1 LiDAR Odometry
参考：**LiDAR Odometry**
[[Robust Real-time LiDAR-inertial Initialization.pdf#page=3&selection=257,3,257,17|Robust Real-time LiDAR-inertial Initialization, 页面 3]]
这部分使用的是迭代误差EKF，应该就是仿照Fast-LIO写的，这里为了更贴合恒速/恒角速度模型，对单帧进行拆分为子帧的方法**Frame Splitting**，让更新速率更加高。
#### 3.1.2 运动补充
参考： **Motion Compensation**
[[Robust Real-time LiDAR-inertial Initialization.pdf#page=3&selection=967,3,968,1|Robust Real-time LiDAR-inertial Initialization, 页面 3]]
同样为了补偿运动导致的点云畸变，进行了运动补偿。采用的方法还是和FAST-LIO一致，只是用恒速/恒角速度替换IMU前向和后向递推获取每一个点云相对于最后一个点的Pose。
### 3.2 Lidar-IMU Initialization
#### 3.2.1 数据预处理
参考：Data Process
[[Robust Real-time LiDAR-inertial Initialization.pdf#page=4&selection=116,3,116,18|Robust Real-time LiDAR-inertial Initialization, 页面 4]]
- 对应上图中的**Measurement Noise Attenuation** 。这部分主要就是采用无滞后低通滤波对IMU和Lidar Odometry输出的角速度和线速度进行滤波，这样后续在计算角加速度和线加速度时候不会出现因为噪声而导致的异常值.
- 对应图上的**Central Difference Calculation** 。这部分主要是通过Lidar Odometry的加速度和线速度数据中计算出角加速度和线加速度。这里作者采用的中心差分计算微分也借鉴了前人的成果，猜测通过此方法可以获得精度高且数值较稳定的微分信号吧。
#### 3.2.2 基于交叉相关的时间粗对齐
参考：temporal Initialization by Cross-Correlation
[[Robust Real-time LiDAR-inertial Initialization.pdf#page=4&selection=473,4,473,47|Robust Real-time LiDAR-inertial Initialization, 页面 4]]
对应图上**Cross-Correlation based Temporal Initialization** 。具体作者是采用zero-centered cross-correlation方法来估计出Lidar Odometry角速度和IMU角速度幅值之间时间相关性，从而将Lidar-IMU的时间误差控制到Lidar帧间的时间差（这里Lidar已经采用了分帧的测量频率在50-100HZ）。
#### 3.2.3 联合Lidar-IMU旋转外参和时间常值offset精准标定
参考：Unified Extrinsic Rotation and Temporal Calibration
[[Robust Real-time LiDAR-inertial Initialization.pdf#page=4&selection=630,3,630,54|Robust Real-time LiDAR-inertial Initialization, 页面 4]]
对应图上**Temporal Offset Refinement & Extrinsic Rotation Calibration**。不同于上一步用的是Lidar Odometry的角速度和IMU角速度的幅值，这里则是建立二者时间关于旋转外参的等式关系，同时为了解决上面标定的时间offset精度不够的问题，加入了IMU角加速度计量，将估计出更加精准的时间offset。
#### 3.2.4 位置外参和重力向量初始化
参考：Extrinsic Translation and Gravity Initialization
[[Robust Real-time LiDAR-inertial Initialization.pdf#page=5&selection=211,2,211,51|Robust Real-time LiDAR-inertial Initialization, 页面 5]]
对应图中的**Extrinsic Translation & Gravity Vector Calibration**，这一步则是利用之前求得的稳定的加速度数据建立和位置外参的约束方程，进行优化。
## 4.个人总计
大体算法如上述介绍，我提出两点问题：
1. 上述标定过程前提条件是先通过LO系统获得Lidar的Odometry信息，为了让Odometry的数值够准，作者建议需要静止系统5s时间，这样可以获得一个比较稳定的Global Map。对于在运动平台上就需要执行初始化步骤来说基本上不可能实现稳定的LO系统。
2. 作者为了实现比较准确的外参估计，需要IMU获得足够的激励，为了检测收集的数据是否满足要求还特意检查了信息矩阵是否满秩，这在车辆的环境下基本上不可能实现。
因此，本文的贡献还是局限在小型LIO系统且离线标定环境下进行。对于高速运动的车辆环境该算法应该是不能很好的初始化系统的。


