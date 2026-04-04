---
title: Micro XRCE-DDS 与 rosserial 的比较
permalink: /docs/concepts/middleware/rosserial/
---

在 ROS 中，当我们需要通过串行通信发送 ROS 消息时，有一个包特别突出，那就是 rosserial。

rosserial 允许基于微控制器的平台与普通计算机通信，代表其连接到 ROS 网络。
rosserial 提供了设置这种通信的协议，使用客户端-服务器架构方法。
rosserial-client 将数据序列化到串行链路，然后，序列化后的数据由 rosserial-server 接收并转发到传统 ROS 网络。
一个类似的过程用于将数据从 ROS 网络转发到微控制器。
这个 rosserial-server 可以用 C++ 或 Python 使用；同时，rosserial-client 有一套可用的受支持微处理器。

此解决方案通常用于集成使用 ROS 的机器人中的硬件部件。
在这些情况下，Rosserial 充当硬件通信协议和 ROS 网络之间的桥梁。

## Micro XRCE-DDS

Micro XRCE-DDS 的功能之一是在微控制器和具有 DDS/ROS 2 功能的计算机之间使用串行连接。
这种连接得益于使用 OMG 的 DDS-XRCE 标准和串行传输层。
此解决方案遵循与 rosserial 相同的客户端-服务器架构，这是当我们谈论微控制器通信时最合适的方法之一。

负责实现此架构的库是 Client 和 Agent。
客户端在 Agent 中生成实体，这些实体将代表客户端在 DDS 网络上运行。

如您所见，这种用法与 rosserial 相似，但它们的实现方式有细微差别，我们将在本文中介绍。

## Micro XRCE-DDS 与 rosserial

现在我们对 rosserial 和 Micro XRCE-DDS 有了基本了解，我们将对它们进行比较。

### Micro XRCE-DDS 串行传输

Micro XRCE-DDS 通过串行传输通信时（它允许通过其他传输进行通信，如 UDP、TCP……），使用具有预定义格式的串行协议。
此格式在以下帧解析中说明：

```
0        8        16       24                40                 X                 X+16
+--------+--------+--------+--------+--------+--------//--------+--------+--------+
| FLAG   |  SADD  |  RADD  |       LEN       |     PAYLOAD      |       CRC       |
+--------+--------+--------+--------+--------+--------//--------|--------+--------+
```

* `FLAG`：同步标志 (0xFF)。
* `SADD`：源地址。
* `RADD`：远程地址。
* `LEN`：不包括组帧的有效载荷长度（2 字节，小端序）。
* `PAYLOAD`：带有 XRCE 头部的序列化消息。
* `CRC`：填充后的消息 CRC。

这是在线路层序列化的消息。
这是由于两种不同操作（发布和订阅）而从微控制器进出的消息。

## rosserial

相反，这是 rosserial 帧：

```
0       8       16              32              40      56               X             X+16
+-------+-------+-------+-------+-------+-------+-------+-------//-------+------+------+
| FLAG  | PROT  |      LEN      |      LCRC     | TOPID |     PAYLOAD    |     MCRC    |
+-------+-------+-------+-------+-------+-------+-------+-------+-------|------+------+
```

* `FLAG`：同步标志 (0xFF)。
* `PROT`：协议版本。
* `LEN`：有效载荷长度（2 字节，小端序）。
* `LCRC`：长度 CRC。
* `TOPID`：主题 ID。
* `PAYLOAD`：序列化消息。
* `MCRC`：消息 CRC。

如您所见，与 Micro XRCE-DDS 串行帧比较，它使用了完全不同的帧。

## 比较

下表总结了两者实现的关键方面：

| | Micro XRCE-DDS 串行 | rosserial |
