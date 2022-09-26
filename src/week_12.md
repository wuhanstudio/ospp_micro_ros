# week12

> 2022/09/19 - 2022/09/26

## 上周任务

- [x] 完成需要在2022开源之夏中提交的文档；
- [x] 提交PR；
- [ ] 继续修复UDP subscriber & service的问题；

周会拟讨论的问题

* UDP subscriber
* 结尾事项；

## 日志

### PR

PR地址: https://github.com/micro-ROS/micro_ros_setup/pull/581 ， 目前没有回应

### bug

对于service的问题可以确定是`rmw ` 需要使用`rmw_fastrtps`

```
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

对于UDP subscriber， 我使用`sub_twist`的问题和`sub_int32`的问题一样：没有回调

测试环境: docker

