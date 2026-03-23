---
title: micro-ROS 构建系统
permalink: /docs/concepts/build_system/
---

micro-ROS 提供了两种为嵌入式平台构建 micro-ROS 应用的方法：
- _micro_ros_setup:_ 将 RTOS 特定的构建工具集成并隐藏在极少数脚本中，这些脚本作为 ROS 2 包提供。
- _平台特定集成:_ 我们已将 micro-ROS 与多个平台的构建工具集成。点击[此处](/docs/concepts/build_system/external_build_systems/)了解更多。

**micro_ros_setup** 提供了一个独立构建系统，以 ROS 2 包的形式用于任何常规 ROS 2 工作区。此工具可在 [micro-ROS/micro_ros_setup](https://github.com/micro-ROS/micro_ros_setup) 存储库中找到。

**micro_ros_setup** 工具允许编译和生成包含 micro-ROS 应用的镜像，用于[支持的硬件](/docs/overview/hardware/)板和 [RTOS](/docs/concepts/rtos/)。

由于 **micro_ros_setup** 包可以像任何其他 ROS 2 包一样安装，其使用将通过 ROS 2 CLI 工具完成。编译、生成镜像和将其烧录到板上只需四个 ROS 2 命令即可完成。关于此包使用方法的详细说明可在[教程部分](/docs/tutorials/core/first_application_rtos/)中找到。

### micro-ROS 客户端

安装后，构建系统工具提供了一些实用程序，可用于准备、构建、烧录和使用 micro-ROS 应用。micro-ROS 构建系统是一个四步过程。在第一步中，用户可以通过配置目标硬件和 RTOS 创建新的 micro-ROS 应用：

```bash
# 创建步骤
ros2 run micro_ros_setup create_firmware_ws.sh [RTOS] [HARDWARE BOARD]
```

可以通过不带任何参数运行命令来获取支持的硬件列表。通过这样做，可以看到除了 micro-ROS 支持的 RTOS 和硬件外，此构建系统还提供三个额外选项：
- 使用 `zephyr` 作为 RTOS 和 `host` 作为硬件名称，可以获取在主机上运行的带有 micro-ROS 应用的 Zephyr RTOS 镜像。
- 仅使用 `host` 作为 RTOS，micro-ROS 将在主机上本地构建一组 [micro-ROS 演示应用](https://github.com/micro-ROS/micro-ROS-demos)。这些应用的行为就像 micro-ROS 应用一样（使用相同的抽象层和中间件实现），允许用户在 PC 上调试和测试应用。
- 使用 `generate_lib` 作为 RTOS，可以为生成静态库 (`.a`) 和一组头文件 (`include`) 配置构建系统，这些可以链接到任何其他外部工具。此选项需要有效的 CMake 工具链。

一旦构建系统创建了新的固件项目，就可以使用以下命令对其进行配置：

```bash
# 配置步骤
ros2 run micro_ros_setup configure_firmware.sh [APP] [OPTIONS]
```

不带任何参数运行此命令将输出适用于所选 RTOS 的示例应用列表。
此配置步骤的常用选项包括：
  - `--transport` 或 `-t`：`udp`、`serial` 或任何硬件特定传输标签
  - `--dev` 或 `-d`：类似串口的传输中的代理字符串描述符
  - `--ip` 或 `-i`：网络类传输中的代理 IP
  - `--port` 或 `-p`：网络类传输中的代理端口

最后，可以使用以下命令构建和烧录 micro-ROS 应用：

```bash
# 构建步骤
ros2 run micro_ros_setup build_firmware.sh

# 烧录步骤
ros2 run micro_ros_setup flash_firmware.sh
```

### micro-ROS 代理

micro-ROS 构建系统还能够通过使用以下命令简化在 ROS 2 工作区中编译 micro-ROS 代理的过程：

```bash
# 下载 micro-ROS-Agent 包
ros2 run micro_ros_setup create_agent_ws.sh
ros2 run micro_ros_setup build_agent.sh
source install/local_setup.bash
ros2 run micro_ros_agent micro_ros_agent [OPTIONS]
```

**提示 1：** 要了解 micro_ros_setup 构建系统的实际使用，请参阅[核心教程](https://micro-ros.github.io/docs/tutorials/core/first_application_rtos/)。

**提示 2：** 请记住，micro-ROS 代理也可以使用此简单 Docker 命令使用：`docker run -it --rm -v /dev:/dev --privileged --net=host microros/micro-ros-agent:$ROS_DISTRO [OPTIONS]`
