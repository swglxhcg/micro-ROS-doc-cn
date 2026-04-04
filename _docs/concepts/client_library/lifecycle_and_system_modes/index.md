---
title: 生命周期和系统模式
permalink: /docs/concepts/client_library/lifecycle_and_system_modes/
redirect_from:
  - /system_modes/
  - /docs/concepts/client_library/system_modes/
---


目录
- [简介和目标](#introduction-and-goal)
- [需求](#requirements)
- [背景：ROS 2 生命周期](#background-ros-2-lifecycle)
- [主要特性](#main-features)
  - [基础生命周期](#basic-lifecycle)
  - [扩展生命周期](#extended-lifecycle)
  - [系统层次结构和模式](#system-hierarchy-and-modes)
  - [模式推理](#mode-inference)
  - [模式管理器](#mode-manager)
  - [错误处理和规则](#error-handling-and-rules)
- [致谢](#acknowledgments)

## 简介和目标

现代机器人软件架构通常采用分层方法。包含 SLAM、基于视觉的目标识别、运动规划等核心算法的层通常被称为*技能层*或*功能层*。为了执行复杂任务，这些技能由一个或多个上层进行协调，这些上层被称为*执行层和规划层*。其他常见名称包括*任务和任务层*或* deliberation 层*。在本文中，我们使用后者。

我们观察到在 deliberation 层需要处理三个不同但密切相关的方面：

1. **任务处理**：对实际任务的协调，即*直接*、*无错误*的流程。
2. **偶发情况处理**：处理特定任务的偶发情况，例如可预见的重试和失败尝试、障碍物、电量不足。
3. **系统错误处理**：处理异常情况，例如传感器/执行器故障。

用于协调技能的机制包括服务调用和动作调用、重新参数化、设置值、激活/停用组件等。我们区分对运行中的技能组件的*面向功能调用*（设置值、动作查询等）和对单个或多个组件的*面向系统调用*（在组件模式之间切换、重启、关闭等）。

![技能层和 deliberation 层之间的交互](interactions_between_skill_and_deliberation_layer.png)

类似地，我们区分来自技能层的*面向功能通知*（以对长时间运行的服务调用的反馈形式、关于环境中相关事件的消息等）和关于组件故障、硬件错误等的*面向系统通知*。

我们的观察是，任务处理、偶发情况处理和系统错误处理的交织通常会导致 deliberation 层的控制流高度复杂。然而，我们假设通过为面向系统的调用和通知引入适当的抽象，可以降低这种复杂性。

因此，我们在这项工作中的**目标**是提供合适的抽象和框架功能，用于（1）系统运行时配置和（2）系统错误和偶发情况诊断，以减少应用开发者在设计和实现任务、偶发情况和错误处理方面的工作量。

此目标在下图所示的示例架构中进行了说明，该架构基于模型文件进行描述和管理：

![高层架构](mode-management.png)

该方法的主要特性（在本文档的其余部分有详细说明）如下：

1. _扩展生命周期_：用于指定组件运行时状态的可扩展概念，即 ROS 2 生命周期节点。
2. _系统层次结构和模式_：用于根据系统层次结构和*系统模式*（即不同的（子）系统配置）指定 ROS 系统的建模方法。
3. _模式推理_：用于根据可观察的系统信息（即组件的状态、模式和参数）推导整个系统状态和模式的模块。
4. _模式管理器_：用于管理和更改系统运行时配置的模块。
5. _错误处理_：用于指定错误处理和恢复机制的轻量级概念。

## 需求

需求列表保存在 micro-ROS 系统模式仓库的 doc 文件夹中，网址为：
https://github.com/micro-ROS/system_modes/blob/master/system_modes/doc/requirements.md

## 背景：ROS 2 生命周期

我们的方法基于 ROS 2 生命周期。ROS 2 生命周期的首要目标是允许更好地控制 ROS 系统的状态。它允许在运行时一致地初始化、重新启动和/或更换系统部件。它为托管的 ROS 2 节点提供了默认生命周期以及用于管理生命周期节点的匹配工具集。

概念描述可访问：
[http://design.ros2.org/articles/node_lifecycle.html](http://design.ros2.org/articles/node_lifecycle.html)
生命周期节点的实现描述见：
[https://design.ros2.org/articles/node_lifecycle.html](https://design.ros2.org/articles/node_lifecycle.html).

## 主要特性

### 基础生命周期

ROS 2 生命周期已作为 C 编程语言客户端库*[rclc](https://github.com/ros2/rclc)*的一部分为 micro-ROS 实现，请参阅 [rclc_lifecycle](https://github.com/ros2/rclc/tree/master/rclc_lifecycle) 获取源代码和文档。

rclc_lifecycle 包是一个 ROS 2 包，提供了便捷函数，用于将 ROS 客户端库（rcl）节点与 C 编程语言中的 ROS 2 节点生命周期状态机捆绑在一起，类似于 C++ 的 [rclcpp 生命周期节点](https://github.com/ros2/rclcpp/blob/master/rclcpp_lifecycle/include/rclcpp_lifecycle/lifecycle_node.hpp)。

[rclc_examples](https://github.com/ros2/rclc/blob/master/rclc_examples/) 包中的文件 `lifecycle_node.c` 提供了如何使用 rclc 生命周期节点的示例。

### 扩展生命周期

在 micro-ROS 中，我们通过允许指定模式（即子状态，基于标准 ROS 2 参数机制专门化*活动*状态）来扩展 ROS 2 生命周期。我们基于 rclc_lifecycle 和 rclcpp_lifecycle 为 ROS 2 和 micro-ROS 实现了这一概念。

文档和代码见：
[github.com:system_modes/README.md#lifecycle](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#lifecycle)

### 系统层次结构和模式

我们提供了一种建模概念，用于递归地从节点指定系统的层次组合，并使用扩展生命周期（类似于节点）指定系统和（子）系统的状态和模式。此系统模式和层次结构（SMH）模型还包括应用程序特定的模式和状态沿系统层次结构向下到节点的映射。

此模型的描述见：
[github.com:system_modes/README.md#system-modes](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#system-modes)
简单示例见：
[github.com:system_modes_examples/README.md#example-mode-file](https://github.com/micro-ROS/system_modes_examples/README.md#example-mode-file)

### 模式推理

模式推理根据其组件（即 ROS 2 生命周期节点）的生命周期状态、模式和参数配置来推断整个系统状态（和模式）。它解析 SMH 模型并订阅生命周期/模式更改请求、生命周期/模式更改和参数事件。

根据生命周期更改事件，它了解所有节点的*实际*生命周期状态。根据参数更改事件，它了解所有节点的*实际*参数值，这允许根据 SMH 模型推断所有节点的*模式*。
根据 SMH 模型以及所有节点推断的状态和模式，可以沿系统层次结构自下而上地*推断*所有（子）系统 的状态和模式。
这可以与最新*请求*的状态和模式进行比较，以检测偏差。

文档和代码见：
[github.com:system_modes/README.md#mode-inference](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode-inference)
模式推理最好在模式监视器中观察，这是一个基于控制台的调试工具，见：
[github.com:system_modes/README.md#mode-monitor](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode-monitor)

### 模式管理器

基于*模式推理*机制，模式管理器提供额外的服务和主题，以根据 SMH 模型中的规范*管理和调整*系统状态和模式。

文档和代码见：
[github.com:system_modes/README.md#mode-manager](https://github.com/micro-ROS/system_modes/blob/master/system_modes/README.md#mode-manager)
简单示例见：
[github.com:system_modes_examples/README.md#setup](https://github.com/micro-ROS/system_modes_examples/README.md#setup)

### 错误处理和规则

如果系统或其任何部件的*实际*状态/模式与*目标*状态/模式存在偏差，我们定义了试图将系统恢复到有效的*目标*状态/模式的规则，例如降级模式。规则以自下而上的方式工作，即从纠正节点开始，然后是子系统，最后是系统。规则基本上按以下方式定义：

```pseudo
if:
 system.target == {target state/mode} && system.actual != {target state/mode} && part.actual == {specific state/mode}
then:
 system.target := {specific state/mode}
```

如果*实际*状态/模式与*目标*状态/模式存在偏差，但对于这种情况没有精确规则，自下而上的规则将只是尝试将系统/部件返回到其*目标*状态/模式。

*注意：*此特性适合根据所示语法定义简单、明确的规则。对于更复杂的编排，已验证系统模式与本体推理（*元控制*）的集成，并在 [MROS 项目](https://robmosys.eu/mros/)中成功展示，例如在[移动机器人导航子系统](https://github.com/MROS-RobMoSys-ITP/Pilot-URJC)中。

## 致谢

此活动获得了欧洲研究理事会（ERC）在欧盟 Horizon 2020 研究和创新计划下的资助（资助协议编号 780785）。
