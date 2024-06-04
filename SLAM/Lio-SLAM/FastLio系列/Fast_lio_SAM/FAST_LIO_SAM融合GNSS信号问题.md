---
tags:
  - FAST_LIO_SAM
  - GPS
  - GNSS
  - Fusion
---
## 1. GPS信号坐标系转换问题
	1. UTM坐标系
		之前Lijia提供的GPS转到UTM坐标系下的坐标和SLAM输出对比发现SLAM存在很严重的尺度误差
	cmd: evo_ape tum gps_utm.txt lidar_pos_out.txt  -a  -p  -v 
	result:
		APE w.r.t. translation part (m)
		(with SE(3) Umeyama alignment)
		       max      8.563742
		      mean      3.613755
		    median      3.296584
		       min      0.141922
		      rmse      4.213433
		       sse      48394.717601
		       std      2.166515


![[Pasted image 20240510112802.png]]![[Pasted image 20240510112948.png]]

	2. ENU坐标系
		讲GPS坐标转ENU坐标系，不存在系数误差问题：
		cmd:
		result:
			APE w.r.t. translation part (m)
			(with SE(3) Umeyama alignment)
		       max      2.095144
		      mean      1.218763
		    median      1.169024
		       min      0.482763
		      rmse      1.248828
		       sse      3591.692158
		       std      0.272376


		![[Pasted image 20240510113527.png]]
		![[Pasted image 20240510130403.png]]


## 2. GNSS信号和LIO坐标系对齐问题

1. GNSS转ENU坐标系之后，需要LIO坐标系和ENU坐标系进行对齐，如果LIO中使用的是6-axis轴IMU，基本上不可能事先静态对齐两个坐标系，因为静止状态无法估计LIO和ENU坐标系之间的航向差异，roll和pitch可以直接通过加速度计在静止状态下的数据估计个大体初始值，但是yaw没有磁力计是无法获得准确的初始值的。因此，我们只能动态情况下对齐航向。

2. 另一个需要注意的是，GNSS信号是GPS天线的位置信息，而我们LIO是IMU所在的位置，所以二者还需要一个外参的转化。转换完之后的对齐如下：


	![[Pasted image 20240517142740.png]]
![[Pasted image 20240517142847.png]]
## 3. GNSS和Lidar外参标定和对齐

**上面的内容在对齐GNSS和Lidar坐标系的时候使用EVO自带的Umeyama算法，该算法的模型是针对两个位置重合的传感器来对齐它们的位置信息的，如果GNSS天线和Lidar按照位置相差较远则不可以用Umeyama算法。**

个人开发了新的GNSS和Lidar外参标定方法，可参照文章[[VIO或者LIO和GNSS信号对齐]]

最终结果如下：


![[evo_traj.png]]
![[Pasted image 20240521103009.png]]

## 4. Lane Marker在地图上的标记


![[Pasted image 20240522085110.png]]

![[Pasted image 20240522134655.png]]
下图事路标点采集时候的路段图片：
采集点在道路线上

![[GT03_CAM_1_compressed_image-1672533495-552999936.png]]
![[GT03_CAM_1_compressed_image-1672533515-72000000.png]]

![[GT03_CAM_1_compressed_image-1672533518-283000064 1.png]]

![[GT03_CAM_1_compressed_image-1672533530-92000000 2.png]]
## 5.当前的问题和改进可能方法

**当前问题：**
1. 上图红色矩形框可以发现在弯道地段，明显路标点没有打在两侧的车道线上，但是弯道之前的直线路段还是比较准确的打在了车道线上的，而出弯后虽然路标点和车道线有所偏离，但是相比弯道路段还是误差小很多的。
2. 当前采用的ENU和LIO坐标系对齐的方式是假设LIO和ENU给出的误差都比较小，而且这些数据是离线处理和获取的，后续也只能适用于后融合方式。
3. 融合GPS之后反而让定位误差增大，需要进一步改进GPS和LIO融合框架和算法。


**解决方案**：
统筹分析，上述问题的首要原因是实现ENU和LIO的坐标系对齐。这里可行的方案如下：
1. 9轴IMU为初始化阶段对齐ENU坐标系：当前我们使用的是两个非同源传感器的pose信息采用优化方式来对齐坐标系，后续我们应该使用带磁力计的9-IMU来获取ENU和LIO坐标系的姿态对齐，从而为实时的融合做好准备；
2. 重构优化状态和量测更新：即便初始化阶段已经获得了姿态对齐，但是该对齐还是存在误差的，包括gnss天线和lidar的相对位置信息也都是存在误差的，因此我们在设计滤波器的时候应该将它们的外参作为状态变量实时更新和优化；
3. 当前GPS和Lidar的优化方法使用的是因子图优化的方式，而且只考虑pose位姿之间的约束以及GNSS绝对位置约束，没有考虑scan和map之间的match约束，这个也是不可取的，应该将scan和match约束加入其中，否则导致pose虽然得到了约束但是建出的图出现错误匹配的问题。

## 6.问题追踪

![[Pasted image 20240522185429.png]]
上面绿色是lane_marker和gnss pose在ENU坐标系下的图像，可以看到lane marker是分布在gnss pose两边的。而下面红色部分是lane_marker和gnss pose在LIO坐标系下的，这部分可以发现他是错的，没有分布在gnss pose两边，所以应该有两个原因：1）ENU和LIO外参不准；2）使用的Rwl不是很准的时刻，导致修正的gnss pose不准。

**问题解决：**
上述原因均不对，之所以出现上问题是lane marker转换到LIO坐标系公式不对，修改：
```
           V3D ptInENU( gnss_data.local_E, gnss_data.local_N, gnss_data.local_U + (i-8) * 0.2);
           不需要这个补偿
          // V3D ptInLIO = gnss_to_lidar * ptInENU - GeoQuat * Gnss_T_wrt_Lidar;
           V3D ptInLIO = gnss_to_lidar * ptInENU;  //  gnss 转到 lidar 系下
           
```

![[Pasted image 20240523091318.png]]

![[Pasted image 20240523091111.png]]![[Pasted image 20240523091133.png]]


## 7. 评估定位和建图精度

### 1）融合后GNSS信号的定位精度如下：
![[Pasted image 20240527135612.png]]
该精度略低于fast-lio不融合gnss的表现，可能原因：1) ENU和LIO的外参估计有误差；2）LIO和GNSS信号时间不能严格对齐。
![[Pasted image 20240527134928.png]]![[Pasted image 20240527135551.png]]

### 2） 融合GNSS信号后的建图精度

![[Pasted image 20240527131652.png]]

上图是将路标点转到LIO坐标系下和地图在俯视角度的对比图，红点是用RTK打在车道线内测的点，而所建的地图在打点的附件也出现高亮的点云，这些点云即是道路线，因为无法出3D车道线，只能在CC中大体测定红点到最近3D车道线内测距离，横向误差低于5cm。
