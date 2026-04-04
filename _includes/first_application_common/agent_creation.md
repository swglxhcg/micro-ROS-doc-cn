## 创建 micro-ROS 代理

现在 micro-ROS 应用程序已准备好连接到 micro-ROS 代理，以开始与 ROS 2 其他部分进行通信。
为此，让我们首先创建一个 micro-ROS 代理：

```bash
# 下载 micro-ROS-Agent 软件包
ros2 run micro_ros_setup create_agent_ws.sh
```

现在，让我们构建代理软件包，完成后source安装目录：

```bash
# 构建步骤
ros2 run micro_ros_setup build_agent.sh
source install/local_setup.bash
```
