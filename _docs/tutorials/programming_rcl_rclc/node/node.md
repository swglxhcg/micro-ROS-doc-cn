---
title: 节点
permalink: /docs/tutorials/programming_rcl_rclc/node/
---

<img src="https://img.shields.io/badge/Written_for-Humble-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Rolling-green" style="display:inline"/> <img src="https://img.shields.io/badge/Tested_on-Iron-green" style="display:inline"/>

ROS 2 节点是 ROS 2 生态系统中的主要参与者。它们将通过发布者、订阅者、服务等进行相互通信。关于 ROS 2 节点的更多信息可以在[这里](https://docs.ros.org/en/iron/Tutorials/Understanding-ROS2-Nodes.html)找到


- [初始化](#initialization)
  - [清理](#cleaning-up)
- [生命周期](#lifecycle)
  - [初始化](#initialization-1)
  - [回调](#callbacks)
  - [运行](#running)
  - [清理](#cleaning-up-1)
  - [限制](#limitations)

## 初始化

- 使用默认配置创建节点：
  ```c
  // 初始化 micro-ROS 分配器
  rcl_allocator_t allocator = rcl_get_default_allocator();

  // 初始化支持对象
  rclc_support_t support;
  rcl_ret_t rc = rclc_support_init(&support, argc, argv, &allocator);

  // 创建节点对象
  rcl_node_t node;
  const char * node_name = "test_node";

  // 节点命名空间（可以保留为空 ""）
  const char * namespace = "test_namespace";

  // 初始化默认节点
  rc = rclc_node_init_default(&node, node_name, namespace, &support);
  if (rc != RCL_RET_OK) {
    ... // 处理错误
    return -1;
  }
  ```

- 使用自定义选项创建节点：

  节点的配置也将应用于其未来的元素（发布者、订阅者、服务等）。节点选项使用自定义 API 在 `rclc_support_t` 对象上进行配置：

  ```c
  // 初始化 micro-ROS 分配器
  rcl_allocator_t allocator = rcl_get_default_allocator();

  // 初始化并修改选项（设置 DOMAIN ID 为 10）
  rcl_init_options_t init_options = rcl_get_zero_initialized_init_options();
  rcl_init_options_init(&init_options, allocator);
  rcl_init_options_set_domain_id(&init_options, 10);

  // 使用自定义选项初始化 rclc 支持对象
  rclc_support_t support;
  rclc_support_init_with_options(&support, 0, NULL, &init_options, &allocator);

  // 创建节点对象
  rcl_node_t node;
  const char * node_name = "test_node";

  // 节点命名空间（可以保留为空 ""）
  const char * namespace = "test_namespace";

  // 使用配置的支持对象初始化节点
  rclc_node_init_default(&node, node_name, namespace, &support);

  if (rc != RCL_RET_OK) {
    ... // 处理错误
    return -1;
  }
  ```

### 清理

要销毁已初始化的节点，必须先销毁节点拥有的所有实体（发布者、订阅者、服务等），然后再销毁节点本身：

```c
// 销毁创建的实体（示例）
rcl_publisher_fini(&publisher, &node);
...

// 销毁节点
rcl_node_fini(&node);
```

这将从 ROS2 图中删除节点，包括代理上生成的任何基础架构（如果可能）和客户端上使用的内存。

## 生命周期

rclc 生命周期包提供了 C 语言中的便捷函数，用于将 rcl 节点与 ROS 2 节点生命周期状态机捆绑在一起，类似于 C++ 的 [rclcpp Lifecycle Node](https://github.com/ros2/rclcpp/blob/master/rclcpp_lifecycle/include/rclcpp_lifecycle/lifecycle_node.hpp)。关于 ROS 2 节点生命周期的更多信息可以在[这里](https://design.ros2.org/articles/node_lifecycle.html)找到

使用示例见 [rclc_examples](https://github.com/ros2/rclc/blob/master/rclc_examples/src/example_lifecycle_node.c) 包。

### 初始化

创建生命周期节点作为 rcl 节点和 rcl 生命周期状态机的捆绑。假设已初始化节点和执行器：

```c
// 创建 rcl 状态机
rcl_lifecycle_state_machine_t state_machine =
rcl_lifecycle_get_zero_initialized_state_machine();

// 创建生命周期节点
rclc_lifecycle_node_t my_lifecycle_node;
rcl_ret_t rc = rclc_make_node_a_lifecycle_node(
  &my_lifecycle_node,
  &node,
  &state_machine,
  &allocator);

// 在分配器上注册生命周期服务
rclc_lifecycle_add_get_state_service(&lifecycle_node, &executor);
rclc_lifecycle_add_get_available_states_service(&lifecycle_node, &executor);
rclc_lifecycle_add_change_state_service(&lifecycle_node, &executor);
```

*注意：执行器需要为每个节点和每个服务配备 1 个句柄*

### 回调

支持可选回调以在生命周期状态变化时执行操作。示例：

```c
rcl_ret_t my_on_configure() {
  printf("  >>> my_lifecycle_node: on_configure() callback called.\n");
  return RCL_RET_OK;
}
```

将它们添加到生命周期节点：

```c
// 注册生命周期服务回调
rclc_lifecycle_register_on_configure(&lifecycle_node, &my_on_configure);
rclc_lifecycle_register_on_activate(&lifecycle_node, &my_on_activate);
rclc_lifecycle_register_on_deactivate(&lifecycle_node, &my_on_deactivate);
rclc_lifecycle_register_on_cleanup(&lifecycle_node, &my_on_cleanup);
```

### 运行

更改生命周期节点的状态：

```c
bool publish_transition = true;
rc += rclc_lifecycle_change_state(
  &my_lifecycle_node,
  lifecycle_msgs__msg__Transition__TRANSITION_CONFIGURE,
  publish_transition);

rc += rclc_lifecycle_change_state(
  &my_lifecycle_node,
  lifecycle_msgs__msg__Transition__TRANSITION_ACTIVATE,
  publish_transition);
```

除了错误处理转换外，转换通常从外部触发，例如通过 ROS 2 服务。

### 清理

要清理一切，只需执行

```c
rc += rcl_lifecycle_node_fini(&my_lifecycle_node, &allocator);
```

### 限制

生命周期服务尚无法通过 ros2 生命周期客户端调用（`ros2 lifecycle set /node ...`）。请改用 ros2 service CLI，（示例：`ros2 service call /node/change_state lifecycle_msgs/ChangeState "{transition: {id: 1, label: configure}}"`）。
