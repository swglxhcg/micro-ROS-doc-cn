---
title: Micro XRCE-DDS
permalink: /docs/concepts/middleware/Micro_XRCE-DDS/
redirect_from: /docs/concepts/middleware/
---

本文档专门介绍 [**Micro XRCE-DDS**](https://micro-xrce-dds.docs.eprosima.com/en/latest/index.html) 最显著的特性，它是 micro-ROS rmw 层的默认中间件实现。

**eProsima Micro XRCE-DDS** 是一个开源有线协议，实现了 OMG DDS for e**X**tremely **R**esource **C**onstrained **E**nvironment 标准（[DDS-XRCE](https://www.omg.org/spec/DDS-XRCE/)）。
DDS-XRCE 协议的目标是为资源受限的设备提供对 DDS Global-Data-Space 的访问。
这是通过 **客户端-服务器** 架构实现的，其中称为 *XRCE Clients* 的低资源设备连接到称为 *XRCE Agent* 的服务器，该服务器代表其客户端在 DDS Global-Data-Space 中行事。

![](uxrce_scope.png)

Micro XRCE-DDS 由两个主要元素组成：

* [Micro XRCE-DDS Agent](https://github.com/eProsima/Micro-XRCE-DDS-Agent)：一个 **C++11 开箱即用应用程序**，实现了 XRCE Agent 功能。
* [Micro XRCE-DDS Client](https://github.com/eProsima/Micro-XRCE-DDS-Client)：一个 **C99 库**，实现了 XRCE Client 端功能。

此外，Micro XRCE-DDS 还使用其他两个组件：

* [Micro CDR](https://github.com/eProsima/Micro-CDR)：客户端库中使用的**反序列化引擎**。
* [Micro XRCE-DDS Gen](https://github.com/eProsima/Micro-XRCE-DDS-Gen)：一个**代码生成工具**，用于从 IDL 源生成 Micro CDR 反序列化函数和客户端应用示例。

## 应用

Micro XRCE-DDS 专注于需要访问发布/订阅架构的微控制器应用。
这类应用的例子包括传感器网络、物联网或机器人技术中的应用。
一些公司如 [Renesas](https://www.sensorsmag.com/iot-wireless/mcus-support-dds-xrce-protocol-for-ros-2) 和 [ROBOTIS](https://xelnetwork.readthedocs.io/en/latest/) 正在使用 Micro XRCE-DDS 作为中间件解决方案。
此外，[micro-ROS](https://microros.github.io) 项目（其目标是将 ROS 2 带到微控制器上）已采用 Micro XRCE-DDS 作为其默认中间件层。

## 主要特性

### 低资源消耗

如上所述，Micro XRCE-DDS 专注于微控制器应用。因此，该中间件的设计和实现已考虑到此类设备的内存约束。
一个证明是 XRCE Client 完全不使用动态内存。
从内存占用的角度来看，该库的[最新版本](https://github.com/eProsima/Micro-XRCE-DDS-Client/releases/latest)对于处理约 512 B 消息大小的完整发布者和订阅者应用，内存消耗小于 **75 KB Flash 内存**和约 **3 KB RAM**。
有关作为消息大小、实体数量和中间件库内部内存管理函数的内存消耗的更详细信息，请参阅 [Micro XRCE-DDS 内存分析](/docs/concepts/middleware/memo_prof/) 部分。
此外，该库具有高度可配置性，这归功于一个 *profile* 概念，允许在配置时选择、添加或删除某些功能。这允许自定义 XRCE Client 库大小（如果不使用某些功能）。
有多个定义可用于在编译时配置和构建客户端库。
这些定义允许根据应用需求创建库版本，并且可以在 `client.config` 文件中进行修改。
要合并所需的配置，每次更改定义时都必须运行 `cmake` 命令。

有关如何通过正确调整 Micro XRCE-DDS 库或其 rmw 实现 [`rmw_microxrcedds`](https://github.com/micro-ROS/rmw-microxrcedds) 中的参数来配置 micro-ROS 的更多信息，请参阅此[教程](/docs/tutorials/advanced/microxrcedds_rmw_configuration/)和 `rmw_microxrcedds` [README](https://github.com/micro-ROS/rmw-microxrcedds#rmw-micro-xrce-dds-implementation)。

### 多传输支持

作为上一节讨论的 profiles 的一部分，用户可以在多种传输层之间进行选择，以实现 Client 与 Agent 的通信。
实际上，与其他物联网中间件（如 MQTT 和 CoaP，仅在特定传输层上工作）形成对比的是，XRCE 原生支持多种传输协议。
特别是，最新版本的 Micro XRCE-DDS 支持：**UDP**、**TCP** 和自定义 **Serial** 传输协议。

除此之外，Micro XRCE-DDS 为 Agent 和 Client 提供了传输接口，允许以直接的方式实现自定义传输。
这使得将 Micro XRCE-DDS 移植到不同平台以及添加新传输成为任何用户都可以无缝完成的任务。

### 多平台支持

XRCE Client 支持 **FreeRTOS**、**Zephyr** 和 **NuttX** 作为嵌入式 RTOS。此外，它还可以在 **Windows** 和 **Linux** 上运行。
另一方面，XRCE Agent 支持 **Windows** 和 **Linux**。

### QoS 支持

XRCE Client 库允许用户使用两种不同的方法在 XRCE Agent 中创建 DDS 实体：

* 通过 XML（默认选项）
* 通过引用

使用默认选项时，用户可以创建可靠或尽力而为模式的实体，XML 文件由客户端编写和存储。但这些 QoS 配置可能无法满足某些用户的要求。
对于这些情况，Micro XRCE-DDS 允许直接在 Agent 上创建实体，用户可以在其中编写自定义 XML QoS（如 DDS 中一样）。
Agent 上每个可用的实体都将与一个标签相关联，这样 Client 只需引用这些标签就可以创建通信所需的实体。

此外，使用引用还会减少 Client 在 MCU 中的内存消耗。
这是因为引用方法允许避免构建存储 XML 的代码部分。

请注意，micro-ROS 继承了这一机制，因此能够利用与 ROS 2 相同的完整 QoS 集。
有关如何在 micro-ROS 中使用自定义 QoS 的综合说明，请访问教程部分的此[专门页面](/docs/tutorials/advanced/create_dds_entities_by_ref/)。

## 其他链接

* [Read the Docs 上的手册](https://micro-xrce-dds.readthedocs.io/en/latest/)
* [GitHub 上的 Micro XRCE-DDS](https://github.com/eProsima/Micro-XRCE-DDS)
* [GitHub 上的 XRCE Client](https://github.com/eProsima/Micro-XRCE-DDS-Client)
* [GitHub 上的 XRCE Agent](https://github.com/eProsima/Micro-XRCE-DDS-Agent)
* [GitHub 上的 rmw_microxrcedds](https://github.com/micro-ROS/rmw-microxrcedds)
* [Micro XRCE-DDS 内存分析](/docs/concepts/middleware/memo_prof/)
* [中间件优化教程](/docs/tutorials/advanced/microxrcedds_rmw_configuration/)。
* [如何在 micro-ROS 中使用自定义 QoS](/docs/tutorials/advanced/create_dds_entities_by_ref/)
