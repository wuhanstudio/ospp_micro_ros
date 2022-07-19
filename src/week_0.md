# Week 0

> 2022/06/27 - 2022/07/03

## 前提

虚拟机上安装Ubuntu20.4，ROS2安装foxy

## 工作内容

1. microROS在linux上使用；
2. microROS在rtthread上使用；

### microROS在linux上使用

### 目的

1. 初步尝试microROS；

1. 查看microROS的源码的组成部分；

### 结果

按照[First micro-ROS Application on Linux](https://eur03.safelinks.protection.outlook.com/?url=https%3A%2F%2Fmicro.ros.org%2Fdocs%2Ftutorials%2Fcore%2Ffirst_application_linux%2F&data=05|01|hw630%40exeter.ac.uk|ed85ac1384874bc22a5208da577fc661|912a5d77fb984eeeaf321334d8f04a53|0|0|637918503559592081|Unknown|TWFpbGZsb3d8eyJWIjoiMC4wLjAwMDAiLCJQIjoiV2luMzIiLCJBTiI6Ik1haWwiLCJXVCI6Mn0%3D|3000|||&sdata=g3W5ZeTgvvpbWF9Ap6%2BcIhbwq%2BLdUOldDnTp7ZU8U20%3D&reserved=0)构建microROS和micro-ROS agent。由于运行`ros2 run micro_ros_setup build_firmware.sh`和`ros2 run micro_ros_setup build_agent.sh`报错：

```
Compiling for host environment: not cleaning path
Building firmware for host platform generic
usage: colcon [-h] [--log-base LOG_BASE] [--log-level LOG_LEVEL]
              {build,test,test-result} ...
colcon: error: unrecognized arguments: --packages-up-to rosidl_typesupport_microxrcedds_c --metas src
```

所以直接使用`colcon build`

服务端：

```
ros2 run micro_ros_agent micro_ros_agent udp4 -p 8888
```

客户端：

```
ros2 run micro_ros_demos_rclc ping_pong 
```

客户端报错：

```
[ERROR] [1656251587.172644707] [rclc]: [rclc_publisher_init_best_effort] Error in rcl_publisher_init: Undefined type support, at /home/haijun/Desktop/test/src/uros/rmw_microxrcedds/rmw_microxrcedds_c/src/rmw_publisher.c:127, at /tmp/binarydeb/ros-foxy-rcl-1.1.13/src/rcl/publisher.c:180
```

从error来看，应该是`rclc_publisher_init_best_effort`函数出错。

具体原因未知

### microROS在rtthread上使用

### 目的

1. 尝试rtt能否接入ROS2

### 结果

rtt客户端：

连接方式：udp

运行时，从rtt终端终端提示来说，线程能正常运行。



ros agent 服务端：

```
ros2 run micro_ros_agent micro_ros_agent udp4 -p 8888
[1656252343.224844] info     | UDPv4AgentLinux.cpp | init                     | running...             | port: 8888
[1656252343.225293] info     | Root.cpp           | set_verbose_level        | logger setup           | verbose_level: 4
[1656252350.362219] info     | Root.cpp           | create_client            | create                 | client_key: 0x7097FB67, session_id: 0x81
[1656252350.362475] info     | SessionManager.hpp | establish_session        | session established    | client_key: 0x7097FB67, address: 192.168.31.73:704
[1656252350.400982] info     | ProxyClient.cpp    | create_participant       | participant created    | client_key: 0x7097FB67, participant_id: 0x000(1)
```

重新查看rtt端的代码，发现`rclc_publisher_init_default()`的返回值不为零，可以定位创建publish失败。原因未知。

## 总结

在linux端的rtt端运行microROS均失败，定位到的错误均由于`publisher_init`初始化失败导致。目前不知道原因为何，猜测是不是`micro-ROS agent`构建有问题。

## 下一步计划

1. rtt端测试下uart通信，看是否出现同一样问题；
2. 使用docker 尝试下。
