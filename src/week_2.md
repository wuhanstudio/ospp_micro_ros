# Week 2

> 2022/07/11 - 2022/07/17

## 上周任务

* 解决uart1问题（完成）；
* 在单片机端使用` subscribe int32`（未完成）
* 尝试Hardfloat / Softfloat（未完成）

### uart1通信问题

未对uart1底层驱动进行适配，硬件没有问题

## 周会需帮助解决问题

* micro ros 软件包编译报错

arm-none-eabi-gcc 版本：

```bash
arm-none-eabi-gcc -v

Using built-in specs.
COLLECT_GCC=arm-none-eabi-gcc
COLLECT_LTO_WRAPPER=/home/haijun/env_released_1.2.0/gcc-arm-none-eabi-5_4-2016q2/bin/../lib/gcc/arm-none-eabi/5.4.1/lto-wrapper
Target: arm-none-eabi
Configured with: /home/build/work/GCC-5-0-build/src/gcc/configure --target=arm-none-eabi --prefix=/home/build/work/GCC-5-0-build/install-native --libexecdir=/home/build/work/GCC-5-0-build/install-native/lib --infodir=/home/build/work/GCC-5-0-build/install-native/share/doc/gcc-arm-none-eabi/info --mandir=/home/build/work/GCC-5-0-build/install-native/share/doc/gcc-arm-none-eabi/man --htmldir=/home/build/work/GCC-5-0-build/install-native/share/doc/gcc-arm-none-eabi/html --pdfdir=/home/build/work/GCC-5-0-build/install-native/share/doc/gcc-arm-none-eabi/pdf --enable-languages=c,c++ --enable-plugins --disable-decimal-float --disable-libffi --disable-libgomp --disable-libmudflap --disable-libquadmath --disable-libssp --disable-libstdcxx-pch --disable-nls --disable-shared --disable-threads --disable-tls --with-gnu-as --with-gnu-ld --with-newlib --with-headers=yes --with-python-dir=share/gcc-arm-none-eabi --with-sysroot=/home/build/work/GCC-5-0-build/install-native/arm-none-eabi --build=i686-linux-gnu --host=i686-linux-gnu --with-gmp=/home/build/work/GCC-5-0-build/build-native/host-libs/usr --with-mpfr=/home/build/work/GCC-5-0-build/build-native/host-libs/usr --with-mpc=/home/build/work/GCC-5-0-build/build-native/host-libs/usr --with-isl=/home/build/work/GCC-5-0-build/build-native/host-libs/usr --with-cloog=/home/build/work/GCC-5-0-build/build-native/host-libs/usr --with-libelf=/home/build/work/GCC-5-0-build/build-native/host-libs/usr --with-host-libstdcxx='-static-libgcc -Wl,-Bstatic,-lstdc++,-Bdynamic -lm' --with-pkgversion='GNU Tools for ARM Embedded Processors' --with-multilib-list=armv6-m,armv7-m,armv7e-m,armv7-r,armv8-m.base,armv8-m.main
Thread model: single
gcc version 5.4.1 20160609 (release) [ARM/embedded-5-branch revision 237715] (GNU Tools for ARM Embedded Processors)
```

编译报错：

```bash
scons

scons: Reading SConscript files ...
Newlib version:unknown
scons: done reading SConscript files.
scons: Building targets ...
scons: building associated VariantDir targets: build
CC build/packages/micro_ros-galactic/src/rtt_serial_transports.o
packages/micro_ros-galactic/src/rtt_serial_transports.c:47:16: error: conflicting types for '__ctype_ptr__'
 char __EXPORT *__ctype_ptr__ = (char *) _ctype_b + 127;
                ^
In file included from packages/micro_ros-galactic/src/rtt_serial_transports.c:5:0:
/home/haijun/env_released_1.2.0/gcc-arm-none-eabi-5_4-2016q2/arm-none-eabi/include/ctype.h:46:23: note: previous declaration of '__ctype_ptr__' was here
 extern __IMPORT char *__ctype_ptr__;
                       ^
In file included from packages/micro_ros-galactic/src/rtt_serial_transports.c:52:0:
packages/micro_ros-galactic/src/micro_ros_rtt.h:20:44: warning: 'struct timespec' declared inside parameter list
 int clock_gettime(clockid_t unused, struct timespec *tp);
                                            ^
packages/micro_ros-galactic/src/micro_ros_rtt.h:20:44: warning: its scope is only this definition or declaration, which is probably not what you want
packages/micro_ros-galactic/src/rtt_serial_transports.c:71:5: error: conflicting types for 'clock_gettime'
 int clock_gettime(clockid_t unused, struct timespec *tp)
     ^
In file included from packages/micro_ros-galactic/src/rtt_serial_transports.c:52:0:
packages/micro_ros-galactic/src/micro_ros_rtt.h:20:5: note: previous declaration of 'clock_gettime' was here
 int clock_gettime(clockid_t unused, struct timespec *tp);
     ^
scons: *** [build/packages/micro_ros-galactic/src/rtt_serial_transports.o] Error 1
scons: building terminated because of errors.
```

**猜测原因：GCC版本问题**

上次周会下载的arm-none-eabi-gcc 5.4.1因为网络问题，似乎没有下载全，我从该[链接](https://developer.arm.com/downloads/-/gnu-rm)下载 ：

选择： 5-2016-q2-update