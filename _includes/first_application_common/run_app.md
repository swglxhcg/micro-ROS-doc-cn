## 运行 micro-ROS 应用程序

此时，客户端和代理都已正确安装。

要使 micro-ROS 访问 ROS 2 数据空间，只需运行代理：

```bash
# 运行 micro-ROS 代理
ros2 run micro_ros_agent micro_ros_agent serial --dev [device]
```

***提示：**您可以使用此命令查找串行设备名称：`ls /dev/serial/by-id/*`*
