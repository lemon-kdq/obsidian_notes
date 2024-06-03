---
tags:
  - C
  - C_Plus
  - Debug
  - coding
  - gdb
---
# Platform

	针对UBUNTU系统调试c或者c++代码，常常需要定位bug的位置，我们可以使用gdb去定位

# 编译设置

对于使用CMake去编译的程序，我们需要进入CMakeLists.txt中加入如下字样：

```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  -g2 -ggdb}")

set(CMAKE_BUILD_TYPE Debug)
```

# 启动gdb调试

1. 对于不带参数的可执行文件：

	$gdb ./A

2. 对于带参数的可执行文件：

	**$gdb --args ./A V1 V2 V3**

	$gdb ./A，进入gdb后 r V1 V2 V3

	$gdb ./A，进入gdb后 设置参数set args V1 V2 V3 再直接 r。


3. 对于rosnode，我们需要进入devel/lib中直接的可执行文件，然后按照：
	**$gdb --args ./A V1 V2 V3**
	或者（还未测试）
	 rosrun --prefix 'gdb -ex run --args'  rosnode rosexec arg1
4. 对于roslaunch进行gdb调试的适合需要在launch文件中加入：
		sudo apt install xterm
	```xml
  <node pkg="your_package" type="your_node" name="your_node" output="screen" launch-prefix="xterm -e gdb --args" />

```
# 使用gdb定位问题

进入gdb环境后，可以使用如下指令：

- run  - 执行程序
- where - 定位出错的位置
-
