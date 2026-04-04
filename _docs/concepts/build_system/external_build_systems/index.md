---
title: 外部构建系统
permalink: /docs/concepts/build_system/external_build_systems/
---

阅读完官方的 [**micro_ros_setup** 工具](/docs/concepts/build_system/)后，本页将介绍其他一些将 micro-ROS 作为模块或组件集成到其他构建系统中的构建方法。

## ESP-IDF 的 micro-ROS 组件

[ESP-IDF 的 micro-ROS 组件](https://github.com/micro-ROS/micro_ros_espidf_component) 允许将 micro-ROS 作为组件集成到 Espressif ESP-IDF 构建系统中。该组件允许用户只需通过克隆或复制文件夹即可在已创建的 ESP-IDF 项目中集成 micro-ROS API 和工具。

micro-ROS 库的配置基于 `colcon.meta` 文件。更多详情请访问 [Git 仓库](https://github.com/micro-ROS/micro_ros_espidf_component)。

## Zephyr 的 micro-ROS 模块

[Zephyr 的 micro-ROS 模块](https://github.com/micro-ROS/micro_ros_zephyr_module) 允许将 micro-ROS 作为模块集成到基于 Zephyr 的项目中。详细地说，它允许用户只需通过克隆或复制文件夹即可在现有 Zephyr 项目中集成 micro-ROS API 和工具。

配置已构建的 micro-ROS 库的过程基于 `colcon.meta`。更多详情请访问 [Git 仓库](https://github.com/micro-ROS/micro_ros_espidf_component)。

## Arduino 的 micro-ROS

[Arduino 的 micro-ROS](https://github.com/micro-ROS/micro_ros_arduino) 支持包是 micro-ROS 的一个特殊端口，作为一组针对特定平台的预编译库提供。这种方法的主要原因是 Arduino 不允许构建像 micro-ROS 这样的复杂库，因此使用这种方法可以为 Arduino 用户提供即用型解决方案。

除了此支持包外，还有[详细说明](https://github.com/micro-ROS/micro_ros_arduino#how-to-build-the-precompiled-library)，供需要调整默认配置的用户重新构建 Arduino 的 micro-ROS 库。

## STM32CubeMX 的 micro-ROS

[STM32CubeMX 的 micro-ROS](https://github.com/micro-ROS/micro_ros_stm32cubemx_utils) 包是一组工具，可实现 micro-ROS 到基于 STMicroelectronics 控制器的项目的无缝配置、设置和集成。因此，通过它 micro-ROS 几乎可以支持 <a href="https://www.st.com/content/st_com/en.html">STMicroelectronics</a> 提供的全系列开发板。

其使用基于 Docker，通过准备好的 [Dockerfile](https://github.com/micro-ROS/docker/blob/humble/micro-ROS-static-library-builder/Dockerfile) 来简化在 ROS 2 环境之外生成 micro-ROS 库的过程。
