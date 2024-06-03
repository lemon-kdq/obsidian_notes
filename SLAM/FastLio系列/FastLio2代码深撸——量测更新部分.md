---
tags:
  - SLAM
  - fastliosam
  - 量测更新
aliases:
---
# 1.量测模型

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

