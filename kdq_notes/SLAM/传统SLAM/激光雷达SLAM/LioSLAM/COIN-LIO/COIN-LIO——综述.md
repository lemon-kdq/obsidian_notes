
### 1. 概述

opensource：[ COIN-LIO](https://github.com/ethz-asl/COIN-LIO)

coin-lio是在fast-lio2基础上进行开发的，大体了解看该模块利用3D点云的几何和强度信息提取特征点，然后将其投射到图像上，采用图像的关联方法，然后将特征信息加入状态参与EKF更新。该方法可以大大提高系统的鲁棒性，特别是在LID退化场景下应该大有用处。

### 2.测试效果

因为当前只支持Ouster lidar，我们现在目前只有livox lidar数据。


