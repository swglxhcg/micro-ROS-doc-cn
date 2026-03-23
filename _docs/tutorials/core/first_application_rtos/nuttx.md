---
title: NuttX 上的第一个 micro-ROS 应用
permalink: /docs/tutorials/core/first_application_rtos/nuttx/
redirect_from:
  - /docs/tutorials/advanced/nuttx/nuttx_getting_started/
---

<img src="https://img.shields.io/badge/Tested_on-Humble-green" style="display:inline"/>

在本教程中，您将通过测试 Ping Pong 应用程序来学习在 NuttX 上使用 micro-ROS。
{% include first_application_common/target_hardware.md %}
* [USB转串口线 母头](https://www.olimex.com/Products/Components/Cables/USB-Serial-Cable/USB-SERIAL-F/)
* [USB转Mini-USB线](https://www.olimex.com/Products/Components/Cables/CABLE-USB-A-MINI-1.8M/)

{% include first_application_common/build_system.md %}

```bash
# 创建步骤
ros2 run micro_ros_setup create_firmware_ws.sh nuttx olimex-stm32-e407
```

执行此命令后，您的工作空间中必须存在一个名为 `firmware` 的文件夹。

此步骤负责（除其他事项外）下载针对您要处理的特定平台的 micro-ROS 应用程序集。
对于 NuttX，这些应用程序位于[此处](https://github.com/micro-ROS/nuttx_apps/tree/foxy/examples)。
每个应用程序由一个包含以下文件的文件夹表示：

* `app.c`：此文件包含应用程序的逻辑。
* `Kconfig`：此文件包含 NuttX Kconfig 配置。
* `Make.defs`：此文件包含 NuttX 构建系统定义。
* `Makefile`：此文件包含特定于 NuttX 的应用程序构建脚本。

## 配置固件

配置步骤将设置主要的 micro-ROS 选项并选择所需的应用程序。
可以使用以下命令执行：

```bash
# 配置步骤
ros2 run micro_ros_setup configure_firmware.sh [APP] [OPTIONS]
```

在本教程中，我们将使用串行传输，并重点介绍位于[此处](https://github.com/micro-ROS/nuttx_apps/tree/foxy/examples/uros_pingpong)的开箱即用的 `uros_pingpong` 应用程序。
要使用所选传输执行此应用程序，请通过如下指定 `[APP]` 参数来运行上述配置命令：

```bash
# 使用 ping_pong 应用程序和串口-USB 传输进行配置步骤
ros2 run micro_ros_setup configure_firmware.sh pingpong
```

且不带 `[OPTIONS]` 参数。

还提供了一个预配置的以太网示例：
```bash
# 使用 ping_pong 应用程序和串口-USB 传输进行配置步骤
ros2 run micro_ros_setup configure_firmware.sh pingpong-eth
```

要继续进行配置，请克隆以下 NuttX 工具仓库：

```bash
# 下载使用 NuttX 所需的工具
git clone https://bitbucket.org/nuttx/tools.git firmware/tools
```

然后安装所需的 `kconfig-frontends`：

```bash
pushd firmware/tools/kconfig-frontends
./configure
make

# 如果 make 命令失败，输入：autoreconf -f -i ，然后重新运行 make 命令。

sudo make install
sudo ldconfig
popd
```

现在我们有两种配置 micro-ROS 传输的方式：

- 交互式 NuttX 菜单配置
  * 启动配置菜单：

    ```bash
    cd firmware/NuttX
    make menuconfig
    ```

  * 您可以在 `Application Configuration ---> Examples ---> micro-ROS Ping Pong` 下检查是否已选择该应用程序。
  * 传输也已预配置在 `Application Configuration ---> micro-ROS ---> Transport` 选项下。
  * 要配置传输，对于 UDP 使用 `IP address of the agent` 和 `Port number of the agent` 选项，对于串行示例使用 `Serial port to use`。
  * 要保存更改，使用左右箭头导航到底部菜单，然后点击 `Save` 按钮。
  * 系统会询问您是否要保存新的 `.config` 配置，您需要点击 `Ok`，然后点击 `Exit`。
  * 按三次 `Esc` 键关闭菜单并返回 `microros_ws`：

      ```bash
      cd ../..
      ```

- `kconfig-tweak` 控制台命令：
  * 进入 Nuttx 配置路径：

    ```bash
    cd firmware/NuttX
    ```

  * UDP 传输配置：
    ```bash
    kconfig-tweak --set-val CONFIG_UROS_AGENT_IP "127.0.0.1"
    kconfig-tweak --set-val CONFIG_UROS_AGENT_PORT 8888
    ```

  * 串行传输配置：
    ```bash
    kconfig-tweak --set-val CONFIG_UROS_SERIAL_PORT "/dev/ttyS0"
    ```

您可以在[此处](https://github.com/micro-ROS/nuttx_apps/tree/foxy/examples/uros_pingpong)查看 `uros_pingpong` 应用程序的完整内容。

{% include first_application_common/pingpong_logic.md %}

FreeRTOS 应用程序特定文件的内容可以在以下位置找到：
[app.c](https://github.com/micro-ROS/nuttx_apps/blob/foxy/examples/uros_pingpong/app.c)、
[Kconfig](https://github.com/micro-ROS/nuttx_apps/blob/foxy/examples/uros_pingpong/Kconfig)、
[Make.defs](https://github.com/micro-ROS/nuttx_apps/blob/foxy/examples/uros_pingpong/Make.defs) 和
[Makefile](https://github.com/micro-ROS/nuttx_apps/blob/foxy/examples/uros_pingpong/Makefile)。
仔细查看这些文件可以说明如何在此 RTOS 中创建 micro-ROS 应用程序。

{% include first_application_common/build_and_flash.md %}

{% include first_application_common/agent_creation.md %}

然后，根据所选的传输和 RTOS，开发板与代理的连接方式可能有所不同。
在本教程中，我们使用 Olimex STM32-E407 串行连接，Olimex 开发板通过 usb 转串口线连接到计算机。

<img width="400" style="padding-right: 25px;" src="../imgs/5.jpg">

此外，您还需要将 USB 转 Mini-USB 线连接到 USB OTG 1 连接器（靠近以太网端口的 MiniUSB 连接器）。

<img width="500" style="padding-right: 25px;" src="../imgs/7.jpg">

***提示：** 颜色代码适用于[此电缆](https://www.olimex.com/Products/Components/Cables/USB-Serial-Cable/USB-SERIAL-F/)。
请确保将 Olimex Rx 与电缆 Tx 匹配，反之亦然。记得接地！*

## 运行 micro-ROS 应用程序

此时，您已正确安装好客户端和代理。

要使 micro-ROS 访问 ROS 2 数据空间，请运行代理：

```bash
# 运行 micro-ROS 代理
ros2 run micro_ros_agent micro_ros_agent serial --dev [device]
```

***提示：** 您可以使用此命令查找您的串行设备名称：`ls /dev/serial/by-id/*`*

然后，要启动 micro-ROS 应用程序，您需要安装并打开 Minicom，这是一个基于文本的串行端口通信程序。打开一个新的 shell，并输入：

```bash
sudo minicom -D [device] -b 115200
```

***提示：** 您可以使用此命令查找您的串行设备名称：`ls /dev/serial/by-id/*`。选择以 `usb-NuttX` 开头的那个。*

从 Minicom 应用程序内部，按三次 `Enter` 键直到出现 Nuttx Shell (NSH)。一旦进入 NSH 命令行，输入：

```bash
uros_pingpong
```

{% include first_application_common/test_app_rtos.md %}

这完成了 NuttX 上的第一个 micro-ROS 应用程序教程。您想[返回](../)并尝试不同的 RTOS，即 FreeRTOS 或 Zephyr 吗？
