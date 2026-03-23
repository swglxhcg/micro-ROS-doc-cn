---
title: 概述
permalink: /docs/tutorials/programming_rcl_rclc/overview/
redirect_from:
  - /docs/tutorials/programming_rcl_rclc/
---

在本节中，您将学习 micro-ROS C API 的基础知识：**rclc**。

主要概念（发布者、订阅者、服务、定时器等）与 ROS 2 相同。它们甚至依赖于*相同的*实现，因为 micro-ROS C API 基于 ROS 2 客户端支持库 (rcl)，并通过 [rclc](https://github.com/ros2/rclc/) 包增强了一组便捷函数。也就是说，rclc 没有在 rcl 之上添加新的类型层（如 rclcpp 和 rclpy 所做的那样），而只是提供使用 rcl 类型进行编程的便捷函数。新类型仅引入 rcl 中缺失的概念，例如执行器的概念。

* [**节点**](../node/)
* [**发布者和订阅者**](../pub_sub/)
* [**服务**](../service/)
* [**参数**](../parameters/)
* [**执行器和定时器**](../executor/)
* [**服务质量**](../qos/)
* [**micro-ROS 工具**](../micro-ROS/)
