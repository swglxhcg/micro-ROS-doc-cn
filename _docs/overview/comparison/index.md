---
title: 与相关方法的比较
permalink: /docs/overview/comparison/
---


Micro-ROS 将 ROS 2 带到微控制器上。在这里，我们对相关方法进行分析，最终展示一个比较表。

## ROSSerial

ROSSerial 是一种用于包装标准 ROS 序列化消息并通过串口或网络套接字等设备复用多个主题和服务的协议。除了协议定义之外，此套件中还包含三种类型的包：

- 客户端库：客户端库允许用户在各种系统上轻松启动和运行 ROS 节点。这些客户端是通用 ANSI C++ rosserial_client 库的移植版本。

- ROS 端接口：这些包提供了一个节点，用于在主机上桥接从 rosserial 协议到更通用的 ROS 网络的连接。

- 示例和用例。

值得注意的是，此选项无法与 micro-ROS 完全比较，因为这种方法适用于 ROS 1，而不是专注于 ROS 2 的 micro-ROS。

参考：[ROSserial Wiki](http://wiki.ros.org/rosserial)

## RIOT-ROS2

RIOT-ROS2 是主 ROS 2 堆栈的修改版，得益于 RIOT 操作系统，使其能够在微控制器上运行。

ROS 2 由多个层组成。其中一些已被修改以能够在微控制器上运行，以下是 RIOT-ROS2 项目可用的层列表：
- ROS 客户端库绑定：RCLC
- ROS 客户端库：RCL
- ROS 中间件：rmw_ndn
- ROS IDL 生成器：generator_c
- ROS IDL 类型支持：CBOR
- ROS IDL 接口：
    - common_interfaces
    - rcl_interfaces

最终数据显示，开发似乎已被冻结。这一考虑是因为最后一次提交可以追溯到 [2018 年 7 月](https://github.com/astralien3000/riot-ros2/commits/master)。

参考：[RIOT-ROS2](https://github.com/astralien3000/riot-ros2/wiki)

## 比较表

|  | rosserial | RIOT-ROS2 | micro-ROS |
|-------|-----------|-----------|-----------|
| 操作系统 | 裸金属 | RIOT | NuttX、FreeRTOS 和 Zephyr |
| 通信架构 | 桥接 | 不适用 | 桥接 |
| 消息格式 | ROS1 | 不适用 | CDR（来自 DDS） |
| 通信链路 | 串口 | 串口 | 串口、SPI、IP (UDP)、6LoWPAN、... |
| 通信协议 | 自定义 | NDN | XRCE-DDS（或任何 rmw 实现） |
| 代码库 | 独立实现 | 标准 ROS 2 堆栈直到 RCL | 标准 ROS 2 堆栈直到 RCL（即将推出 RCLCPP） |
| 节点 API | 自定义 rosserial API | RCL、RCLC | RCL（即将推出 RCLCPP） |
| 回调执行 | 按消息顺序顺序执行 | 不适用 | 选择 ROS 2 执行器或 MCU 优化执行器 |
| 定时器 | 未包含 | 未包含 | 标准 ROS 2 定时器 |
| 与主机时间同步 | 自定义 | 不适用 | NTP/PTP |
| 生命周期 | 不支持 | 部分支持 | 部分支持，即将完全支持 |
