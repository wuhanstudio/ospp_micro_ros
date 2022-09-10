# week10

> 2022/09/05 - 2022/09/10

## 上周任务

- [x] 完成下载脚本编写；
- [x] 将外接串口换成USB虚拟串口；
- [x] 增加demo；
- [ ] 尝试修复UDP bug；

## 完成情况

### 下载脚本编写：

**完成，无问题**

`micro_ros_setup/config/rtthread/generic`文件中增加`flash.sh` 调用 `STM32CubeProgrammer` 完成下载：

核心代码：

```bash
RTTHREAD_DIR=$FW_TARGETDIR/sdk-bsp-stm32h750-realthread-artpi/projects/art_pi_wifi
STLDR_DIR=$FW_TARGETDIR/sdk-bsp-stm32h750-realthread-artpi/debug/stldr/ART-Pi_W25Q64.stldr

STM32_Programmer_CLI -c port=swd -w "$RTTHREAD_DIR/rtthread.bin" 0x90000000 -v -el  $STLDR_DIR
```

### 将外接串口换成USB虚拟串口

**完成，无问题**

增加一个`rtconfig.h`文件，用于提前配置USB 虚拟串口；

### 增加demo

| demo                          | 备注                                       |
| ----------------------------- | ------------------------------------------ |
| micro_ros_pub_int32.c         | uart可用、udp4 可用                        |
| micro_ros_sub_int32.c         | uart可用、udp4 不可用                      |
| micro_ros_pub_sub_int32.c     | uart可用、udp4 不可用                      |
| micro_ros_ping_pong.c         | uart可用、udp4 不可用                      |
| micro_ros_addtwoints_server.c | service 客户端：似乎没有进入回调           |
| micro_ros_addtwoints_client.c | service 客户端：回调接收的数据和实际不符合 |

### UDP

参考：

```
 Micro-XRCE-DDS-Client/src/c/profile/transport/ip/udp/udp_transport_posix_nopoll.c 
```

对读写进行改写，publisher 没有问题， subscriber能正常建立，但是似乎进不去回调。

问题已经在micro_ros_setup 提出  [link](https://github.com/micro-ROS/micro_ros_setup/issues/576)



## 周会拟讨论问题

1. 是否可以提交pr；
2. 解决service demos 的问题；
