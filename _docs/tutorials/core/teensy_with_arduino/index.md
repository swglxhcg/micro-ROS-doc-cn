---
title: Teensy 与 Arduino
permalink: /docs/tutorials/core/teensy_with_arduino/
---

<img src="https://img.shields.io/badge/Written_for-Foxy-green" style="display:inline"/>

在本教程中，您将学习如何将 Teensy 与 micro-ROS 和 ROS 2 连接。
您还将学习如何在 Linux 系统中安装 micro-ROS 代理，以通过 Arduino IDE 与基于 Teensy 的 Arduino 板进行通信。
本教程还将介绍从 Teensy 发布的一个简单发布者主题，并使用 ROS 2 接口订阅它。

## 目标平台

首先，我们需要一台主机，要么安装有原生 Ubuntu 20.04 并运行 ROS 2 Foxy，要么使用从该链接构建的 docker 版本的 ROS 2 Foxy。
现在让我们也看一下连接图，这将帮助我们更好地理解整体情况。

![Teensy 3.2 与运行 ros2 和 micro-ros-agent 的主机 PC 连接图示](Teensy_micro_ros_connection.png)

## 在主机上安装 ROS 2 和 micro-ROS：

注意：这些最初几个步骤与 micro-ROS 安装页面中的步骤相同，见此链接

对于本教程，您必须在 Ubuntu 20.04 LTS 计算机上安装 ROS 2 Foxy Fitzroy。
您可以通过 Ubuntu 软件包从二进制文件安装，详细说明见[*此处*](https://docs.ros.org/en/foxy/Installation/Alternatives/Ubuntu-Install-Binary.html)。

注意：否则也可以通过运行以下命令使用新构建的 ROS 2 Foxy 的 docker 版本：

```bash
 sudo apt install docker.io
 sudo docker run -it --net=host -v /dev:/dev --privileged ros:foxy
```
运行 docker 后，按照命令验证 ROS2 是否正在运行并显示主题列表：

![主题图示](rostopic_show.png)

Docker 构建的 ROS 2 Foxy 版本也可以在无法从二进制文件安装原生 ROS 2 Foxy 的情况下使用，例如，运行 Jetpack 4.5 且使用 Ubuntu 18.04 的 Jetson Nano。

现在，一旦您在计算机或 docker 中安装了 ROS 2，请按照以下步骤安装 micro-ROS 构建系统：

```bash
# Source ROS 2 安装
source /opt/ros/foxy/setup.bash
# 创建工作区并下载 micro-ROS 工具
mkdir microros_ws
cd microros_ws
git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
# 使用 rosdep 更新依赖项
sudo apt update && rosdep update
rosdep install --from-paths src --ignore-src -y
# 安装 pip
sudo apt-get install python3-pip

# 构建 micro-ROS 工具并 source 它们
colcon build
source install/local_setup.bash
```

一旦 micro-ROS 安装完成，我们就可以继续在主机或 docker 版本中安装 micro-ROS 代理。
由于我们将使用 Teensy 3.2 和预编译的 micro-ROS 客户端库进行演示，我们不需要构建固件，
因此我们将跳过[*在 RTOS 上构建第一个 micro-ROS 应用教程*](../first_application_rtos/)中的固件构建步骤。

要安装 micro-ROS 代理，请按照以下步骤操作：

```bash
# 下载 micro-ROS 代理包
ros2 run micro_ros_setup create_agent_ws.sh
```
现在我们将构建代理包，完成后 source 安装：

```bash
# 构建步骤
ros2 run micro_ros_setup build_agent.sh
source install/local_setup.bash
```
现在，让我们通过运行以下命令来试运行 micro-ROS 代理：

```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0
```
结果应显示如下内容：

![代理运行图示](micro_ros_agent_start.png)

这意味着代理安装成功。
现在我们可以继续下一步，即安装 Arduino IDE 和 Teensyduino，并为使用预编译库修补基于 Arduino 的 Teensy 开发板，
如 [*micro_ros_arduino 仓库*](https://github.com/micro-ROS/micro_ros_arduino#patch-teensyduino) 中所述。

## 安装 Arduino IDE、Teensyduino 并设置用于将 Teensy 与 micro-ROS 和 ROS2 Foxy 配合使用的补丁：

请按照链接下载最新版本的 [*Arduino 1.8.15*](https://github.com/arduino/Arduino/releases/download/1.8.15/arduino-1.8.15.tar.xz) 
并按照此处的 [*Linux 版本链接*](https://www.arduino.cc/en/Guide/Linux) 进行安装。

安装 Arduino IDE 后，从 [*此处链接*](https://www.pjrc.com/teensy/td_154/TeensyduinoInstall.linux64) 下载 Teensyduino，
并按照 [*此页面*](https://www.pjrc.com/teensy/td_154/TeensyduinoInstall.linux64) 上显示的说明进行操作。
总结说明如下：

```
1. 下载 Linux udev 规则并将文件复制到 /etc/udev/rules.d。
https://www.pjrc.com/teensy/00-teensy.rules

2. 在终端中输入以下命令
$ sudo cp 00-teensy.rules /etc/udev/rules.d/

3. 下载并解压 Arduino 的 Linux 软件包之一。
注意：不支持来自 Linux 发行版软件包的 Arduino。

4. 下载相应的 Teensyduino 安装程序。

5. 在终端中添加执行权限后运行安装程序。
$ chmod 755 TeensyduinoInstall.linux64
$ ./TeensyduinoInstall.linux64
```
现在让我们设置 Teensy Arduino 的补丁以使用预编译的 micro-ros-client 库，打开终端窗口并按照以下命令操作：
有关更多信息，请遵循 [*micro_ros_arduino*](https://github.com/micro-ROS/micro_ros_arduino/tree/foxy) 的 GitHub 链接

```bash
# 对我来说它是 $ export ARDUINO_PATH=/home/manzur/arduino-1.8.13/
export ARDUINO_PATH=[您的 Arduino + Teensiduino 路径]

cd $ARDUINO_PATH/hardware/teensy/avr/

curl https://raw.githubusercontent.com/micro-ROS/micro_ros_arduino/foxy/extras/patching_boards/platform_teensy.txt > platform.txt
```

一旦上述说明完成，我们现在将能够使用 Teensy 3.2 并使用 Arduino IDE 通过预编译的 micro-ros-client 库对其进行编程。

## 对 Teensy 进行编程

现在我们已经修补了 Teensy Arduino IDE，我们将能够按照以下说明使用预编译库：

1. 转到 [*发布版本链接*](https://github.com/micro-ROS/micro_ros_arduino/releases) 
并下载用于 Arduino 的 micro-ROS 库的最后一个版本。
将文件放入 `/home/$USERNAME/Arduino/libraries/` 中，如下所示。

![补丁位置图示](patch_location.png)

一旦此过程完成，现在让我们查看下面的示例文件夹：

![示例位置图示](arduino_example_location.png)

对于本教程和测试，我们将使用上面显示的 mico-ros-publisher 示例，因为此程序只会发布在每个周期会增加的整数数据。
选择示例程序后，我们现在将代码上传到连接到主机的 Teensy 3.2，结果应如下所示。

![上传完成图示](upload_completion.png)

## 在 ROS 2 Foxy 中运行 micro-ROS 代理

现在，让我们暂时断开主机上的 Teensy 连接。然后我们将在 docker 中再次打开终端或运行代理程序，如步骤 2 末尾所示。
确保按照以下方式 source ROS 路径：

```bash
source /opt/ros/foxy/setup.bash
```

然后运行代理程序：

```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0
```
程序运行后将显示此消息：

![代理重启完成图示](agent_restart.png)

然后，我们将 Teensy 重新连接到主机，然后我们将看到连接完成并显示如下：

![代理已连接图示](agent_connected.png)

这意味着包含 micro-ros-client 的 Teensy 与主机上的 micro-ros-agent 之间的连接完成。
现在是重要时刻，测试从 Teensy 发布的 ROS 主题。
这次我们将打开另一个终端或 docker 窗口并输入如下内容：

```bash
ros2 topic list
```
应列出如下所示的内容：

![ros2 主题已连接图示](ros2_topic_all.png)

看，我们现在在主机上有了 `/micro_ros_arduino_node_publisher` 主题在发布。
如果监听该主题，我们将看到如下内容：

![ros2 主题显示数据图示](topic_show.png)

整数消息数据在每个周期增加。

_注意：本教程最初由作者 [Manzur Murshid](https://github.com/shazib2t) 发布于 [https://manzurmurshid.medium.com/how-to-connect-teensy-3-2-with-micro-ros-and-ros2-foxy-6c8f99c9b66a](https://manzurmurshid.medium.com/how-to-connect-teensy-3-2-with-micro-ros-and-ros2-foxy-6c8f99c9b66a)。_
