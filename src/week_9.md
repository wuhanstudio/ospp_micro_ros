# week9

> 2022/08/29 - 2022/09/04

## 上周任务

1. 在`micro_ros_setup`中添加rtthread的构建脚本；

## 完成情况

* 初步完成：通过`micro_ros_setup`完成代码下载、编译（microROS & rtthread）

代码：

micro_ros_setup ，新增config/rtthread

```bash
git clone -b rtthread https://github.com/navy-to-haijun/micro_ros_setup.git
```

micro_ros_rtthread： 提供demos&transport

```bash
git clone https://github.com/navy-to-haijun/micro_ros_rtthread.git
```



## 周会拟讨论问题

* 当前构建方式(先用clocon编译microROS为静态文件，在使用scons链接静态文件和rtthread一起编译出可执行文件)是否合理；
* 当前方案的优化指出；
* 为了工程完整必会提供一个demo和transport的文档，这个文档如何处理

## 日志

micro_ros_setup：为不同的嵌入式平台提供代码下载、编译、运行的脚本；

1. micro_ros_setup/config`文件夹下添加rtthread脚本：

```bash
tree
.
├── dev_ros2_packages.txt			# 官方提供：ament相关包的索引
├── dev_uros_packages.repos			# 官方提供：
├── generic
│   ├── build.sh					# 提供编译脚本（microROS & rtthread）
│   ├── client-colcon.meta			# 官方提供：基本宏定义 --> microROS 
│   ├── client_uros_packages.repos  # 官方提供:microros相关包
│   ├── configure.sh				# 提供配置脚本：配置demo，使用的通信方式
│   ├── create.sh					# 提供代码的下载：rtthread demos transport
│   └── supported_platforms			# 手动写入支出的板卡
└── list_apps.sh					# 列出所有的demo(辅助脚本)

1 directory, 9 files
```

2. 自己提供一个文件夹用于存放demos&transport文件

文件名：micro_ros_rtthread（暂时放到自己的仓库下）

```bash
.
├── custom_toolchain.cmake			# 为microROS提供编译预定义
├── examples						# example
│   ├── micro_ros_pub_int32.c
│   └── micro_ros_sub_int32.c
├── Kconfig							# 方便后期menuconfig配置
├── microros_extensions				# 提供通信方式
│   ├── micro_ros_rtt.h
│   └── rtt_serial_transports.c		# serial
├── README.md
├── SConscript						# 编译配置脚本
└── stm32h7xx_hal_msp.c				# 底层驱动配置文件(提供uart1)
```

## 执行步骤

### 1 编译micro_ros_setup

将`config/rtthread`编译(复制)到install中

```bash
clocon build
source install/local_setup.sh
```

### 2 下载源码

主要通过`create_firmware_ws.sh `调用`create.sh`,实现:

* 下载microROS的相关代码

* 下载 rtthread  --> `firmware/sdk-bsp-stm32h750-realthread-artpi`
* 下载micro_ros_rtthread  -->`firmware/gcc-arm-none-eabi-5_4-2016q3`
* 下载交叉编译链  -->`irmware/sdk-bsp-stm32h750-realthread-artpi/projects/art_pi_wifi/micro_ros_rtthread`

```
ros2 run micro_ros_setup create_firmware_ws.sh  rtthread art-pi
```

create.sh核心代码

```shell
 echo "dowmload code "
        git clone https://github.com/RT-Thread-Studio/sdk-bsp-stm32h750-realthread-artpi.git
        git clone https://github.com/navy-to-haijun/micro_ros_rtthread.git

    echo "dowmload  gcc-arm-none-eabi-5_4"
        wget -c https://armkeil.blob.core.windows.net/developer//sitecore/shell/-/media/Files/downloads/gnu-rm/5_4-2016q3/gcc-arm-none-eabi-5_4-2016q3-20160926-linux,-d-,tar.bz2
        tar -xvf gcc-arm-none-eabi-5_4-2016q3-20160926-linux,-d-,tar.bz2
        rm gcc-arm-none-eabi-5_4-2016q3-20160926-linux,-d-,tar.bz2
    fi

    if [ -e $FW_TARGETDIR/sdk-bsp-stm32h750-realthread-artpi/projects/art_pi_wifi ]; then
        mv  micro_ros_rtthread/ ./sdk-bsp-stm32h750-realthread-artpi/projects/art_pi_wifi
        # soft link
        pushd $FW_TARGETDIR/sdk-bsp-stm32h750-realthread-artpi/projects/art_pi_wifi>/dev/null
           ln -s ../../libraries/ libraries
           ln -s ../../rt-thread/ rt-thread 
        popd >/dev/null
    fi
```



### 3. 配置rtthread工程

主要通过`configure_firmware.sh `调用`configure.sh`,实现:

* 为*cconfig.h*提供以下宏定义：
  * 开启UART1 or udp
  * 选中的demo的相关宏定义
* 为Kconfig添加一个source 使其能找到`micro_ros_rtthread`

```
ros2 run micro_ros_setup configure_firmware.sh micro_ros_pub_int32.c -t serial
```

configure.sh核心代码

```shell
if [ "$UROS_TRANSPORT" == "serial" ]; then
		echo "Using serial device."
		echo "add macro definition for  cconfig.h."
		sed -i '$i #define BSP_USING_UART1 ' 			$RTCONIF
		sed -i '$i #define USING_MICROROS ' 			$RTCONIF
		sed -i '$i #define MICROROS_SERIAL ' 			$RTCONIF
		sed -i '$i #define MICROROS_DEVIVE "uart1" ' 	$RTCONIF
		echo "add macro: BSP_USING_UART1; USING_MICROROS; MICROROS_SERIAL; MICROROS_DEVIVE "
		# modify stm32h7xx_hal_msp.c
		cp micro_ros_rtthread/stm32h7xx_hal_msp.c board/CubeMX_Config/Core/Src/stm32h7xx_hal_msp.c
fi

if [ "$CONFIG_NAME" == "micro_ros_pub_int32.c" ]; then
		sed -i '$i #define MICROS_EXAMPLE_PUB_INT32 ' 	$RTCONIF
		echo "add macro: MICROS_EXAMPLE_PUB_INT32 "
	elif [ "$CONFIG_NAME" == "micro_ros_sub_int32.c" ]; then
		sed -i '$i #define MICROS_EXAMPLE_SUB_INT32 ' 	$RTCONIF
		echo "add macro:define MICROS_EXAMPLE_SUB_INT32 "
	else
		help
	fi

	sed -i '$a source "micro_ros_rtthread/Kconfig" ' 	Kconfig
	echo 'add source :  source "micro_ros_rtthread/Kconfig" --> Kconfig'
```

### 4. 编译

通过`create_firmware_ws.sh`调用`create.sh`

主要通过`configure_firmware.sh `调用`configure.sh`,实现:

* 配置交叉编译链 ----> custom_toolchain.cmake & rtconfig.py
* 编译microROS：最终会生成一个静态文件
* 编译rtthread

```
ros2 run micro_ros_setup build_firmware.sh 
```

​	build.sh核心代码

```shell
colcon build \
		--merge-install \
		--packages-ignore-regex=.*_cpp \
		--metas $COLCON_META \
		--cmake-args \
		"--no-warn-unused-cli" \
		-DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=OFF \
		-DTHIRDPARTY=ON \
		-DBUILD_SHARED_LIBS=OFF \
		-DBUILD_TESTING=OFF \
		-DCMAKE_BUILD_TYPE=Release \
		-DCMAKE_TOOLCHAIN_FILE=$TOOLCHAIN \
		-DCMAKE_VERBOSE_MAKEFILE=ON; \

pushd $RTTHREAD_DIR >/dev/null

	scons --target=vsc
	scons

popd >/dev/null
```

### 5. 下载

由于art-pi下载需要调用外部算法，不知道如何写成命令，所有这一步的脚本没有写

## 问题

micro_ros_rtthread/build/include/rmw/types.h 在编译时会报错误, 错误定位：

```c
#ifndef _WIN32
# define RMW_DECLARE_DEPRECATED(name, msg) name __attribute__((deprecated(msg)))
#else
# define RMW_DECLARE_DEPRECATED(name, msg) name __pragma(deprecated(name))
#endif
```

为什么_WIN32宏定义被启用， 于是编译之前，修改：

````c
#ifndef _WIN32
# define RMW_DECLARE_DEPRECATED(name, msg) name
#else
# define RMW_DECLARE_DEPRECATED(name, msg) name __pragma(deprecated(name))
#endif
````

