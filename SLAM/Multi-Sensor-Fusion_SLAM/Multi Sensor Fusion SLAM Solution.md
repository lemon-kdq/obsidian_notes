## 1. MSF-SLAM Architecture

![[Pasted image 20240603161924.png]]
## 2. Modules Design

### 1. Initialization 
**目标：**  MSF-SLAM Initialization设计目标是将不同传感器都统一到相同的时空下，对于这里面需要设计到车身坐标系，ENU坐标系，Lidar坐标系和IMU坐标系的外参实时估计，同时有可能需要估计出不同传感器之间的Time offset。
**方法：**
https://github.com/hku-mars/LiDAR_IMU_Init

### 2. Time Alignment
**目标：** 不同传感器频率不同，在融合的时候需要确保时间完全对齐，Time Alignment可能不是一个独立的模块，但应该在每一个传感器中都考虑到该设计目的。
**方法：** B-Spline或者LIO的前向和后向递推


### 3. Fusion Algorithm

#### 1. State Definition




#### 2. Front-End Algorithm
**目标**：
 
#### 3. Back-End Algorithm


### 4. Function Safety Assessment

**目标：** GNSS容易因为信号遮挡、多路径效应等因素导致数据不可信，lidar也会有隧道、空旷场地、长廊和动态目标过多等信息维度退化的问题，所以需要设计功能安全模块来实时监测和评估当前的传感器信号的有效性，从而及时隔离有问题的传感器信号。






