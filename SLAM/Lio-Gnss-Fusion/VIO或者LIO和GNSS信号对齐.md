
## 1.参考文章

VIO-GNSS: https://blog.csdn.net/hltt3838/article/details/118357973


## 2.坐标系定义

参照[[Lidar和Gnss坐标系对齐]]，并定义如下变量：
	$^WP_L$ : Lidar在***World***坐标系下的坐标
	$^WP_G$ : Gnss天线在***World***下的坐标 
	$^EP_G$ : Gnss天线在***ENU***下的坐标
	$^LP_{GL}$ : Gnss天线到Lidar的向量在***Lidar***坐标系下的坐标 
    $R_{WL}$ : Lidar在***World***下的姿态矩阵
	$T_{WE}$ : ***ENU***到***World***的转换SE3矩阵
因为***World***系定义是在起始位置和***Lidar***坐标系重合，所以我们可得：
 $$
	 T_{WE} = [R_{WE}|^WP_{WE}] = [R_{WE}|^LP_{LG}]
 $$

## 3.GNSS和Lidar外参标定

   如果我们想将GNSS信号融入LO或者LIO系统中，需要知道$T_{WE}$ 和$^LP_{GL}$ ，因为我们可以采集和处理得到Lidar和Gnss天线的Pose信息，根据它们的关系：

   $$
    ^WP_L = T_{WE} * {^EP_G} + R_{WL} * {^LP_{GL}} = R_{WE} * {^EP_G} - ^LP_{GL} + R_{WL} * {^LP_{GL}}
  $$
   我们可以构造Cost_Function来优化变量$T_{WE}$ 和$^LP_{GL}$ ，具体代码如下：

```
  

// 代价函数：测量的变换与估计变换之间的残差

struct PositionCostFunctor {

  EIGEN_MAKE_ALIGNED_OPERATOR_NEW

  PositionCostFunctor(const V3D& lid_position, const M3D& lid_quat,

                      const V3D& gnss_position)

      : lid_position_(lid_position),

        Rwl_(lid_quat),

        gnss_position_(gnss_position) {}

  

  template <typename T>

  bool operator()(const T* const pose1,const T* const pose2, T* residuals) const {

    Eigen::Map<const Eigen::Matrix<T, 3, 1>> t_enu2world(pose1);

    Eigen::Map<const Eigen::Quaternion<T>> q_enu2world(pose2);

    Eigen::Matrix<T, 3, 3> Rwl = Rwl_.cast<T>();

    Eigen::Matrix<T, 3, 1> lid_position = lid_position_.cast<T>();

    Eigen::Matrix<T, 3, 1> gnss_position = gnss_position_.cast<T>();

  

    Eigen::Matrix<T, 3, 1> t_error =

        lid_position -

        (q_enu2world * gnss_position + t_enu2world - Rwl * t_enu2world);

  

    // 将平移误差和旋转误差组合成残差

    residuals[0] = t_error[0];

    residuals[1] = t_error[1];

    residuals[2] = t_error[2];

    return true;

  }

private:

  const V3D lid_position_;

  const V3D gnss_position_;

  const M3D Rwl_;

};
```

## 4. 标定步骤

   **数据采集**：确保车辆在室外GPS信号良好的场所，同时周边有一定高度的物体使得Lidar能够采集到丰富的点云数据，然后驾驶车辆慢速绕“8”字，半径在3-10米左右即可。
   **数据处理**： 
	**lidar odometry**：需要使用Loam或者Fast-LIO获得Lidar的相对起点的$^WP_L$，注意一定时Lidar相    对于World坐标系的Pose。
	**Gnss -> ENU**： GNSS这边采集到的是经纬度信息，我们需要将其转换到ENU坐标系下，同时扣除起点的偏置坐标
	上述两个步骤均可以使用个人开发的[Fast-LIO-SAM]( ssh://git@sourcecode.socialcoding.bosch.com:7999/~kdo1sgh/fast-lio2-sam-loopclosure.git)
   **标定参数**：
	 使用个人开发的软件[gt-gnss2lidar-calibr](ssh://git@sourcecode.socialcoding.bosch.com:7999/~kdo1sgh/gt-gnss2lidar-calibr.git) 执行如下命令：
	 ```
rosrun gt_gnss2lidar calib_gnss2lidar_run lidar_pose_tum gnss_enu_pose_tum output

## 5.结果展示

![[evo_traj.png]]


![[Pasted image 20240521103009.png]]