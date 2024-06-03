---
tags:
  - SLAM
  - 算法架构
  - fastlio
---


## 1. FastLio1算法架构

![[Pasted image 20240425131050.png]]

## 2.FastLio2算法架构



![[Pasted image 20240425130559.png]]

## 3.FastLio1和2算法框架比较

**相同点**
1.  状态估计（state estimation）这部分基本上是相同的
2.  整个算法处理流程基本上也是一致的

**不同点**
1.  fastlio2取消了lidar点云的特征提取：fastlio1先针对点云进行了线特征和面特征的提取，之后滤波器的更新也针对这些特征进行，同时插入地图也是这些特征点；而fastlio2取消了特征提取这部分，主要原因是fastlio2认为将完整的scan插入map可以提高点云匹配的机率从而提供定位精度；另外，取消特征提取可以避免因为lidar扫描方式不同带来特征提取的复杂性（[[FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry.pdf#page=1&selection=27,38,31,32|FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry, page 1]]）

2.  fastlio2使用ikd-tree去管理地图而fastlio使用的是kd-tree，ikd-tree可以更加高效地查找临近点的同时，还能实现自动降采样和平衡地图拓展和运算效率。（[[FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry.pdf#page=1&selection=35,1,49,58|FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry, page 1]]）

## 4. FastLio2算法模块原理

#### 4.1 State Estimation
	该模块数据流行如下：
		状态前向递推 -> 状态后向递推正畸点云 -> 计算量测误差和H矩阵 -> 迭代EKF更新状态
	首先IMU和Lidar数据的融合需要在相同的时间点，所以前向递推的目的是将状态递推到lidar点云的end时刻，而状态后向递推则是为了得到每一个点云采集时刻对应的Pose信息，这样可以通过正畸算法将其转到lidar点云的end时刻位置，从而统一所有点云的时刻为lidar的end时刻，进而进行EKF滤波。可以参照fastlio的前后递推的说明图像：

		![[Pasted image 20240425154407.png]]

1. Forward Propagation（[[FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry.pdf#page=6&selection=122,3,122,14|FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry, page 6]]）
	这部分就是根据状态的运动学模型和IMU数据进行状态递推和协方差递推。


2. Backward Propagation
	后向递推是根据imu数据和lidar-end时刻的状态开始反向推算出从end到begin时刻的位姿，从而将此时的lidar点云正畸到lidar-end时刻。
		
3. Iterative EKF

#### 4.2 Ikd-Tree
