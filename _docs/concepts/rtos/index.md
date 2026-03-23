---
title: 为什么需要实时操作系统？
permalink: /docs/concepts/rtos/
---

实时操作系统 (RTOS) 的使用是当今嵌入式系统中的通用实践。这些系统通常由资源受限的微控制器组成，该微控制器执行需要与外部组件交互的应用程序。在许多情况下，此应用程序包含需要严格的时间截止期限或确定性响应的关键时间任务。

当今也使用裸金属应用程序，但需要非常低级的编程技能，并且缺乏 RTOS 提供的硬件抽象层。另一方面，RTOS 通常使用硬件抽象层 (HAL)，以便轻松使用硬件资源，例如定时器和通信总线，减轻开发工作并允许代码重用。此外，它们提供线程和任务实体，结合调度器的使用，提供在应用程序中实现确定性所需的工具。调度包含多种算法，用户可以选择最适合其应用程序的算法。RTOS 通常提供的另一个特性是堆栈管理，帮助正确使用 MCU 资源，这在嵌入式系统中非常宝贵。

## micro-ROS 中的 RTOS

由于上述好处，micro-ROS 将 RTOS 集成到其软件堆栈中。这增强了 micro-ROS 的能力，并允许重用 RTOS 提供的所有工具和功能。由于 micro-ROS 软件堆栈是模块化的，因此软件实体的交换在所有层面（包括 RTOS 层）都是预期和希望的。

与计算机可用的操作系统 (OS) 一样，RTOS 对标准接口也有不同的支持。这在名为 [POSIX](https://pubs.opengroup.org/onlinepubs/9699919799/) 的标准系列中建立。由于我们的目标是在 Linux（一个大部分符合 POSIX 的操作系统）中原生编码的 ROS 2 代码的移植或重用，使用符合这些标准的 RTOS 是有益的，因为代码的移植工作最少。NuttX 和 Zephyr 在很大程度上符合 POSIX 标准，使移植工作最少，而 FreeRTOS 提供了一个插件 *FreeRTOS+POSIX*，借助它现有的符合 POSIX 的应用程序可以轻松移植到 FreeRTOS 生态系统，从而利用其所有功能。

请注意，micro-ROS 堆栈中的多个抽象层都会调用 RTOS 函数。使用 RTOS 原语的主要层是中间件。实际上，它需要访问传输资源（例如串口、UDP 或 6LoWPAN 通信）和 RTOS 的时间资源才能正常操作。此外，micro-ROS 客户端库（rcl、rclc）也最好能够访问 RTOS 资源，以便处理调度或电源管理等机制。这样，开发者可以在各个层面优化应用程序。

目前，micro-ROS 支持三种 RTOS，它们都带有（基本）POSIX 实现：FreeRTOS、Zephyr 和 NuttX，它们都[集成到 micro-ROS 构建系统](/docs/concepts/build_system/)中。
点击下面的徽标，您将被重定向到概述部分，其中介绍了每个 RTOS 的最相关方面和关键特性。

<table style="border:none;">
 <tr>
  <td style="width:33%; text-align:center; vertical-align:bottom; font-weight:bold;"><a href="/docs/overview/rtos/#freertos"><img style="margin-left:auto; margin-right:auto; padding-bottom:5px;" width="263" height="100" src="https://upload.wikimedia.org/wikipedia/commons/4/4e/Logo_freeRTOS.png"><br/>FreeRTOS</a></td>
  <td style="width:33%; text-align:center; vertical-align:bottom; font-weight:bold;"><a href="/docs/overview/rtos/#zephyr"><img style="margin-left:auto; margin-right:auto; padding-bottom:5px;" width="220" height="114" src="/img/posts/logo-zephyr.jpg"><br/>Zephyr</a></td>
  <td style="width:33%; text-align:center; vertical-align:bottom; font-weight:bold;"><a href="/docs/overview/rtos/#nuttx"><img style="margin-left:auto; margin-right:auto; padding-bottom:5px;" width="125" height="125" src="https://upload.wikimedia.org/wikipedia/commons/b/b0/NuttX_logo.png"><br/>NuttX</a></td>
 </tr>
</table>

这些 RTOS 之间的详细技术比较可以在[这里](/docs/concepts/rtos/comparison/)找到。

{% include logos_disclaimer.md %}
