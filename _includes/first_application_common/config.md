## 配置固件

配置步骤将设置主要的 micro-ROS 选项并选择所需的应用程序。
可以使用以下命令执行：

```bash
# 配置步骤
ros2 run micro_ros_setup configure_firmware.sh [APP] [OPTIONS]
```

此配置步骤的可用选项：
  - `--transport` 或 `-t`：`udp`、`serial` 或任何硬件特定的传输标签
  - `--dev` 或 `-d`：串行类传输中的代理字符串描述符
  - `--ip` 或 `-i`：网络类传输中的代理 IP
  - `--port` 或 `-p`：网络类传输中的代理端口
