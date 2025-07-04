---
tags:
  - SLAM
  - lidar
  - cornercase
  - 场景退化
author: KDQ
email: kdq1290@gmail.com
date: 2025/05/15
---
Lidar SLAM常见的Corner Case多数发生在场景退化和动态环境，总结如下：

1. 高架或者高速场景下动态目标物多：
	lidar Fov内存在大量的动态物体，举一个极端的例子，载具四周全是和自己同速运行的车辆，那么Lidar SLAM估计的速度状态可能是0，所以如何将动态物体直接从lidar点云中剔除很有必要。当前很少有开源的lidar slam算法做动态物体的剔除，即便能使用liper感知算法也很难通过单帧数据来判断是否是动态物体，所以为了降低难度，很多开源算法尝试把车辆的点云剔除。
	
	**LIO-Livox**：使用快速点云分割的方法将点云分为背景点、前景点和地面点，前景点则认为是可能的动态物体点，通过欧式聚类的方式得到多个点云簇，然后根据这些点云簇的尺寸大小来筛选出不大不小可能是动态物体的点云簇然后从点云中剔除。需要说明的是该方法并不是真正的检测出动态点，只是对可能是车辆的点云进行剔除，实际并不能把所有的车辆点云都剔除掉，但是确实提高了slam的鲁棒性。另外LIO-Livox也对点云进行了特征提取，主要提取了corner 和 平面点，这些点在pose优化中的权重会大一点，剩下的是非常规点，非常规点在非结构化场景中也会提供一些有效约束。

	
2. 高架或者高速基本上没有遮挡物
3. 隧道场景


