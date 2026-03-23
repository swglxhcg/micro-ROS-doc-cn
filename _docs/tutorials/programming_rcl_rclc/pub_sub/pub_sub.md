---
title: 发布者和订阅者
permalink: /docs/tutorials/programming_rcl_rclc/pub_sub/
---

<img src="https://img.shields.io/badge/Written_for-Humble-green" style="display:inline"/>  <img src="https://img.shields.io/badge/Tested_on-Rolling-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Iron-green" style="display:inline"/>

ROS 2 发布者和订阅者是节点之间使用主题进行通信的基本机制。关于 ROS 2 发布-订阅模式的更多信息可以在[这里](https://docs.ros.org/en/humble/Tutorials/Topics/Understanding-ROS2-Topics.html)找到。

与这些概念相关的可直接使用的代码可以在 [`micro-ROS-demos/rclc/int32_publisher`](https://github.com/micro-ROS/micro-ROS-demos/blob/humble/rclc/int32_publisher/main.c) 和 [`micro-ROS-demos/rclc/int32_subscriber`](https://github.com/micro-ROS/micro-ROS-demos/blob/humble/rclc/int32_subscriber/main.c) 文件夹中找到。本教程中使用了这些示例的代码片段。

- [发布者](#publisher)
  - [初始化](#initialization)
  - [发布消息](#publish-a-message)
- [订阅者](#subscription)
  - [初始化](#initialization-1)
  - [回调](#callbacks)
- [消息初始化](#message-initialization)
- [清理](#cleaning-up)

## 发布者

### 初始化

从已初始化 RCL 并创建了 micro-ROS 节点的代码开始，根据所需的服务质量配置，有三种初始化发布者的方式：

- 可靠模式（默认）：
  ```c
  // 发布者对象
  rcl_publisher_t publisher;
  const char * topic_name = "test_topic";

  // 获取消息类型支持
  const rosidl_message_type_support_t * type_support =
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32);

  // 创建可靠的 rcl 发布者
  rcl_ret_t rc = rclc_publisher_init_default(
    &publisher, &node,
    type_support, topic_name);

  if (RCL_RET_OK != rc) {
    ...  // 处理错误
    return -1;
  }
  ```

- 尽力而为模式：
  ```c
  // 发布者对象
  rcl_publisher_t publisher;
  const char * topic_name = "test_topic";

  // 获取消息类型支持
  const rosidl_message_type_support_t * type_support =
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32);

  // 创建尽力而为的 rcl 发布者
  rcl_ret_t rc = rclc_publisher_init_best_effort(
    &publisher, &node,
    type_support, topic_name);

  if (RCL_RET_OK != rc) {
    ...  // 处理错误
    return -1;
  }
  ```

- 自定义 QoS：

  ```c
  // 发布者对象
  rcl_publisher_t publisher;
  const char * topic_name = "test_topic";

  // 获取消息类型支持
  const rosidl_message_type_support_t * type_support =
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32);

  // 设置发布者 QoS
  const rmw_qos_profile_t * qos_profile = &rmw_qos_profile_default;

  // 创建具有自定义服务质量选项的 rcl 发布者
  rcl_ret_t rc = rclc_publisher_init(
    &publisher, &node,
    type_support, topic_name, qos_profile);

  if (RCL_RET_OK != rc) {
    ...  // 处理错误
    return -1;
  }
  ```

  有关可用 QoS 选项以及可靠模式和尽力而为模式的优缺点的详细信息，请查看 [QoS 教程](../qos/)。

### 发布消息

向主题发布消息：

```c
// Int32 消息对象
std_msgs__msg__Int32 msg;

// 设置消息值
msg.data = 0;

// 发布消息
rcl_ret_t rc = rcl_publish(&publisher, &msg, NULL);

if (rc != RCL_RET_OK) {
  ...  // 处理错误
  return -1;
}
```

对于周期性发布，可以将 `rcl_publish` 放在定时器回调中。详细信息请查看[执行器和定时器](../executor/)部分。

注意：`rcl_publish` 是线程安全的，可以从多个线程调用。

## 订阅者

### 初始化

订阅者的初始化与发布者几乎相同：

- 可靠模式（默认）：
  ```c
  // 订阅者对象
  rcl_subscription_t subscriber;
  const char * topic_name = "test_topic";

  // 获取消息类型支持
  const rosidl_message_type_support_t * type_support =
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32);

  // 初始化可靠订阅者
  rcl_ret_t rc = rclc_subscription_init_default(
    &subscriber, &node,
    type_support, topic_name);

  if (RCL_RET_OK != rc) {
    ...  // 处理错误
    return -1;
  }
  ```

- 尽力而为模式：

  ```c
  // 订阅者对象
  rcl_subscription_t subscriber;
  const char * topic_name = "test_topic";

  // 获取消息类型支持
  const rosidl_message_type_support_t * type_support =
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32);

  // 初始化尽力而为订阅者
  rcl_ret_t rc = rclc_subscription_init_best_effort(
    &subscriber, &node,
    type_support, topic_name);

  if (RCL_RET_OK != rc) {
    ...  // 处理错误
    return -1;
  }
  ```

- 自定义 QoS：

  ```c
  // 订阅者对象
  rcl_subscription_t subscriber;
  const char * topic_name = "test_topic";

  // 获取消息类型支持
  const rosidl_message_type_support_t * type_support =
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32);

  // 设置客户端 QoS
  const rmw_qos_profile_t * qos_profile = &rmw_qos_profile_default;

  // 使用自定义服务质量选项初始化订阅者
  rcl_ret_t rc = rclc_subscription_init(
    &subscriber, &node,
    type_support, topic_name, qos_profile);

  if (RCL_RET_OK != rc) {
    ...  // 处理错误
    return -1;
  }
  ```

有关可用 QoS 选项以及可靠模式和尽力而为模式的优缺点的详细信息，请查看 [QoS 教程](../qos/)。

### 回调
执行器负责在发布消息时调用配置的回调函数。该函数将接收到的消息作为唯一参数，包含发布者发送的值：

```c
// 函数原型：
void (* rclc_subscription_callback_t)(const void *);

// 实现示例：
void subscription_callback(const void * msgin)
{
  // 将接收到的消息转换为使用的类型
  const std_msgs__msg__Int32 * msg = (const std_msgs__msg__Int32 *)msgin;

  // 处理消息
  printf("Received: %d\n", msg->data);
}
```

一旦订阅者和执行器初始化完成，必须将订阅者回调添加到执行器中，以便在其轮询时接收传入的发布：

```c
// 用于接收发布者数据的消息对象
std_msgs__msg__Int32 msg;

// 将订阅者添加到执行器
rcl_ret_t rc = rclc_executor_add_subscription(
  &executor, &subscriber, &msg,
  &subscription_callback, ON_NEW_DATA);

if (RCL_RET_OK != rc) {
  ...  // 处理错误
  return -1;
}

// 轮询执行器以接收消息
rclc_executor_spin(&executor);
```

## 消息初始化
在发布或接收消息之前，可能需要为包含字符串或序列的类型初始化其内存。
详细信息请查看 [在 micro-ROS 中处理消息内存](../../advanced/handling_type_memory/) 部分。

## 清理

完成发布者/订阅者后，节点将不再宣传它在该主题上发布/监听。
要销毁已初始化的发布者或订阅者：

```c
// 销毁发布者
rcl_publisher_fini(&publisher, &node);

// 销毁订阅者
rcl_subscription_fini(&subscriber, &node);
```

这将删除代理上自动创建的任何基础架构（如果可能），并释放客户端上使用的内存。
