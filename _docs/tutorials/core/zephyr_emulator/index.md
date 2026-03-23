---
title: Zephyr 模拟器
permalink: /docs/tutorials/core/zephyr_emulator/
---

<img src="https://img.shields.io/badge/Tested_on-Humble-green" style="display:inline"/>

在本教程中，您将通过测试 Ping Pong 应用程序来学习将 micro-ROS 与 **[Zephyr RTOS](https://www.zephyrproject.org/)** 模拟器（也称为 [Native POSIX](https://docs.zephyrproject.org/4.1.0/boards/native/native_posix/doc/index.html)）结合使用。

<div>
<img  width="300" style="padding-right: 25px;" src="/img/posts/logo-zephyr.jpg">
</div>

{% include first_application_common/build_system.md %}

```bash
# 创建步骤
ros2 run micro_ros_setup create_firmware_ws.sh zephyr host
```

{% include first_application_common/zephyr_common.md %}

{% include first_application_common/config.md %}

在本教程中，我们将使用 UDP 传输，该传输在 localhost 的 UDP/8888 端口上查找代理，并重点介绍位于 `firmware/zephyr_apps/apps/ping_pong` 的开箱即用的 `ping_pong` 应用程序。要使用所选传输执行此应用程序，请通过如下指定 `[APP]` 和 `[OPTIONS]` 参数来运行上述配置命令：

```bash
# 配置步骤
ros2 run micro_ros_setup configure_firmware.sh ping_pong --transport udp --ip 127.0.0.1 --port 8888
```

您可以在[此处](https://github.com/micro-ROS/zephyr_apps/tree/humble/apps/ping_pong)查看 `ping_pong` 应用程序的完整内容。

{% include first_application_common/pingpong_logic.md %}

Zephyr 应用程序特定文件的内容可以在以下位置找到：
[main.c](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/src/main.c)、
[app-colcon.meta](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/app-colcon.meta)、
[CMakeLists.txt](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/CMakeLists.txt)
和 [host-udp.conf](https://github.com/micro-ROS/zephyr_apps/blob/humble/apps/ping_pong/host-udp.conf)。
仔细查看这些文件可以说明如何在此 RTOS 中创建 micro-ROS 应用程序。

## 构建固件

当配置步骤结束时，只需构建固件：

```bash
# 构建步骤
ros2 run micro_ros_setup build_firmware.sh
```

现在您拥有了一个可以在自己计算机上运行的 Zephyr + micro-ROS 应用程序。
请注意，在这种情况下，烧录固件和运行 micro-ROS 应用程序的步骤是结合在一起的。

{% include first_application_common/agent_creation.md %}

## 运行 micro-ROS 应用程序

此时，您的主机上已正确安装好客户端和代理。

要使 micro-ROS 访问 ROS 2 数据空间，请运行代理：

```bash
# 运行 micro-ROS 代理
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
```

## 烧录固件

最后，为了在 Zephyr RTOS 模拟器中运行 micro-ROS 节点，请打开一个新的命令 shell 并通过烧录命令执行烧录步骤：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash
source install/local_setup.bash

# 烧录/运行步骤
ros2 run micro_ros_setup flash_firmware.sh
```

{% include first_application_common/test_app_host.md %}

## 多个 Ping Pong 节点

拥有模拟器的优势之一是您无需购买大量硬件即可测试一些多节点 micro-ROS 应用程序。
因此，使用上一节中的相同 micro-ROS 代理，让我们打开四个不同的命令行并在每个命令行上运行以下命令：

```bash
cd microros_ws

# 这是执行 Zephyr 模拟器的替代方式
./firmware/build/zephyr/zephyr.exe
```

一旦所有 micro-ROS 节点启动并连接到 micro-ROS 代理，您将看到它们相互通信：

```
user@user:~$ ./firmware/build/zephyr/zephyr.exe
*** Booting Zephyr OS build zephyr-v2.2.0-492-gc73cb85b4ae9  ***
Ping send seq 1711620172_1742614911                         <---- 此 micro-ROS 节点发送一个 ping，ping ID 为 "1711620172"，节点 ID 为 "1742614911"
Pong for seq 1711620172_1742614911 (1)                      <---- 第一个伙伴回复我的 ping
Pong for seq 1711620172_1742614911 (2)                      <---- 第二个伙伴回复我的 ping
Pong for seq 1711620172_1742614911 (3)                      <---- 第三个伙伴回复我的 ping
Ping received with seq 1845948271_546591567. Answering.     <---- 收到来自标识为 "546591567" 的伙伴的 ping，让我们回复它。
Ping received with seq 232977719_1681483056. Answering.     <---- 收到来自标识为 "1681483056" 的伙伴的 ping，让我们回复它。
Ping received with seq 1134264528_1107823050. Answering.    <---- 收到来自标识为 "1107823050" 的伙伴的 ping，让我们回复它。
Ping send seq 324239260_1742614911
Pong for seq 324239260_1742614911 (1)
Pong for seq 324239260_1742614911 (2)
Pong for seq 324239260_1742614911 (3)
Ping received with seq 1435780593_546591567. Answering.
Ping received with seq 2034268578_1681483056. Answering.
```

***提示：** 使用帮助标志可以发现一些 Zephyr 模拟器功能 `./firmware/build/zephyr/zephyr.exe -h`*
