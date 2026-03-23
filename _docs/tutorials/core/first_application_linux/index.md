---
title: Linux 上的第一个 micro-ROS 应用
permalink: /docs/tutorials/core/first_application_linux/
---

<img src="https://img.shields.io/badge/Written_for-Humble-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Rolling-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Iron-green" style="display:inline"/>

在本教程中，您将通过测试 Ping Pong 应用程序来学习在 Linux 上使用 micro-ROS。
在后续教程 [*在 RTOS 上构建第一个 micro-ROS 应用* 中，您将学习如何构建并运行此应用程序在运行 NuttX、FreeRTOS 或 Zephyr RTOS 的微控制器上。
最后，在 [*Zephyr 模拟器* 教程中，您将学习如何在 Zephyr 模拟器上测试 micro-ROS 应用程序。

{% include first_application_common/build_system.md %}

```bash
# 创建固件步骤
ros2 run micro_ros_setup create_firmware_ws.sh host
```

执行此命令后，您的工作空间中必须存在一个名为 `firmware` 的文件夹。

此步骤负责（除其他事项外）下载一组用于 Linux 的 micro-ROS 应用程序，这些应用程序位于
`src/uros/micro-ROS-demos/rclc`。
每个应用程序由一个包含以下文件的文件夹表示：

* `main.c`：此文件包含应用程序的逻辑。
* `CMakeLists.txt`：这是包含编译应用程序脚本的 CMake 文件。

为了使用户能够创建自定义应用程序，需要在此位置注册一个名为 `<my_app>` 的文件夹，
其中包含刚才描述的两个文件。
此外，任何此类新应用程序文件夹都需要通过添加以下行来注册到
`src/uros/micro-ROS-demos/rclc/CMakeLists.txt`：

```
export_executable(<my_app>)
```

在本教程中，我们将重点介绍位于
`src/uros/micro-ROS-demos/rclc/ping_pong` 的开箱即用的 `ping_pong` 应用程序。
您可以在[此处](https://github.com/micro-ROS/micro-ROS-demos/tree/humble/rclc/ping_pong)查看此应用程序的完整内容。

{% include first_application_common/pingpong_logic.md %}

主机应用程序特定文件的内容可以在以下位置找到：
[main.c](https://github.com/micro-ROS/micro-ROS-demos/blob/humble/rclc/ping_pong/main.c) 和
[CMakeLists.txt](https://github.com/micro-ROS/micro-ROS-demos/blob/humble/rclc/ping_pong/CMakeLists.txt)。
仔细查看这些文件可以说明如何在此 RTOS 中创建 micro-ROS 应用程序。

## 构建固件

创建应用程序后，接下来是构建步骤。
请注意，关于上述四步工作流程，我们期望在构建应用程序之前会发生配置步骤。但是，由于我们是在主机上而不是在开发板上编译 micro-ROS，
因此配置步骤实现的交叉编译在此情况下不是必需的。
因此，我们可以继续构建固件并执行本地安装：

```bash
# 构建步骤
ros2 run micro_ros_setup build_firmware.sh
source install/local_setup.bash
```
{% include first_application_common/agent_creation.md %}

### 将 micro-ROS 环境添加到 bashrc（可选）

您可以将 ROS 2 和 micro-ROS 工作区设置文件添加到您的 `.bashrc`，这样每次打开新命令行时就不需要手动执行 source 命令了。
```bash
echo source /opt/ros/$ROS_DISTRO/setup.bash >> ~/.bashrc
echo source ~/microros_ws/install/local_setup.bash >> ~/.bashrc
```

## 运行 micro-ROS 应用程序

此时，您的主机上已正确安装好客户端和代理。

要使 micro-ROS 访问 ROS 2 数据空间，请运行代理：

```bash
# 运行 micro-ROS 代理
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
```

然后，在另一个命令行中，运行 micro-ROS 节点（请记得 source ROS 2 和 micro-ROS 安装，并设置 RMW Micro XRCE-DDS 实现）：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash
source install/local_setup.bash

# 使用 RMW Micro XRCE-DDS 实现
export RMW_IMPLEMENTATION=rmw_microxrcedds

# 运行 micro-ROS 节点
ros2 run micro_ros_demos_rclc ping_pong
```

{% include first_application_common/test_app_host.md %}

## 多个 Ping Pong 节点

拥有 Linux micro-ROS 应用程序的优势之一是您无需购买大量硬件即可测试一些多节点 micro-ROS 应用程序。
因此，使用上一节中的相同 micro-ROS 代理，让我们打开四个不同的命令行并在每个命令行上运行以下命令：

```bash
cd microros_ws

source /opt/ros/$ROS_DISTRO/setup.bash
source install/local_setup.bash

export RMW_IMPLEMENTATION=rmw_microxrcedds

ros2 run micro_ros_demos_rclc ping_pong
```

一旦所有 micro-ROS 节点启动并连接到 micro-ROS 代理，您将看到它们相互通信：

```
user@user:~$ ros2 run micro_ros_demos_rclc ping_pong
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
