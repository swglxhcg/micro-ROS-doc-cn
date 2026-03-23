---
title: FreeRTOS 上的第一个 micro-ROS 应用
permalink: /docs/tutorials/core/first_application_rtos/freertos/
redirect_from:
  - /docs/tutorials/advanced/freertos/freertos_getting_started/
---

<img src="https://img.shields.io/badge/Tested_on-Humble-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Rolling-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Iron-green" style="display:inline"/>

在本教程中，您将通过测试 Ping Pong 应用程序来学习在 FreeRTOS 上使用 micro-ROS。
{% include first_application_common/target_hardware.md %}
* [USB转串口线 母头](https://www.olimex.com/Products/Components/Cables/USB-Serial-Cable/USB-SERIAL-F/)

本教程可以相对容易地适配其他目标硬件。[Sameer Tuteja](https://sam-tj.github.io/) 为使用 ESP32 WROOM32 开发板写了一篇很好的博客文章，见 [https://link.medium.com/JFof42RUwib](https://link.medium.com/JFof42RUwib)。

{% include first_application_common/build_system.md %}

```bash
# 创建步骤
ros2 run micro_ros_setup create_firmware_ws.sh freertos olimex-stm32-e407
```

执行此命令后，您的工作空间中必须存在一个名为 `firmware` 的文件夹。

此步骤负责（除其他事项外）下载针对您要处理的特定平台的 micro-ROS 应用程序集。
对于 FreeRTOS，这些应用程序位于 `firmware/freertos_apps/apps`。
每个应用程序由一个包含以下文件的文件夹表示：

* `app.c`：此文件包含应用程序的逻辑。
* `app-colcon.meta`：此文件包含特定于 micro-ROS 应用程序的 colcon 配置。通过此文件配置 RMW 的详细信息可以在[此处](/docs/tutorials/advanced/microxrcedds_rmw_configuration/)找到。

为了使用户能够创建自定义应用程序，需要在此位置注册一个名为 `<my_app>` 的文件夹，其中包含刚才描述的两个文件。

{% include first_application_common/config.md %}

在本教程中，我们将使用串行传输（标记为 `serial`），并重点介绍位于 `firmware/freertos_apps/apps/ping_pong` 的开箱即用的 `ping_pong` 应用程序。要使用所选传输执行此应用程序，请通过如下指定 `[APP]` 和 `[OPTIONS]` 参数来运行上述配置命令：

```bash
# 使用 ping_pong 应用程序和串行传输进行配置步骤
ros2 run micro_ros_setup configure_firmware.sh ping_pong --transport serial
```

您可以在[此处](https://github.com/micro-ROS/freertos_apps/tree/humble/apps/ping_pong)查看 `ping_pong` 应用程序的完整内容。

{% include first_application_common/pingpong_logic.md %}

FreeRTOS 应用程序特定文件的内容可以在以下位置找到：
[app.c](https://github.com/micro-ROS/freertos_apps/blob/humble/apps/ping_pong/app.c) 和
[app-colcon.meta](https://github.com/micro-ROS/freertos_apps/blob/humble/apps/ping_pong/app-colcon.meta)。
仔细查看这些文件可以说明如何在此 RTOS 中创建 micro-ROS 应用程序。

{% include first_application_common/build_and_flash.md %}

{% include first_application_common/agent_creation.md %}

然后，根据所选的传输和 RTOS，开发板与代理的连接方式可能有所不同。
在本教程中，我们使用 Olimex STM32-E407 串行连接，Olimex 开发板通过 usb 转串口线连接到计算机。

<img width="400" style="padding-right: 25px;" src="../imgs/5.jpg">

***提示：** 颜色代码适用于[此电缆](https://www.olimex.com/Products/Components/Cables/USB-Serial-Cable/USB-SERIAL-F/)。
请确保将 Olimex Rx 与电缆 Tx 匹配，反之亦然。记得接地！*

{% include first_application_common/run_app.md %}

{% include first_application_common/test_app_rtos.md %}

这完成了 FreeRTOS 上的第一个 micro-ROS 应用程序教程。您想[返回](../)并尝试不同的 RTOS，即 NuttX 或 Zephyr 吗？
