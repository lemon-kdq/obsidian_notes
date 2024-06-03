
### 1. Lidar and Camera Time Synchronization Evaluation

Lidar和Camera时间对齐的验证方案如下：


![[Pasted image 20240603104444.png]]
上述方案是用于定量评估lidar和camera的时间对齐误差的方案，但是实际上从定性角度就能发现当前lidar和camera时间对齐的误差较大且是时变误差。

### 2. Time offset Error 

所谓的定性分析二者传感器的时间同步误差是舍弃了lidar点云和image在特征层面的定性对齐误差分析，直接将点云投影到image上，观察其是否能在运动状态特别是转弯场景下点云能否和对应图像的物体很好的重合，最终结果如下：

[lidar和camera时间对齐评估]("C:\Users\KDO1SGH\Desktop\kdq_important_no_delete\project\ground_truth\lidar_camera_timestamp_synctronization\lidar_and_camera_timestamp_sync.mp4")

视频中的控制条可以调整图像的时间offset，以及控制视频暂停，启动和播放速度，演示的是lidar1将点云投影的cam0，1和2，4四个前向和左前和右前的摄像头，中间是cam1摄像头。


![[2024-06-03_10h56_24.gif]]