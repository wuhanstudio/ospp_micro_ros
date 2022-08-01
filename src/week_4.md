# week4

> 2022/0725 - 2022/07/25 - 2022/07/31

## 1.遗留的问题

单片机端和PC端的create_topic没有成功，估计是/ libc (POSIX) or clock_gettime的问题

## 2. 任务

- [ ] 由于将rtthread的版本降低到4.0.1尝试运行micro -ros软件包；
- [ ] 编译基于cortex-m7的micro -ros的静态编译库，使micro -ros软件包能在art-pi上运行；

## 3. 工作情况

### 3.1 降低rtthread的版本

rtthread4.0.1没有art-pi的bsp，考虑到以后会使用rtthread的最新版本，因此配置art-pi的意义不大，终止该任务；

### 3.2 静态编译micro -ros

参考网址：

[Creating custom static micro-ROS library](https://micro.ros.org/docs/tutorials/advanced/create_custom_static_library/)

[docker](https://github.com/micro-ROS/docker)

由于VPN的原因，导致download的仓库不全，无法进行正常的编译；

## 4. 周会讨论问题

* VPN
* 如何生成静态库；

## 5. 下周任务

1. 解决Ubuntu端的vpn问题；
2. 继续完成为art-pi生成静态库的任务，期望下周能在art-pi运行micro -ros；



