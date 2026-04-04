---
title: 与 FIWARE 的互操作性
permalink: /docs/concepts/fiware_interoperability/
---

## 动机
在 micro-ROS 项目的目标中，关键问题之一是提供与其他杰出且广泛使用的平台的互操作性。
选择的组件之一是 FIWARE 上下文代理（FIWARE Context Broker），这是一个上下文数据管理标准，已被多个欧盟推动的举措所采用，以促进针对不同领域的智能解决方案的开发。

本节说明如何通过将 FIWARE 与 ROS 2 集成来实现 micro-ROS 与该平台之间的互操作性。
借助这种互操作性，FIWARE 的上下文代理成为 micro-ROS 与集成到 FIWARE 生态系统中的任何其他系统共享上下文信息的首选平台。

## 互操作性：不同可能解决方案的优缺点

本小节将解释 micro-ROS 与 FIWARE 上下文代理之间互操作性的所有设计替代方案。
从现在开始，为 micro-ROS 与 FIWARE 之间的通信开发的解决方案将被称为 **FIROS2 集成服务**。

FIROS2 需要转换库将 ROS 2 消息转换为 FIWARE NGSIv2 消息，反之亦然。
每个消息需要一个转换库。

![image](http://www.plantuml.com/plantuml/svg/ZP712i8m38RlUOempuKvfrv49gYmap05BmCfhfs5hOMslhzjLuQYu1e8_E5Fyf4Mnb9jdtq77UCMhK8jseV5HcXsjq99uA9ZcA1xjQnEvmnxPWnjMIrzBK5giDpVvlXXF9RNNNNuRSqGf6f6guymr-sERHTDfU5AzzGJ39Rt2GkShJddQJeHBfyEj_o6YtQ75pRyWrkDS03XC8Hi1sW8ESeio1mtX0nT47AK3gDWil7_yW80)

在这些转换库的实现中，需要能够序列化/反序列化 ROS 2 消息。
此外，还将使用 NGSIv2 序列化/反序列化机制。

FIROS2 包提供了标准的 NGSIv2 序列化/反序列化机制，但由于 ROS 2 序列化与其消息类型的依赖关系更加复杂，因此更加复杂。
因此，FIROS2 集成服务需要提供一个简单的面向用户的解决方案，以自动为 ROS 2 类型生成转换库。

为此，可以采用两种不同的方法：
* 实现一个定制的桥接通信工具，用于将 FIWARE 的消息转换为 micro-ROS（即 ROS 2）消息类型，反之亦然。
* 依赖于使用通用类型语言表示的集成平台，并定义从/到通用类型到每个中间件特定类型的转换库。

第一种方法可能产生更轻量的工具，但它有几个缺陷，例如维护更困难，以及无法与除 ROS2 或 micro-ROS 之外的任何其他中间件通信。
另一方面，使用集成服务平台，例如 [SOSS](https://github.com/eProsima/Integration-Service)，如果其系统句柄实现可用，则可以自动实现与广泛（且不断增长）的中间件集通信的可能性。

## SOSS：系统合成器

**SOSS** 解决的任务是为使用不同语言的通信软件平台提供通用接口。
它由一个**核心**库组成，该库定义了一组抽象接口并提供一些实用类来形成基于插件的框架。

这个可插拔接口允许用户利用特定中间件的任何受支持插件或系统句柄，例如 DDS、ROS2、FIWARE 或 ROS，以实现所需的集成。

SOSS 可以作为中间消息传递工具，通过使用通用语言，集中和协调在不同通信中间件下运行的多个应用程序的集成。
SOSS 实例通过 **YAML** 文件进行配置和启动，允许用户提供不同主题和服务之间的映射，以便两个或更多应用程序可以交换信息。

用户还可以为新中间件开发自己的系统句柄，自动授予与所有其他受支持中间件的通信能力。

通常，类型使用通用语言表示定义，SOSS 使用该表示创建所交换信息的共享表示，以便在需要时可以处理、转换和重新映射到每个中间件的类型实现。
这种通用表示通过 IDL 定义面向用户提供，使用 [eProsima 的 XTypes-DDS](https://github.com/eProsima/xtypes) 实现，在运行时将其解析并转换为动态类型表示。
