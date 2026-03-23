---
title: 在 RTOS 上构建第一个 micro-ROS 应用
permalink: /docs/tutorials/core/first_application_rtos/
---

在完成 [在 Linux 上构建第一个 micro-ROS 应用教程](../first_application_linux) 后，您现在可以将此应用程序烧录到基于实时操作系统 (RTOS) 的微控制器上了。

Micro-ROS 目前支持三个不同的 RTOS，即 NuttX、FreeRTOS 和 Zephyr。当然，应用程序代码中与 micro-ROS 相关的部分独立于底层 RTOS。此外，基本工具与集成 RTOS 工具和 ROS 2 元构建系统 colcon 相同。但是，三种 RTOS 在配置和可执行文件定义方面存在细微差别。因此，在本教程中，请选择要使用的一个 RTOS：

<table style="border:none;">
 <tr>
  <td style="width:33%; text-align:center; vertical-align:bottom; font-weight:bold;"><a href="nuttx/"><img style="margin-left:auto; margin-right:auto; padding-bottom:5px;" width="125" height="125" src="https://upload.wikimedia.org/wikipedia/commons/b/b0/NuttX_logo.png"><br/>NuttX</a></td>
  <td style="width:33%; text-align:center; vertical-align:bottom; font-weight:bold;"><a href="freertos/"><img style="margin-left:auto; margin-right:auto; padding-bottom:5px;" width="263" height="100" src="https://upload.wikimedia.org/wikipedia/commons/4/4e/Logo_freeRTOS.png"><br/>FreeRTOS</a></td>
  <td style="width:33%; text-align:center; vertical-align:bottom; font-weight:bold;"><a href="zephyr/"><img style="margin-left:auto; margin-right:auto; padding-bottom:5px;" width="220" height="114" src="/img/posts/logo-zephyr.jpg"><br/>Zephyr</a></td>
 </tr>
</table>

{% include logos_disclaimer.md %}
