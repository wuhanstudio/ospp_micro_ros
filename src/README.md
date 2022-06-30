# MicroROS on RT-Thread

OSPP 2022 - MicroROS on RT-Thread

**Student**: Haijun Du

**Mentor**: Han Wu

Robot Operating System (ROS) is a set of a software frameworks for robotic application development that was initially released by Stanford University in 2007. Further improvements on ROS could introduce breaking changes, making ROS1 unstable. As a result, the second generation of ROS was released (ROS2). The target platform of ROS2 is Linux, while micro_ros devote to bringing ROS2 API to MCU.

This project aims to use micro_ros on RT-Thread, which is a Real-time Operating System (RTOS). Some issues remain unsolved on the existing micro_ros software package(https://github.com/wuhanstudio/micro_ros): the UDP communication is unstable, more service examples are to be added, a precompiled static library is required for compilation of the micro_ros package, and not integrated into the micro_ros official build system to provide benchmark results.
