---
tags:
  - ROS
  - MultiThreadedSpinner
  - 高频数据发布
  - 多线程
---
## 背景

  常用ros的人知道，如果数据发布频率过快，订阅方回调函数来不及处理就会导致数据丢失。ros提供了MultiThreadedSpinner类，可以增加线程来并行处理回调函数。

## 用法

```
#include <ros/ros.h>
#include <std_msgs/String.h>
#include <mutex>

class Listener {
public:
    Listener() {
        sub = nh.subscribe("chatter", 1000, &Listener::chatterCallback, this);
    }

    void chatterCallback(const std_msgs::String::ConstPtr& msg) {
        std::lock_guard<std::mutex> lock(mtx);
        message = msg->data;
        ROS_INFO("I heard: [%s]", message.c_str());
    }

private:
    ros::NodeHandle nh;
    ros::Subscriber sub;
    std::mutex mtx;
    std::string message;
};

int main(int argc, char **argv) {
    ros::init(argc, argv, "listener");
    Listener listener;
    ros::MultiThreadedSpinner spinner(4);
    spinner.spin();
    return 0;
}

```

注意线程安全： 对于回调函数使用的全局变量或者公共资源需要防止同时访问，可以增加锁来避免出现问题，常见的方法有：
	
	1. std::mutex增加互斥锁，每次只可以一个线程访问
	2. std::shared_mutex 多读少写锁，所有线程可以同时读，但是只能一个线程写入
	3. std::atomic 原子操作
