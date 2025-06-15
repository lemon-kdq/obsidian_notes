---
tags:
  - SLAM
  - 算法架构
  - fastlio
author: KDQ
email: kdq1290@gmail.com
---
## 0.前言

FastLio系列是LIO-SLAM中使用EKF算法架构达到较好表现的开源算法之一，其在鲁棒性和精度方面甚至要超过一般的基于图优化的系统（个人感觉）。初学者可以参照[[FastLio系列——FastLio的优势和不足]] 来了解FastLIO，


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

