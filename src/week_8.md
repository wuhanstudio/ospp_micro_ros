# week8

> 2022/08/08 - 2022/08/012

## 上周任务：

* 理清msg类型转换为c的流程；

## 完成情况

* 没有理清msg转化为c的流程，但是可以找到转化层c的源码；
* 结合msg源码、rclc、rcl、rmw基于cmake构建microROS静态库。因此，基于固定的microROS版本，可以实现基于scons构建；

> 注：基于cmake构建出来的静态文件只支持std_msgs包含的数据类型，另外只使用`microros_pub_int32`在单片机上测试，所以构建出来的静态文件只能说明`rclc、rcl、rmw`包含完整，不能实现完整的ros2功能(其他的msg类型、srv、action)

* 大致分离出`rclc、rcl、rmw`，第二问可以算初步完成，详见[基于cmake构建microros(裁剪版)](#基于cmake构建microros(裁剪版))

## 周会拟讨论问题

1. 目前有希望直接通过scons构建microROS：

   优点：

   * microROS包和rtt中的其他软件包使用流程一致；
   * 使用源码构建，可以解决目前为每款单片机的创造一个静态文件的问题；

   缺点：

   * 源代码无法和官方保持同步；

2. 关于第三问的处理方式；

## 日志

### std_msgs

#### 源码

解析std_msg package需要以下四个package:

* `rosidl_generator_c`： 提供ROS接口 ;
* `rosidl_typesupport_introspection_c`: 提供消息类型支持;
* `rosidl_default_generators`
* `rosidl_typesupport_microxrcedds_c`为Micro XRCE-DDS提供接口;

**最终生成的 c 可以在`build/std_msg`下面找到源码**（使用microROS提供的命令编译的目录下）

源码文件结构：

```bash
haijun@haijun-Lenovo:~/micro_ROS/cmake_microros/std_msgs$ tree -L 3
.
├── std_msgs_rosidl_generator_c
│   ├── CMakeLists.txt
│   └── std_msgs
│       └── msg
├── std_msgs_rosidl_typesupport_c
│   ├── CMakeLists.txt
│   └── std_msgs
│       └── msg
├── std_msgs_rosidl_typesupport_introspection_c
│   ├── CMakeLists.txt
│   └── std_msgs
│       └── msg
└── std_msgs_rosidl_typesupport_microxrcedds_c
    ├── CMakeLists.txt
    └── std_msgs
        └── msg
```

#### 编译

* 对应文件单独编译成静态文件

```cmake

file(GLOB_RECURSE SRC_DIR_LIST ${PROJECT_SOURCE_DIR}/std_msgs/msg/detail/*.c)
add_library(${PROJECT_NAME}  ${SRC_DIR_LIST})
target_include_directories(${PROJECT_NAME} PUBLIC
    ${PROJECT_SOURCE_DIR}
    ${PARA_INCLUDE_PATH}
  )
  install(
    TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
  )
```

> 注意：`std_msgs_rosidl_typesupport_c`需要添加一个宏定义`add_definitions(-DROSIDL_TYPESUPPORT_STATIC_TYPESUPPORT)`表示使用静态类型

### 基于cmake构建microros(裁剪版)

使用的文件：

**rmw**

* Micro-CDR：实施CDR标准序列化方法的C库  位置: `/firmware/eProsima/Micro-CDR`
* Micro-XRCE-DDS-Client：DDS客户端   位置：`/firmware/eProsima/Micro-XRCE-DDS-Client`
* rmw：中间件，为不同的DDS实现提供了一个抽象层  位置：`/firmware/ros2/rmw/rmw`
* rmw_microxrcedds_c： 为XRCE-DDS（RMW层）提供了中间件实现 位置`/firmware/uros/rmw_microxrcedds/rmw_microxrcedds_c`

**数据类型转化（idl）：**

* rosidl_runtime_c:提供定义，初始化和最终化功能以及用于获取和使用rosidl typesupport类型的宏  位置：`/firmware/ros2/rosidl/rosidl_runtime_c`
* rosidl_typesupport_microxrcedds_c： 为microxrcedds提供能支持idl的接口  位置:`/firmware/uros/rosidl_typesupport_microxrcedds/rosidl_typesupport_microxrcedds_c`
* rosidl_typesupport_c：为idl提供c 语言的接口   位置：`/firmware/uros/rosidl_typesupport/rosidl_typesupport_c`

**rcl**

* rcutils: 为ROS2提供数据结构（C API）位置:`firmware/uros/rcutils`
* rcl_logging_interface ：提供日志函数  位置`/firmware/ros2/rcl_logging/rcl_logging_interface`
* rcl_logging_noop: 提供日志函数    位置：`/firmware/ros2/rcl_logging/rcl_logging_noop`
* tracetools:追踪工具   位置：`/firmware/uros/tracetools/tracetools`
* rcl： ros客户端库  位置：`/firmware/uros/rcl/rcl`

**rclc**

* rclc：ros客户端c库  位置`/firmware/uros/rclc/rclc`

**std_msg**

* std_msgs_rosidl_generator_c
* std_msgs_rosidl_typesupport_c
* std_msgs_rosidl_typesupport_introspection_c
* std_msgs_rosidl_typesupport_microxrcedds_c

根据以上文件生成的静态文件:

```bash
.
├── libmicrocdr.a
├── libmicroxrcedds_client.a
├── librcl.a
├── librclc.a
├── librcl_logging_interface.a
├── librcl_logging_noop.a
├── librcutils.a
├── librmw.a
├── librmw_microxrcedds.a
├── librosidl_runtime_c.a
├── librosidl_typesupport_c.a
├── librosidl_typesupport_microxrcedds_c.a
├── libstd_msgs_rosidl_generator_c.a
├── libstd_msgs__rosidl_typesupport_c.a
├── libstd_msgs_rosidl_typesupport_introspection_c.a
├── libstd_msgs_rosidl_typesupport_microxrcedds_c.a
├── libtracetools.a

```

合并成一个静态文件：

```bash
# 合并静态文件
cd ${Dir_Install}/lib

for file in $(find -name '*.a'); do
    echo ${file}
    folder=$(echo $file | sed -E "s/(.+)\/(.+).a/\2/"); 
    mkdir -p $folder; 
    cd $folder; 
    ar x ../$file;
    for f in *; do 
			mv $f ../$folder-$f; 
		done; 
    cd ../
    rm -rf $folder
done
ar rc libmicroros.a *.obj
rm *.obj

mv libmicroros.a ${Base_File}
echo "mv libmicroros.a to ${Base_File}"
```

最终生成的静态文件：

```bash
1.2M	libmicroros.a
```

