## 安装 ROS 2 和 micro-ROS 构建系统

首先，在 Ubuntu 22.04 LTS 计算机上安装 **ROS 2 Humble Hawksbill**。
要通过 Debian 软件包从二进制文件安装，请按照[此处](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html)详细说明的说明进行操作。

***提示：**或者，您可以使用包含全新 ROS 2 Humble 安装的 Docker 容器。合适的容器运行命令是：*

```bash
docker run -it --net=host -v /dev:/dev --privileged ros:humble
```

在计算机上安装 ROS 2 后，按照以下步骤安装 micro-ROS 构建系统：

```bash
# source ROS 2 安装目录
source /opt/ros/$ROS_DISTRO/setup.bash

# 创建工作空间并下载 micro-ROS 工具
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

这些说明将设置一个工作空间，其中包含可立即使用的 micro-ROS 构建系统。
该构建系统负责下载所需的交叉编译工具并为所需平台构建应用程序。

构建系统的工作流程包括四个步骤：

* **创建步骤：**此步骤负责下载特定硬件平台所需的所有代码仓库和交叉编译工具链。其中还包括一组可立即使用的 micro-ROS 应用程序。
* **配置步骤：**在此步骤中，用户可以选择将由工具链交叉编译的应用程序。其他一些选项，如传输方式、代理 IP 地址/端口（用于 UDP 传输）或设备 ID（用于串行连接）也将在此步骤中选择。
* **构建步骤：**在此进行交叉编译并生成特定平台的二进制文件。
* **烧录步骤：**将上一步生成的二进制文件烧录到硬件平台内存中，以允许执行 micro-ROS 应用程序。
