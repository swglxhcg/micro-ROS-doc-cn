---
title: 概述
permalink: /docs/tutorials/core/overview/
redirect_from:
  - /docs/tutorials/core/
  - /docs/tutorials/
---

本章提供了一系列教程，帮助您学习 micro-ROS 及其支持的各个 RTOS 的相关工具。如果您是 micro-ROS 的新手，我们强烈建议您按以下顺序学习教程：

* [**在 Linux 上构建第一个 micro-ROS 应用**](../first_application_linux/)
    
  本教程将教您如何安装 micro-ROS 框架和工具。然后它将引导您在 Linux 上开发您自己的第一个 micro-ROS 应用程序。（如果您已经了解 ROS 2，您会发现这些工具与标准 ROS 2 很好地集成了。）
    
* [**在 RTOS 上构建第一个 micro-ROS 应用**](../first_application_rtos/)

  在本教程中，您将学习如何为实时操作系统 (RTOS) 构建前一个教程中的应用程序。您将了解如何将应用程序烧录到微控制器板中，以及如何与运行在 Linux 上的 ROS 2 的微处理器进行通信。（本教程涵盖 micro-ROS 支持的所有三个 RTOS，即 NuttX、FreeRTOS 和 Zephyr。由您选择！）

然后，在这一点上，您可以前往下一节 [**使用 rcl 和 rclc 编程**](../../programming_rcl_rclc/)，在那里您将深入了解本教程中的 micro-ROS C API 概念。如果您已经熟悉 ROS 2 C++ API，甚至底层 ROS 客户端支持库 (rcl)，您将很快掌握它。

如果您使用相应的 RTOS 或硬件，在进入 [**使用 rcl 和 rclc 编程**](../../programming_rcl_rclc/) 部分之前，以下基础教程可能会让您感兴趣：

* [**Zephyr 模拟器**](../zephyr_emulator/)

  在本教程中，您将学习通过测试 Ping Pong 应用程序来使用 Zephyr 模拟器的 micro-ROS。
  
* [**Teensy 与 Arduino**](../teensy_with_arduino/)
     
  在本教程中，您将学习如何将 Teensy 与 micro-ROS 和 ROS 2 连接。您还将学习如何在 Linux 系统中安装 micro-ROS 代理，以通过 Arduino IDE 与基于 Teensy 的 Arduino 板进行通信。本教程还将介绍从 Teensy 发布的一个简单发布者主题，并使用 ROS2 接口订阅它。 

 在 [**高级教程**](../../advanced/overview/) 部分，您将找到更多高级教程来加强您的 micro-ROS 知识。
