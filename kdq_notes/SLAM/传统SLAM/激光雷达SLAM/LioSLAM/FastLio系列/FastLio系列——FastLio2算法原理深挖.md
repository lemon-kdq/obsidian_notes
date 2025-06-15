---
tags:
  - SLAM
  - fastliosam
  - 量测更新
  - fastlio2
  - 代码深挖
aliases:
---

# 0.前言

因为FastLio2继承了FastLio1的有点并在此基础上进行了改进整体算法框架变化不大可参考[[FastLio系列——FastLio1和2算法架构比较]]，所以这里只介绍FastLio2的算法原理。本文会结合代码和文章深度讲解FastLio2的算法原理。
# 1. FastLIO2算法架构

![[Pasted image 20240425130559.png]]


# 2. FastLio2模块详解

## 2.0 状态变量定义

**FastLio定义第一帧的IMU坐标系为Odometry的global坐标系，之后的所有Pose是相对于第一帧IMU坐标系而言的。**


## 2.1 数据预处理

## 2.2 状态预测

## 2.3 点云正畸

## 2.4 量测更新

## 2.5 地图更新



#### 4.1 State Estimation
	该模块数据流行如下：
		状态前向递推 -> 状态后向递推正畸点云 -> 计算量测误差和H矩阵 -> 迭代EKF更新状态
	首先IMU和Lidar数据的融合需要在相同的时间点，所以前向递推的目的是将状态递推到lidar点云的end时刻，而状态后向递推则是为了得到每一个点云采集时刻对应的Pose信息，这样可以通过正畸算法将其转到lidar点云的end时刻位置，从而统一所有点云的时刻为lidar的end时刻，进而进行EKF滤波。可以参照fastlio的前后递推的说明图像：

		![[Pasted image 20240425154407.png]]

1. Forward Propagation（[[FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry.pdf#page=6&selection=122,3,122,14|FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry, page 6]]）
	这部分就是根据状态的运动学模型和IMU数据进行状态递推和协方差递推。


2. Backward Propagation
	后向递推是根据imu数据和lidar-end时刻的状态开始反向推算出从end到begin时刻的位姿， -end时刻。
		
3. Iterative EKF

#### 4.2 Ikd-Tree



#### 1.量测模型

    参照fast-lio2文章中的Measurement Model:

[[FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry.pdf#page=5&selection=846,0,847,1|FAST-LIO2_Fast_Direct_LiDAR-Inertial_Odometry, page 5]]:


# 2.代码
## 2.1 伪代码

	
```
void h_share_model(state_ikfom &s,
                   esekfom::dyn_share_datastruct<double> &ekfom_data)
	for p in feats points
		将lidar坐标系下的点pL转到世界坐标系下得到pW；
		查询在ikdtree map中搜索距离该世界坐标系点最近的五个点；
		if 计算该平面的法向量n，五点到法向量的距离和 < 0.1：
			计算该平面到该点的距离d；
			s = 1 - 0.9 * d / p_l.norm() #因为状态误差可能导致越远的点，距离d越大
			if s > 0.9:
				插入残差d和法向量n
	
	for p in effect_feat_num
		计算h矩阵
	
```

