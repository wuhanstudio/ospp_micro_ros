# micro-ROS[RT-Thread]：自定义编译静态库

* ROS2版本：galactic
* 单片机：art-pi

## 1 环境准备

参考链接：

* https://micro.ros.org/docs/tutorials/advanced/create_custom_static_library/
* https://micro.ros.org/docs/tutorials/core/first_application_rtos/freertos/

### 1. 1 准备`micro_ros_setup`

```bash
# 创建micro ros工具
mkdir microros_ws
cd microros_ws
git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup

sudo apt update && rosdep update
rosdep install --from-paths src --ignore-src -y

colcon build
source install/local_setup.bash
```

### 1.2 准备`firmware`

```bash
ros2 run micro_ros_setup create_firmware_ws.sh generate_lib
```

**可能出现的问题1**：`vcs not found `

```bash
vcs not found 
command 'vcs' not found , but there are 17 similar ones.
```

缺少` python3-vcstool`软件包

```bash
sudo apt-get install python3-vcstool
```

**可能出现的问题2**：`colcon: error: unrecognized arguments:`

```bash
colcon: error: unrecognized arguments: --packages-ignore-regex
```

`coclon`安装不全

```bash
pip install -U colcon-common-extensions
```

正确下载firmware和编后的文件夹结构如下：

```bash
$ tree -L 2

.
├── COLCON_IGNORE
├── dev_ws
│   ├── ament
│   ├── build
│   ├── install
│   ├── log
│   ├── ros2
│   └── ros2.repos
├── mcu_ws
│   ├── build
│   ├── colcon.meta
│   ├── eProsima
│   ├── install
│   ├── log
│   ├── ros2
│   ├── ros2.repos
│   └── uros
└── PLATFORM
```

### 1.3 编译 静态文件库

定义交叉编译规则：`my_custom_toolchain.cmake`

```cmake
# 使用交叉编译 目标：嵌入式平台
SET(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_CROSSCOMPILING 1)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
# 交叉编译链:参考单片机端的rtconfig.py编译版本
set(TOOLS /home/haijun/env_released_1.2.0/gcc-arm-none-eabi-5_4-2016q3/bin/)
set(CMAKE_C_COMPILER ${TOOLS}arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER ${TOOLS}arm-none-eabi-g++)

SET(CMAKE_C_COMPILER_WORKS 1 CACHE INTERNAL "")
SET(CMAKE_CXX_COMPILER_WORKS 1 CACHE INTERNAL "")


# SET HERE YOUR BUILDING FLAGS：参考rtconfig.py的DEVICE的配置

set(FLAGS "-O2 -mfloat-abi=softfp -mfpu=fpv5-sp-d16 -mfloat-abi=hard  -ffunction-sections -fdata-sections -nostdlib --param max-inline-insns-single=500 -fno-exceptions -mcpu=cortex-m7 -DF_CPU=480000000L -DARDUINO=10813 -mthumb -DSTM32H750XX" CACHE STRING "" FORCE)


set(CMAKE_C_FLAGS_INIT "-std=c11 ${FLAGS} -DCLOCK_MONOTONIC=0 -D'__attribute__(x)='" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS_INIT "-std=c++11 ${FLAGS} -fno-rtti -DCLOCK_MONOTONIC=0 -D'__attribute__(x)='" CACHE STRING "" FORCE)

set(__BIG_ENDIAN__ 0)

```



定义静态库包含的头文件：`my_custom_colcon.meta`

```
{
    "names": {
        "tracetools": {
            "cmake-args": [
                "-DTRACETOOLS_DISABLED=ON",
                "-DTRACETOOLS_STATUS_CHECKING_TOOL=OFF"
            ]
        },
        "rosidl_typesupport": {
            "cmake-args": [
                "-DROSIDL_TYPESUPPORT_SINGLE_TYPESUPPORT=ON"
            ]
        },
        "rcl": {
            "cmake-args": [
                "-DBUILD_TESTING=OFF",
                "-DRCL_COMMAND_LINE_ENABLED=OFF",
                "-DRCL_LOGGING_ENABLED=OFF"
            ]
        }, 
        "rcutils": {
            "cmake-args": [
                "-DENABLE_TESTING=OFF",
                "-DRCUTILS_NO_FILESYSTEM=ON",
                "-DRCUTILS_NO_THREAD_SUPPORT=ON",
                "-DRCUTILS_NO_64_ATOMIC=ON",
                "-DRCUTILS_AVOID_DYNAMIC_ALLOCATION=ON"
            ]
        },
        "microxrcedds_client": {
            "cmake-args": [
                "-DUCLIENT_PIC=OFF",
                "-DUCLIENT_PROFILE_UDP=OFF",
                "-DUCLIENT_PROFILE_TCP=OFF",
                "-DUCLIENT_PROFILE_DISCOVERY=OFF",
                "-DUCLIENT_PROFILE_SERIAL=OFF",
                "-UCLIENT_PROFILE_STREAM_FRAMING=ON",
                "-DUCLIENT_PROFILE_CUSTOM_TRANSPORT=ON"
            ]
        },
        "rmw_microxrcedds": {
            "cmake-args": [
                "-DRMW_UXRCE_MAX_NODES=1",
                "-DRMW_UXRCE_MAX_PUBLISHERS=10",
                "-DRMW_UXRCE_MAX_SUBSCRIPTIONS=5",
                "-DRMW_UXRCE_MAX_SERVICES=1",
                "-DRMW_UXRCE_MAX_CLIENTS=1",
                "-DRMW_UXRCE_MAX_HISTORY=4",
                "-DRMW_UXRCE_TRANSPORT=custom"
            ]
        }
    }
}
```

编译静态库：

```bas
ros2 run micro_ros_setup build_firmware.sh $(pwd)/my_custom_toolchain.cmake $(pwd)/my_custom_colcon.meta
```

一共编译会编译64个包，其中会出现一些警告。编译时间1min多

```bash
Summary: 64 packages finished [1min 10s]
  40 packages had stderr output: action_msgs actionlib_msgs builtin_interfaces composition_interfaces diagnostic_msgs example_interfaces geometry_msgs libyaml_vendor lifecycle_msgs micro_ros_msgs micro_ros_utilities microxrcedds_client nav_msgs rcl rcl_action rcl_interfaces rcl_lifecycle rcl_logging_interface rcl_logging_noop rclc rclc_lifecycle rclc_parameter rcutils rmw rmw_implementation rmw_microxrcedds rosgraph_msgs rosidl_runtime_c rosidl_typesupport_c rosidl_typesupport_microxrcedds_c sensor_msgs shape_msgs statistics_msgs std_msgs std_srvs stereo_msgs test_msgs trajectory_msgs unique_identifier_msgs visualization_msgs
```

静态库和头文件在`firmware/build`中

```bash
haijun@haijun-Lenovo:~/micro_ROS/microros_ws/firmware/build$ tree -L 2
.
├── include
│   ├── actionlib_msgs
│   ├── action_msgs
│   ├── builtin_interfaces
│   ├── composition_interfaces
│   ├── diagnostic_msgs
│   ├── example_interfaces
│   ├── geometry_msgs
│   ├── lifecycle_msgs
│   ├── micro_ros_msgs
│   ├── micro_ros_utilities
│   ├── nav_msgs
│   ├── rcl
│   ├── rcl_action
│   ├── rclc
│   ├── rclc_lifecycle
│   ├── rclc_parameter
│   ├── rcl_interfaces
│   ├── rcl_lifecycle
│   ├── rcl_logging_interface
│   ├── rcutils
│   ├── rmw
│   ├── rmw_microros
│   ├── rmw_microxrcedds_c
│   ├── rosgraph_msgs
│   ├── rosidl_runtime_c
│   ├── rosidl_typesupport_c
│   ├── rosidl_typesupport_interface
│   ├── rosidl_typesupport_introspection_c
│   ├── rosidl_typesupport_microxrcedds_c
│   ├── sensor_msgs
│   ├── shape_msgs
│   ├── statistics_msgs
│   ├── std_msgs
│   ├── std_srvs
│   ├── stereo_msgs
│   ├── test_msgs
│   ├── tracetools
│   ├── trajectory_msgs
│   ├── ucdr
│   ├── unique_identifier_msgs
│   ├── uxr
│   ├── visualization_msgs
│   └── yaml.h
└── libmicroros.a

43 directories, 2 files
```

## 1.4 部署到单片机端

参考链接：https://github.com/ros2middleware/micro_ros_RTThread_apps/tree/galactic

基于micro_ros_RTThread_apps的配置：

* Device type (UART)  
* ARCH CPU (Cortex M7 (fpv5-d16-hard))  ---> 
* uart1) serial device name 
* [*]   microros_pub_int32: microros publish int32 example  
* [*]   microros_sub_int32: micro ros subscribe int32 example
* Version (galactic)  ---> 

由于生成的头文件和原有micro_ros-galactic的头文件有出入 ，为了快速部署，修改了micro_ros-galactic的文件布局：

* 新增`include`文件夹，防止`firmware`生成的头文件
* 把原来`src`文件夹中的`micro_ros_rtt.h`,  `yaml.h`
* libmicroros.a直接放在`micro_ros-galactic`w文件下
* `src`文件夹只保留`.c`文件

因此会导致micro_ros的以下配置失效：

* ARCH CPU (Cortex M7 (fpv5-d16-hard))  ---> 

再稍微修改`SConscript`

```
# 获取静态文件的位置
LIBPATH = [cwd]
# 获取头文件位置
path	= [cwd]
path	+= [cwd + '/include']
```

**编译中可能出现错误：**

```bash
In file included from packages/micro_ros-galactic/include/rmw/qos_profiles.h:23:0,
                 from packages/micro_ros-galactic/include/rmw/rmw.h:103,
                 from packages/micro_ros-galactic/include/rmw_microros/rmw_microros.h:20,
                 from packages/micro_ros-galactic/include/micro_ros_rtt.h:13,
                 from packages/micro_ros-galactic/examples/micro_ros_pub_int32.c:5:
packages/micro_ros-galactic/include/rmw/types.h:418:49: error: expected ',' or '}' before '__attribute__'
 # define RMW_DECLARE_DEPRECATED(name, msg) name __attribute__((deprecated(msg)))
                                                 ^
packages/micro_ros-galactic/include/rmw/types.h:439:3: note: in expansion of macro 'RMW_DECLARE_DEPRECATED'
   RMW_DECLARE_DEPRECATED(
   ^
In file included from packages/micro_ros-galactic/include/rmw/rmw.h:103:0,
                 from packages/micro_ros-galactic/include/rmw_microros/rmw_microros.h:20,
                 from packages/micro_ros-galactic/include/micro_ros_rtt.h:13,
                 from packages/micro_ros-galactic/examples/micro_ros_pub_int32.c:5:
packages/micro_ros-galactic/include/rmw/qos_profiles.h:111:3: error: 'RMW_QOS_POLICY_LIVELINESS_UNKNOWN' undeclared here (not in a function)
   RMW_QOS_POLICY_LIVELINESS_UNKNOWN,
```

知道错误由`define RMW_DECLARE_DEPRECATED(name, msg) name __attribute__((deprecated(msg)))`引起，但是不知道原有，因此删除`__attribute__((deprecated(msg)))`

```c
// 位置：/micro_ros-galactic/include/rmw/types.h:418
# define RMW_DECLARE_DEPRECATED(name, msg) name 
```

## 1.5 运行

**出现问题**: 建立连接失败

单片机：

```bash
msh >microros_pub_int32
[E/micro_ros_serial] succeed to open device uart1
[micro_ros] failed to initialize
msh >
```

PC 端：

```bash
haijun@haijun-Lenovo:/dev$ ros2 run micro_ros_agent micro_ros_agent serial -D /dev/ttyUSB0 
[1659538409.675036] info     | TermiosAgentLinux.cpp | init                     | running...             | fd: 3
[1659538409.675183] info     | Root.cpp           | set_verbose_level        | logger setup           | verbose_level: 4
[1659538417.480562] info     | Root.cpp           | create_client            | create                 | client_key: 0x6EEC6270, session_id: 0x81
[1659538417.480852] info     | SessionManager.hpp | establish_session        | session established    | client_key: 0x6EEC6270, address: 0
[1659538418.720121] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538419.969156] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538421.218229] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538422.467203] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538423.716348] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538424.965481] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538426.214506] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538427.463394] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0
[1659538428.712708] info     | SessionManager.hpp | establish_session        | session re-established | client_key: 0x6EEC6270, address: 0

```

原因未知，但是可以估计是`int clock_gettime(clockid_t unused, struct timespec *tp)`的原因

修改：

```c
int clock_gettime(clockid_t unused, struct timespec *tp)
{
    (void)unused;
    static uint32_t rollover = 0;
    static uint32_t last_measure = 0;

    uint32_t m = rt_tick_get() * 1000 / RT_TICK_PER_SECOND * 1000;

    rollover += (m < last_measure) ? 1 : 0;

    rt_uint64_t real_us = (rt_uint64_t) (m + rollover * micro_rollover_useconds);

    tp->tv_sec = real_us / 1000000;
    
    last_measure = m;
    last_measure = m;

    return 0;
}
```

**能通信成功，但是仅是权宜之计**

通信测试

单片机端:

```bash
msh >microros_sub_int32
[E/micro_ros_serial] succeed to open device uart1
[micro_ros] node created
[micro_ros] executor created
[micro_ros] New thread mr_subint32
msh >[micro_ros] received data 1
[micro_ros] received data 1
[micro_ros] received data 1
[micro_ros] received data 1
[micro_ros] received data 1
[micro_ros] received data 1
[micro_ros] received data 1
```

PC 端：

```
# 参看话题 & 类型
aijun@haijun-Lenovo:/dev$ ros2 topic list -t
/micro_ros_rtt_subscriber [std_msgs/msg/Int32]
/parameter_events [rcl_interfaces/msg/ParameterEvent]
/rosout [rcl_interfaces/msg/Log]


# 发送话题
haijun@haijun-Lenovo:/dev$ ros2 topic pub /micro_ros_rtt_subscriber std_msgs/msg/Int32 "{data: 1}"
publisher: beginning loop
publishing #1: std_msgs.msg.Int32(data=1)

publishing #2: std_msgs.msg.Int32(data=1)

publishing #3: std_msgs.msg.Int32(data=1)

publishing #4: std_msgs.msg.Int32(data=1)

publishing #5: std_msgs.msg.Int32(data=1)

publishing #6: std_msgs.msg.Int32(data=1)

publishing #7: std_msgs.msg.Int32(data=1)
```

## 1. 6 待解决问题

* int clock_gettime(clockid_t unused, struct timespec *tp)
