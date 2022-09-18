# week11

> 2022/09/03 - 2022/09/18

## 上周任务

- [x] 完善第二问：借助`scons --target=cmake`, 实现将microROS嵌入rtthread，cmake负责编译microROS静态库，后续可以选择可以使用cmake或者scons编译整个工程；
- [x] 向 wuhanstudio/micro_ros 提交pr 完成第一二问；
- [x] UDP bug 反馈；

## 周会拟讨论问题

* PR
* UDP 的bug

## microROS嵌入rtthread

shell 脚本 + cmake的方式

CMakeLists.txt

```cmake
# build micro-ROS : make build_microros_lib
add_custom_target(build_microros_lib
	WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/micro-ROS-rtthread-app"
	COMMAND sh generate_microros_library.sh ${CMAKE_C_COMPILER} ${CMAKE_CXX_COMPILER} ${CMAKE_C_FLAGS} ${CMAKE_CXX_FLAGS}
	COMMENT "build micro-ROS..."
)

# head files
INCLUDE_DIRECTORIES(micro-ROS-rtthread-app/microros/build/include)
INCLUDE_DIRECTORIES(micro-ROS-rtthread-app/transports)

#  teansport: serial
add_definitions(-DMICROROS_SERIAL)
add_definitions(-DMICROROS_DEVIVE="vcom")
list(APPEND PROJECT_SOURCES micro-ROS-rtthread-app/transports/rtt_serial_transports.c)

# example 
# pub_int32
add_definitions(-DMICROS_EXAMPLE_PUB_INT32)
list(APPEND PROJECT_SOURCES micro-ROS-rtthread-app/examples/micro_ros_pub_int32.c)

link_directories(${CMAKE_SOURCE_DIR}/micro-ROS-rtthread-app/microros/build)
LINK_LIBRARIES(microros)

```

shell 脚本

* 下载ament相关包
* 编译ament相关包
* 从`CMakeLists.txt` 获取交叉编译链 相关宏定义，为编译microROS提供编译支持
* 下载microROS相关包
* 编译microROS相关包
* 合并成静态库，打包头文件；

## 提交PR

* 改正UDP bug（第一小问）
* 增加两个example: micro_ros_pub_sub_int32.c; micro_ros_ping_pong.c
* build static library for microROS by cmake：摆脱为每种内核保留一个静态库。（第二问）

## UDP bug 反馈

代码应该没有什么问题，估计是rmw 问题，通过修改 ROS 2`RMW_IMPLEMENTATION`环境变量改善。刚开始有效果，但是目前没有效果了。

```
export RMW_IMPLEMENTATION=rmw_fastrtps_dynamic_cpp
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export RMW_IMPLEMENTATION= rmw_cyclonedds_cpp
```



​	

