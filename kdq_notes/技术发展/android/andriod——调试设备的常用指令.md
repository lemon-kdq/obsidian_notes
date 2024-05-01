---
日期: 2024-04-23T21:05:00
作者: 孔大庆
tags:
  - android
  - 调试
  - debug
  - 指令
---
### 1. 进入无线调试功能
	1. pc和andriod设备在同一局域网
	2. andriod开启usb调试模式和允许无限调试
	3. android设备用usb线连接电脑
	4. adb shell进入android设备，查看android ip地址
	5. pc terminal端输入下面指令
		- adb tcpip 5555 
		- adb connect andriod_ip:5555
	6. 断开usb线，adb shell进入即可

### 2. adb 常用指令
	1. 安装apk： adb install *.apk
	2. 