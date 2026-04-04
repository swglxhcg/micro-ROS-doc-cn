## 测试 micro-ROS 应用程序

此时，micro-ROS 应用程序已构建并烧录，开发板已连接到 micro-ROS 代理。
现在我们想检查一切是否正常工作。

打开一个新的命令行。我们将使用 ROS 2 监听 `ping` 主题，以检查 micro-ROS Ping Pong 节点是否正确发布预期的 ping：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash

# 订阅 micro-ROS ping 主题
ros2 topic echo /microROS/ping
```

您应该看到 Ping Pong 节点每 5 秒发布的主题消息：

```
user@user:~$ ros2 topic echo /microROS/ping
stamp:
  sec: 20
  nanosec: 867000000
frame_id: '1344887256_1085377743'
---
stamp:
  sec: 25
  nanosec: 942000000
frame_id: '730417256_1085377743'
---
```

此时，我们知道我们的 micro-ROS 应用程序正在发布 ping。
让我们检查它是否也会响应其他人的 ping。如果正常工作，它将发布一个 pong。

因此，首先让我们从新的 shell 中使用 ROS 2 订阅 `pong` 主题（请注意，最初我们不期望收到任何 pong，因为还没有发送）：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash

# 订阅 micro-ROS pong 主题
ros2 topic echo /microROS/pong
```

现在，让我们从另一个命令行使用 ROS 2 发布一个 `fake_ping`：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash

# 发送一个假 ping
