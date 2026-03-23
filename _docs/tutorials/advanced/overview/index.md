---
title: 概述
permalink: /docs/tutorials/advanced/overview/
redirect_from:
  - /docs/tutorials/advanced/
---

本章为已经具备一定 micro-ROS 知识的用户提供一系列高级教程。与 [**入门教程**](../../core/overview) 相比，这些教程有助于在更深层次上与 micro-ROS 进行交互。建议按特定顺序学习这些教程，因为每个教程涉及 micro-ROS 堆栈和工具链的不同方面。

* [**优化中间件配置**](../microxrcedds_rmw_configuration/)

  在本教程中，我们将引导您配置微控制器与运行在某些基于 Linux 的微处理器上的 micro-ROS 代理之间的中间件，以针对您的特定用例和应用程序进行优化。

* [**如何在 micro-ROS 中包含自定义 ROS 消息**](../create_new_type/)

  本教程解释如何在 micro-ROS 应用程序中创建或包含自定义 ROS 消息类型，特别是如何将其引入[构建系统](https://github.com/micro-ROS/micro_ros_setup)。

* [**如何在 micro-ROS 中使用自定义 QoS**](../create_dds_entities_by_ref/)

  本教程解释使用 ROS 2 (DDS) 实体创建模式 *by references*（由 micro-ROS 默认中间件 Micro XRCE-DDS Client 允许）创建具有完全可配置 QoS 设置的 micro-ROS 实体的步骤。

* [**创建自定义 micro-ROS 传输**](../create_custom_transports/)

  本教程旨在为有兴趣创建自定义 micro-ROS 传输的用户提供逐步指导，而不是使用 micro-ROS 工具集中默认提供的传输。

* [**创建自定义静态 micro-ROS 库**](../create_custom_static_library/)

  本教程旨在为有兴趣将 micro-ROS 编译为独立库以将其集成到自定义开发工具中的用户提供逐步指导。

* [**使用 Shadow-Builder 进行基准测试**](../benchmarking/)

  本教程旨在描述一种称为 *Shadow-Builder* 的特定基准测试工具。更具体地说，它解释了如何从头到尾创建插件以及如何为代码添加检测工具。
