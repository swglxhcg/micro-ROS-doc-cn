---
title: 诊断
permalink: /docs/concepts/client_library/diagnostics/
---

虽然诊断不是 ROS 2 客户端库包（即 rclcpp、rclpy）的一部分，但它绝对可以被归类为扩展客户端库，因为它提供了非常通用且与应用无关的功能。

这就是为什么 micro-ROS 客户端库带有基本诊断功能。这些与 ROS 2 诊断兼容，仅包含三个功能：

* 诊断消息类型（针对 Micro-XRCE-DDS 进行了优化 - 无动态数组）
* rclc 的更新机制
* 微控制器的基本诊断监视器

micro-ROS 诊断包不提供任何聚合器，因为我们假设此类聚合发生在运行标准 ROS 2 的微处理器上。因此，我们假设以下典型架构：

<img src="diagnostics_architecture.png" style="display:block; width:60%; margin-left:auto; margin-right:auto;"/>

为了让标准 ROS 2 诊断聚合器聚合 micro-ROS 诊断消息类型，ROS 2 代理需要将 micro-ROS 诊断消息转换为标准 ROS 2 诊断消息（*待定*）。

更多信息，请参阅 [https://github.com/micro-ROS/micro_ros_diagnostics/](https://github.com/micro-ROS/micro_ros_diagnostics/)。有关 ROS 2 诊断的更多信息，请参阅 [ROS 2 diagnostics](https://github.com/ros/diagnostics/tree/ros2-devel) 和 [ROS 2 diagnostic_msgs](https://github.com/ros2/common_interfaces/tree/master/diagnostic_msgs)。
