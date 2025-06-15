
## 1. 关键贡献

MINS是一个多传感器紧耦合的融合定位算法框架，它能够将IMU、wheel encoders, cameras, LiDARs和GNSS等多模态的传感器进行融合，从而避免在受限环境下（黑暗环境、空旷场地以及GNSS信号遮挡等）依然能实现鲁棒的定位功能。当前的资料如下:
Paper： [[MINS_Efficient_and_Robust_Multisensor-aided_INS.pdf]]
Code： [MINS](https://github.com/rpng/MINS)
解决问题：
- 基于滤波的多传感器紧耦合算法，能够融合IMU、wheel encoders, cameras, LiDARs和GNSS等传感器，解决因为某些传感器量测退化、失效或者外点等异常情况下的准确定位能力
- 所提出的融合框架能够处理非同步的有时间延迟的传感器信号，而且可以很方便的添加附加传感器，同时算法能够平衡计算复杂度和精度
- 传感器内外参实时校准，基于自感传感器IMU和wheel encoders实现可靠的初始化方法
## 2. 技术框架

![[Pasted image 20240613145937.png]]


- **Initialization**
	使用camera 的SFM和IMU预积分技术来初始化常常会导致尺度不准确，比如匀速或者匀加速场景；使用Lidar和IMU又会因为lidar场景致使ICP误差传入初始状态，因此这里使用的是IMU和轮速的动态初始化，当然这一步初始化并无法获得ENU下的状态，因此在GNSS信号可靠时再进行一次初始化。
![[Pasted image 20240618200118.png]]
- **Polynomial Adaptive On-Manifold Interpolation**: 基于多项式的流形插值方法——为了解决有延迟和非同步的测量数据更新，采用多项式流形插值方式来避免模型误差。对比线性插值，多项式插值更加贴近真实的运动模型，能够应对高动态和敏感传感器带来的不一致问题。
	![[Pasted image 20240618190932.png]]
	对于存在非同步误差的量测数据，也可以将时间误差建模进去一起进行滤波：
	![[Pasted image 20240618191650.png]]
- **Multi Sensor Fusion Framework**
   基于MSCKF来实现当前状态和local历史状态的更新，该方案的好处就是可以处理不同频率、带有延迟的传感器信号。
   1. **状态定义：**
		![[Pasted image 20240617163416.png]]
	多状态EKF中定义的状态包括当前状态和历史pose，当前状态包括位置、姿态和速度以及加速度计和陀螺仪的零偏，而历史状态pose只包括历史的pose信息。其中E表示ENU坐标系，I表示IMU坐标系，定义pose时ENU相对IMU坐标系，这样好处是当GNSS信号有效的话，我们可以很方便的融合GNSS信息，如果在初始化阶段则Pose是定义在我们自己初始化的global坐标系。
	
	**2.量测模型**
	对于camera和lidar都有很多的特征点，因为无法将这些特征点加入状态中，所以采用的是左零空间的映射只根据这些特征点来更新状态，这样可以大大提高计算效率而精度也是不错的。有关这部分知识可以了解一下MSCKF。
	 - Camera更新：camera使用的是经典的重投影误差，这里面需要注意的是MINS据说对相机的内外参均进行了建模，想当于camera内外参的online calibration。之后为了防止状态维度太大导致的计算量过大问题，使用了MSCKF更新方法，在滤波的时候只更新状态何其置信度不更新特征点。
	 - Lidar更新：point-to-plane的方法进行点云量测模型的建模，首先是选择距离比较近的几个点云，然后将其转到local map坐标系下，在local map中找到对应的点云计算其特征向量，如果该特征向量满足一定约束条件，说明其是平面的特征向量；然后计算根据点在面上的约束来构造量测模型。注意这部分的点特别多所以也是采用MSCKF的方式只更新位姿状态，点云通过更新状态后续计算得出。***但是这里面没有介绍点云的动态正畸，需要额外关注。***


- **Online Calibration**
	Lidar、Camera、Wheel Encoder和GNSS量测模型中都带有各自传感器相对于IMU的外参，因此这些外参都会得到实时的校准。

![[Pasted image 20240618195407.png]]
![[Pasted image 20240618195446.png]]

  这些内外参都会增扩到状态中，作者说明这些参数并不是全都加入状态中，添加的是参数不确定的。上述参数有内参也有外参甚至有非同步时间参数。
  
## 3. 仿真工具

![[2024-06-13_16h49_41.gif]]



## 4. 精度和误差分析

- 传感器内外参摄动测试

![[Pasted image 20240618201307.png]]
从上面看加入online calibration不管时时间误差或者是内外参的误差系统状态能够快速收敛，而不进行online calibration就会如上述的红色线导致误差变大。

- Benchmark Score
![[Pasted image 20240618201744.png]]
## 5. 代码深挖

### 5.1 IMU Update Pipeline


### 5.2 Wheel Odometry Update Pipeline


### 5.3 Lidar Update Pipeline

[[MINS-Lidar Update Pipeline]]

### 5.4 Camera Update Pipeline

### 5.5 GNSS Update Pipeline








