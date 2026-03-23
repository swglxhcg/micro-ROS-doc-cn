---
title: Zephyr 上的第一个 micro-ROS 应用
permalink: /docs/tutorials/core/first_application_rtos/zephyr/
redirect_from:
  - /docs/tutorials/advanced/zephyr/zephyr_getting_started/
---

<img src="https://img.shields.io/badge/Tested_on-Humble-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Rolling-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Iron-green" style="display:inline"/>

在本教程中，您将通过测试 Ping Pong 应用程序来学习在 Zephyr 上使用 micro-ROS。
{% include first_application_common/target_hardware.md %}
* [USB转Mini-USB线](https://www.olimex.com/Products/Components/Cables/CABLE-USB-A-MINI-1.8M/)

{% include first_application_common/build_system.md %}

```bash
# 创建步骤
ros2 run micro_ros_setup create_firmware_ws.sh zephyr olimex-stm32-e407
```

{% include first_application_common/zephyr_common.md %}

{% include first_application_common/config.md %}

在本教程中，我们将使用 USB 传输（标记为 `serial-usb`），并重点介绍位于 `firmware/zephyr_apps/apps/ping_pong` 的开箱即用的 `ping_pong` 应用程序。要使用所选传输执行此应用程序，请通过如下指定 `[APP]` 和 `[OPTIONS]` 参数来运行上述配置命令：

```bash
# 使用 ping_pong 应用程序和串口-USB 传输进行配置步骤
ros2 run micro_ros_setup configure_firmware.sh ping_pong --transport serial-usb
```
您可以在[此处](https://github.com/micro-ROS/zephyr_apps/tree/humble/apps/ping_pong)查看 `ping_pong` 应用程序的完整内容。

{% include first_application_common/pingpong_logic.md %}

Zephyr 应用程序特定文件的内容可以在以下位置找到：
[main.c](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/src/main.c)、
[app-colcon.meta](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/app-colcon.meta)、
[CMakeLists.txt](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/CMakeLists.txt)
和 [serial-usb.conf](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/serial-usb.conf)。
仔细查看这些文件可以说明如何在此 RTOS 中创建 micro-ROS 应用程序。

{% include first_application_common/build_and_flash.md %}

{% include first_application_common/agent_creation.md %}

然后，根据所选的传输和 RTOS，开发板与代理的连接方式可能有所不同。
在本教程中，我们使用 Olimex STM32-E407 USB 连接，Olimex 开发板通过 USB OTG 2 连接器（远离以太网端口的 MiniUSB 连接器）连接到计算机。

<img width="400" style="padding-right: 25px;" src="../imgs/6.jpg">

{% include first_application_common/run_app.md %}

{% include first_application_common/test_app_rtos.md %}

这完成了 Zephyr 上的第一个 micro-ROS 应用程序教程。您想[返回](../)并尝试不同的 RTOS，即 NuttX 或 FreeRTOS 吗？
