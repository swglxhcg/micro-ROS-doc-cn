## 测试 micro-ROS 应用程序

现在，我们想检查一切是否正常工作。

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

此时，我们知道我们的应用程序正在发布 ping。
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
ros2 topic pub --once /microROS/ping std_msgs/msg/Header '{frame_id: "fake_ping"}'
```

现在，我们应该在 `ping` 订阅者控制台中看到这个 `fake_ping`，以及 micro-ROS 的 ping：

```
user@user:~$ ros2 topic echo /microROS/ping
stamp:
  sec: 0
  nanosec: 0
frame_id: fake_ping
---
stamp:
  sec: 305
  nanosec: 973000000
frame_id: '451230256_1085377743'
---
stamp:
  sec: 310
  nanosec: 957000000
frame_id: '2084670932_1085377743'
---
```

此外，由于收到了 `fake_ping`，我们期望 micro-ROS 节点会回复一个 `pong`：

```
user@user:~$ ros2 run micro_ros_demos_rcl ping_pong
Ping send seq 1706097268_1085377743
Ping send seq 181171802_1085377743
Ping send seq 1385567526_1085377743
Ping send seq 926583793_1085377743
Ping send seq 1831510138_1085377743
Ping received with seq fake_ping. Answering.
Ping send seq 1508705084_1085377743
Ping send seq 1702133625_1085377743
Ping send seq 176104820_1085377743
```

因此，在 `pong` 订阅者控制台中，我们应该看到 micro-ROS 应用程序对我们的 `fake_ping` 的回复：

```
user@user:~$ ros2 topic echo /microROS/pong
stamp:
  sec: 0
  nanosec: 0
frame_id: fake_ping
---
```
