---
title: 客户端库简介
permalink: /docs/concepts/client_library/introduction/
redirect_from:
  - /docs/concepts/
  - /docs/concepts/client_library/
---

客户端库为用户代码（即应用级 micro-ROS 节点）提供 micro-ROS API。我们的目标是在针对微控制器优化的实现中提供所有相关的主要 ROS 2 概念。同时，我们努力使 API 与标准 ROS 2 尽可能兼容，以方便代码移植。

为了最大限度地降低长期维护成本，我们使用 ROS 2 堆栈中的现有数据结构和算法，并在主堆栈中尽可能带来必要的更改。这就是为什么 micro-ROS 客户端库由标准的 [ROS 2 客户端支持库 (rcl)](https://github.com/ros2/rcl/) 和新的 [ROS 2 客户端库包 (rclc)](https://github.com/ros2/rclc/) 构建而成。如图所示，rcl + rclc 共同构成了一个功能完整的 C 语言客户端库。

<img src="/img/micro-ROS_architecture.png" style="display:block; width:50%; float:right;"/>

重要的特性和属性：

* 尽可能使用 rcl 数据结构以避免包装器带来的运行时开销。
* 由 rclc 提供的常用任务的便捷函数（例如，创建发布者、终止订阅）。
* 专用的执行器，用于对回调的触发和处理顺序进行细粒度控制。
* 专门的实现用于图形、生命周期节点、诊断等。

请查看子页面（见左侧）以获取更多信息。

<br style="clear:both;" />

对于感兴趣的读者：关于使用 rcl + rclc 组合决策的理由在我们 2019 年的[决策论文 (PDF)](/download/client_library_decision_paper_2019.pdf) 中有说明。
