---
title: 执行管理
permalink: /docs/concepts/client_library/execution_management/
redirect_from:
  - /real-time_executor/
  - /docs/concepts/client_library/real-time_executor/
---


## 目录

*   [简介](#introduction)

*   [rclcpp 标准执行器分析](#analysis-of-rclcpp-standard-executor)
    * [架构](#architecture)
    * [调度语义](#scheduling-semantics)

*   [处理模式分析](#analysis-of-processing-patterns)
    * [机器人技术中的感知-规划-动作流水线](#sense-plan-act-pipeline-in-robotics)
    * [多速率同步](#synchronization-of-multiple-rates)
    * [高优先级处理路径](#high-priority-processing-path)
    * [实时嵌入式应用](#real-time-embedded-applications)
*   [rclc 执行器](#rclc-executor)
    * [特性](#features)
      * [触发条件](#trigger-condition)
      * [顺序执行](#sequential-execution)
      * [LET 语义](#let-semantics)
      * [多线程和调度配置](#multi-threading-and-scheduling-configuration)
    * [执行器 API](#executor-api)
      * [配置阶段](#configuration-phase)
      * [运行阶段](#running-phase)
    * [示例](#examples)
      * [机器人感知-规划-动作流水线示例](#sense-plan-act-pipeline-in-robotics-example)
      * [多速率同步示例](#synchronization-of-multiple-rates-example)
      * [高优先级处理路径示例](#high-priority-processing-path-example)
      * [实时嵌入式应用示例](#real-time-embedded-applications-example)
      * [ROS 2 执行器研讨会参考系统](#ros-2-executor-workshop-reference-system)
    * [未来工作](#future-work)
    * [下载](#download)

*   [回调组级执行器](#callback-group-level-executor)
    *   [API 变更](#api-changes)
    *   [测试平台](#test-bench)

*   [相关工作](#related-work)
    * [Fawkes 框架](#fawkes-framework)
*   [参考文献](#references)
*   [致谢](#acknowledgments)


## 简介

在给定的实时约束下实现可预测的执行是许多机器人应用的关键要求。虽然基于服务的 ROS 范式允许快速集成许多不同的功能，但它对执行管理的控制不足。例如，没有机制来强制执行节点内回调的特定执行顺序。多个节点执行顺序对于移动机器人中的控制应用也至关重要。包含传感器采集、数据评估和驱动控制的因果链应该按此顺序映射到 ROS 节点执行，然而没有明确的机制来强制执行这些顺序。此外，当回放现场测试中以 ROS-bag 形式收集的数据时，由于进程调度的非确定性，结果往往出奇地不同。

手动设置订阅和发布主题的特定执行顺序，以及定义相应 Linux 进程的特定用例优先级始终是可能的。然而，这种方法容易出错，难以扩展，并且需要深入了解系统中部署的 ROS 2 包。

因此，micro-ROS 中执行器的目标是帮助机器人技术专家使用实用且易于使用的实时机制，提供以下解决方案：
- 确定性执行
- 实时保证
- 在一个平台上集成实时和非实时功能
- 对 RTOS 和微控制器的专门支持

在 ROS 1 中，网络线程负责接收所有消息并将它们放入 FIFO 队列（在 roscpp 中）。也就是说，所有回调都以 FIFO 方式调用，没有任何执行管理。随着 ROS 2 中 DDS（数据分发服务）的引入，消息被缓冲在 DDS 中。在 ROS 2 中，引入了执行器概念来支持执行管理。在 rcl 层，配置了一个 _wait-set_，其中包含要接收的句柄，然后在第二步从 DDS 队列中获取这些句柄。句柄是 rcl 层为定时器、订阅、服务、客户端和守护条件定义的通用术语。

然而，ROS 2 执行器的标准 C++ API 实现（rclcpp）具有某些不寻常的特性，例如定时器优先于所有其他 DDS 句柄、非定时器句柄的非抢占式轮询调度，以及仅考虑每个句柄的一个输入数据（即使可能有多个可用）。这些特性的结果是，在某些情况下，标准 rclcpp 执行器不是确定性的，并且使其难以保证实时要求 [[CB2019](#CB2019)]。我们没有研究 Python 前端（rclpy）的 ROS 2 执行器实现，因为我们认为在微控制器平台上，通常会运行 C 或 C++ 应用程序。

鉴于实时执行器的目标和 ROS 2 标准 rclcpp 执行器的局限性，挑战在于：
- 为 ROS 2 框架和实时操作系统（RTOS）开发适当的、定义明确的调度机制
- 为 ROS 开发人员定义易于使用的接口
- 对需求进行建模（如延迟、子系统中的确定性）
- ROS 2 框架和操作系统调度器的映射（半自动和优化的映射以及通用的、众所周知的框架机制也是可取的）

我们的方法是为 rcl+rclc 层（如 [客户端库简介](../) 中所述）提供一个支持实时功能的 C 语言执行器。

作为第一步，我们为 C 编程语言中的 rcl 层提出了 rclc 执行器，它具有支持实时和确定性执行的新特性：它支持 1.）用户定义的静态顺序执行，2.）条件执行语义，3.）具有调度配置的多线程执行，以及 4.）逻辑执行时间（LET）语义。顺序执行指的是运行时行为，即所有回调都按预定义顺序执行，与消息到达时间无关。可通过触发条件获得条件执行，该触发条件支持机器人技术中的典型处理模式（在 [处理模式分析](#analysis-of-processing-patterns) 部分中有详细分析）。多线程应用程序的调度参数配置实现优先级执行。逻辑执行时间概念（LET）为嵌入式应用的固定周期性任务调度提供数据同步。

除了 micro-ROS 的高级执行管理机制外，我们还为标准 ROS 2 中的 rclcpp 执行器概念做出了贡献：回调组级执行器。它不是一个新的执行器，而是对 ROS 2 执行器 API 的改进，允许对回调组进行优先级排序，而这在当前 Iron 版本中的 ROS 2 默认执行器中是不可能的。

## rclcpp 标准执行器分析

ROS 2 允许将多个节点捆绑在一个操作系统进程中。rclcpp（以及 rclpy）中引入了执行器概念来协调进程中节点回调的执行。

ROS 2 设计为每个进程定义一个执行器（[rclcpp::executor::Executor](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executor.hpp) 的实例），它通常在自定义 main 函数或启动系统中创建。执行器通过检查 DDS 队列中是否有可用的工作（定时器、服务、消息、订阅等）并将它们分派到一个或多个线程来协调这些节点发出的所有回调的实现，分别在 [SingleThreadedExecutor](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executors/single_threaded_executor.hpp) 和 [MultiThreadedExecutor](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executors/multi_threaded_executor.hpp) 中实现。

调度机制类似于 ROS 1 的自旋线程行为：执行器查找等待集，等待集通知它 DDS 队列中有任何待处理的回调。如果有多个待处理的回调，ROS 2 执行器按在执行器注册时的顺序执行它们。

### 架构

下图描述了标准 ROS 2 执行器实现的相关类：

<center>
<img src="png/executor_class_diagram.png" alt="ROS 2 Executor class diagram" width="100%" />
</center>

请注意，执行器实例仅维护指向节点 NodeBaseInterfaces 的弱指针。因此，可以安全地销毁节点，而无需通知执行器。

此外，执行器不维护显式的回调队列，而是依赖于底层 DDS 实现的队列机制，如以下序列图所示：

<center>
<img src="png/executor_to_dds_sequence_diagram.png" alt="Call sequence from executor to DDS" width="100%" />
</center>

然而，执行器概念没有提供对传入回调调用进行优先级排序或分类的手段。此外，它也没有利用底层操作系统调度器的实时特性来更好地控制执行顺序。这种行为的整体含义是，时间关键的回调可能会遭受可能的截止时间错失和性能下降，因为它们的服务晚于非关键回调。此外，由于 FIFO 机制，很难确定每个回调执行可能产生的最坏情况延迟的可使用界限。

### 调度语义

在最近的一篇论文 [[CB2019](#CB2019)] 中，rclcpp 执行器得到了详细分析，并在基于预留的调度下提出了因果链的响应时间分析。执行器区分四类回调：_定时器_，由系统级定时器触发；_订阅者_，由订阅主题上的新消息触发；_服务_，由服务请求触发；以及_客户端_，由服务请求的响应触发。执行器负责从 DDS 层的输入队列中取出消息并执行相应的回调。由于它执行回调直到完成，因此它是一个非抢占式调度器，但它不考虑所有准备好的任务，而只考虑一个称为 readySet 的快照。当执行器空闲时更新此 readySet，在这一步它与 DDS 层交互以更新准备好的任务集。然后，对于每种类型的任务，有专门的队列（定时器、订阅、服务、客户端）按顺序处理。指出了以下不良特性：

* 定时器具有最高优先级。执行器始终首先处理 _定时器_。这可能导致在过载情况下 DDS 队列中的消息无法处理的固有效果。
* 非定时器句柄的非抢占式轮询调度。在处理 readySet 期间到达的消息不会被考虑，直到下一次更新，这取决于所有剩余回调的执行时间。这会导致优先级反转，因为低优先级回调可能会通过延长 readySet 的当前处理来隐式阻塞更高优先级的回调。
* 每个句柄只考虑一条消息。readySet 只包含一个任务实例。例如，即使同一主题有多条消息可用，也只会处理一个实例，直到执行器再次空闲并从 DDS 层更新 readySet。这会加剧优先级反转，因为积压的回调可能需要多次处理 readySet 才能被考虑进行调度。这意味着非定时器回调实例可能被同一低优先级回调的多个实例阻塞。

基于这些发现，作者提出了一种替代方法来提供确定性，并将众所周知的可调度性分析应用于 ROS 2 系统。在基于预留的调度下描述了响应时间分析。

## 处理模式分析

为 micro-ROS 开发执行管理机制的基础是对机器人和嵌入式系统中常用的处理模式进行分析。首先，介绍移动机器人中用于实现确定性行为的典型处理模式。然后，分析实时嵌入式系统中的处理模式，主要区别在于应用时间触发范式来实现实时行为。

### 机器人技术中的感知-规划-动作流水线

现在我们描述在移动机器人中用于实现确定性行为的常见软件设计模式。对于每种设计模式，我们描述概念以及为确定性执行器派生的要求。

**概念：**

移动机器人中常见的控制范式是一个控制循环，由几个阶段组成：一个感测阶段获取传感器数据，一个规划阶段进行定位和路径规划，以及一个驱动阶段来控制移动机器人。当然，可能有更多的阶段，这里这三个阶段作为示例。这样的处理流水线如图1所示。

<center>
<img src="png/sensePlanActScheme.png" alt="Sense Plan Act Pipeline" width="60%"/>
</center>
<center>
图1：多个传感器驱动的感知-规划-动作流水线。
</center>

通常使用多个传感器来感知环境。例如 IMU 和激光扫描仪。定位算法的质量很大程度上取决于处理这些传感器数据时的数据"年龄"。理想情况下，应该处理所有传感器的最新数据。实现这一目标的一种方法是在感知阶段首先执行所有传感器驱动，然后在规划阶段处理所有算法。

目前，无法使用 rclcpp 的默认执行器定义这样的处理顺序。原则上可以设计数据驱动的流水线，但是如果激光扫描需要在感知阶段以及规划阶段的某些其他回调中使用，这些订阅的处理顺序是任意的。

对于这种感知-规划-动作模式，我们可以为每个阶段定义一个执行器。规划阶段仅在感知阶段的所有回调完成后才被触发。

**派生要求：**
- 触发执行

### 多速率同步

**概念：**

通常，在移动机器人中会使用多个传感器来感知环境。虽然 IMU 传感器以非常高的速率（例如 500 Hz）提供数据样本，但激光扫描可用的频率要慢得多（例如 10 Hz），由旋转时间决定。那么，挑战在于如何确定性地融合不同频率的传感器数据。此问题如图 2 所示。

<center>
<img src="png/sensorFusion_01.png" alt="Sychronization of multiple rates" width="30%" />
</center>
<center>
图2：如何确定性地处理多频率传感器数据。
</center>

由于调度效应，评估激光扫描的回调可能在 IMU 数据到达之前或之后被调用。解决这个问题的一种方法是在应用程序内编写额外的同步代码。显然，这是一个繁琐且不可移植的解决方案。

另一种方法是评估 IMU 样本和激光扫描，通过同步它们的频率。例如，始终处理 50 个 IMU 样本与一次激光扫描。如图 3 所示这种方法。预处理回调聚合 IMU 样本，并以 10 Hz 的速率发送包含 50 个样本的聚合消息。现在两条消息具有相同的频率。通过一个触发条件（当两条消息都可用时触发），传感器融合算法可以期望始终同步的输入数据。

<center>
<img src="png/sensorFusion_02.png" alt="Sychnronization with a trigger" width="40%" />
</center>
<center>
图3：使用触发器同步多个输入数据。
</center>

在 ROS 2 中，由于 rclcpp 和 rclpy 的执行器缺乏触发概念，目前无法对此进行建模。消息过滤器可用于根据标题中的时间戳同步输入数据，但这仅在 rclcpp 中可用（而不是在 rcl 中）。此外，如果在执行器中直接有这样一个触发概念会更有效。

另一个想法是仅在收到激光扫描时主动请求 IMU 数据。这个概念如图 4 所示。收到激光扫描消息后，首先请求包含聚合 IMU 样本的消息。然后处理激光扫描，稍后处理传感器融合算法。支持回调顺序执行执行器可以实现这个想法。

<center>
<img src="png/sensorFusion_03.png" alt="Sychronization with sequence" width="30%" />
</center>
<center>
图4：顺序处理同步。
</center>

**派生要求：**
- 触发执行
- 回调的顺序处理

### 高优先级处理路径
**概念**
通常机器人必须同时完成多项活动。例如跟随路径和避障。跟随路径是一项永久性活动，而避障由环境触发，应该立即做出反应。因此，人们希望为活动指定优先级。如图 5 所示：

<center>
<img src="png/highPriorityPath.png" alt="HighPriorityPath" width="50%" />
</center>
<center>
图5：使用顺序顺序管理高优先级路径。
</center>

假设一个简化的控制循环包含感知-规划-动作活动，可能会暂时停止机器人的避障应该在规划阶段之前处理。在这个例子中，我们假设这些活动在一个线程中处理。

**派生要求：**
- 回调的顺序处理


### 实时嵌入式应用

在嵌入式系统中，实时行为是通过使用时间触发范式来实现的，这意味着进程被周期性激活。进程可以分配优先级以允许抢占。图 6 显示了一个示例，其中显示了三个具有固定周期的进程。中层和下层进程被多次抢占，用空心虚线框表示。

<center>
<img src="png/scheduling_01.png" alt="Schedule with fixed periods" width="30%"/>
</center>
<center>
图6：固定周期抢占式调度。
</center>

每个进程可以分配一个或多个任务，如图 7 所示。这些任务顺序执行，这通常称为协作调度。

<center>
<img src="png/scheduling_02.png" alt="Schedule with fixed periods" width="30%"/>
</center>
<center>
图7：顺序执行任务的进程。
</center>

虽然有多种方法可以为给定数量的进程分配优先级，
但是已证明速率单调调度分配（周期较短的进程具有较高优先级）在处理器利用率低于 69% 时是最优的 [[LL1973](#LL1973)]。

在过去的几十年中提出了许多不同的调度方法；然而，固定周期抢占式调度仍在嵌入式实时系统中广泛使用 [[KZH2015](#KZH2015)]。这在查看当前操作系统的特性时也很明显。像 Linux 一样，实时操作系统，如 NuttX、Zephyr、FreeRTOS、QNX 等，支持固定周期抢占式调度和优先级分配，这使得时间触发范式成为该领域的主导设计原则。

However, data consistency is often an issue when preemptive scheduling is used and if data is being shared across multiple processes via global variables. Due to scheduling effects and varying execution times of processes, writing and reading these variables could occur sometimes sooner or later. This results in a latency jitter of update times (the timepoint at which a variable change becomes visible to other processes). Race conditions can occur when multiple processes access a variable at the same time. So to solve this problem, the concept of logical-execution time (LET) was introduced in [[HHK2001](#HHK2001)], in which communication of data occurs only at pre-defined periodic time instances: Reading data only at the beginning of the period and writing data only at the end of the period. The cost of an additional latency delay is traded for data consistency and reduced jitter. This concept has also recently been applied to automotive applications[[NSP2018](#NSP2018)].

<center>
<img src="png/scheduling_LET.png" alt="Schedule with fixed periods" width="80%"/>
</center>
<center>
Figure 8: Data communication without and with Logical Execution Time paradigm.
</center>

An Example of the LET concept is shown in Figure 8. Assume that two processes are communicating data via one global variable. The timepoint when this data is written is at the end of the processing time. In the default case (left side), the processes p<sub>3</sub> and p<sub>4</sub> receive the update. At the right side of Figure 8, the same scenario is shown with LET semantics. Here, the data is communicated only at period boundaries. In this case, the lower process communicates at the end of the period, so that always processes p<sub>3</sub> and p<sub>5</sub> receive the new data.

**Concept:**
- periodic execution of processes
- assignment of fixed priorities to processes
- preemptive scheduling of processes
- co-operative scheduling of tasks within a process (sequential execution)
- data synchronization with LET-semantics


While periodic activation is possible in ROS 2 by using timers, preemptive scheduling is supported by the operating system and assigning priorities on the granularity of threads/processes that correspond to the ROS nodes; it is not possible to sequentially execute callbacks, which have no data-dependency. Furthermore data is read from the DDS queue just before the callback is executed and data is written sometime during the time the application is executed. While the `spin_period` function of the rclcpp Executor allows to check for data at a fixed period and executing those callbacks for which data is available, however, with this spin-function does not execute all callbacks irrespective wheter data is available or not. So `spin_period` is not helpful to periodically execute a number of callbacks (aka tasks within a process). So we need a mechanism that triggers the execution of multiple callbacks (aka tasks) based on a timer. Data transmission is achieved via DDS which does not allow to implement a LET-semantics. To summarize, we derive the following requirements.

**Derived requirements:**
- trigger the execution
- sequential processing of callbacks
- data synchronization with LET semantics

## rclc 执行器

The rclc Executor is a ROS 2 Executor based on the rcl-layer in C programming language. As discussed above, the default rclcpp Executor is not suitable to implement real-time applications because of three main reasons: timers are preferred over all other handles, no priorization of callback execution and the round-robin to completion execution of callbacks. On the other hand, several processing patterns have been developed as best practices to achieve non-functional requirements, such as bounded end-to-end latencies, low jitter of response times of cause-effect chains, deterministic processing and short response times even in overload situations. These processing patterns are difficult to implement with the concepts availabe in the default ROS 2 Executor, therefore we have developed a flexible Executor: the rclc Executor. 

### 特性

The rclc Executor is feature-complete, i.e. it supports all event types as the default ROS 2 Executor, which are:
- subscriptions
- timers
- services
- clients
- guard conditions
- actions
- lifecycle

The flexible rclc Executor provides on top the following new features:
- triggered execution
- user-defined sequential execution
- multi-threading and scheduling configuration (WIP)
- LET-semantics for data synchronization of periodic process scheduling

First, a *trigger condition* allows to define when the processing of a callback shall start. This is useful to implement sense-plan-act control loops or more complex processing structures with directed acyclic graphs. Second, a user can specify the *processing order* in which these callbacks will be executed. With this feature, the pattern of sensor fusion with multiple rates, in which data is requested from a sensor based on the arrival of some other sensor, can be easily implemented. Third, the assignment of scheduling parameters (e.g., priorities) of the underlying operating system. With this feature, prioritized processing can be implemented. Finally, for periodic applications, the *LET Semantics* has been implemented to support data consistency for periodic process scheduling. These features are now described in more detail.

#### 顺序执行

- At configuration, the user defines the order of handles.
- At configuration, the user defines whether the handle shall be called only when new data is available (ON_NEW_DATA) or whether the callback shall always be called (ALWAYS).
- At runtime, all handles are processed in the user-defined order
  - if the configuration of handle is ON_NEW_DATA, then the corresponding callback is only called if new data is available
  - if the configuration of the handle is ALWAYS, then the corresponding callback is always. If no data is available, then the callback is called with no data (e.g. NULL pointer).

Figure 9 shows three callbacks, A, B and C. Assume, they shall be executed in the order *B,A,C*. Then the user adds the callbacks to the rclc Executor in this order. Whenever new messages have arrived then the callbacks for which a new message is availabe will be always executed in the user-defined processing order. 
<center>
<img src="png/rclc_executor_sequential_execution.png" alt="Sequential execution semantics" width="50%" />
</center>
<center>
Figure 9: Sequential execution semantics.
</center>

#### 触发条件

- Given a set of handles, a trigger condition, which is based on the availability of input data of these handles, decides when the processing of all callbacks starts. This is shown in Figure 10. 

<center>
<img src="png/trigger_01.png" alt="Trigger condition overview" width="50%" />
</center>
<center>
Figure 10: Executor with trigger condition
</center>

- Available options:
  - ALL operation: fires when input data is available for all handles
  - ANY operation: fires when input data is available for at least one handle (OR semantics)
  - ONE: fires when input data for a user-specified handle is available
  - User-defined function: user can implement custom logic

Figure 11 shows an example of the ALL semantics. Only if all messages *msg_A, msg_B, msg_C* were received, then trigger condition is fullfilled and the callbacks are processed in a user-defined order.
<center>
<img src="png/trigger_ALL.png" alt="Trigger condition ALL" width="30%" />
</center>
<center>
Figure 11: Trigger condition ALL
</center>

Figure 12 shows an example of the ANY semantics. Thas is, if any messages *msg_A, msg_B, msg_C* was received, then trigger condition is fullfilled and the callbacks are processed in a user-defined order. This is equivalent to OR semantics.
<center>
<img src="png/trigger_OR.png" alt="Trigger condition ANY" width="30%" />
</center>
<center>
Figure 12: Trigger condition ANY (OR)
</center>

Figure 13 shows an example of the ONE semantics. Thas is, only if message *msg_B* was received, the trigger condition is fullfilled and (potentially all) callbacks are processed in a user-defined order.
<center>
<img src="png/trigger_ONE.png" alt="Trigger condition ONE" width="30%" />
</center>
<center>
Figure 13: Trigger condition ONE
</center>

Figure 14 describes the custom semantics. A custom trigger condition with could be a more complex logic of multiple messages, can be passed to the executor. This might also include hardware triggers, like interrupts. 
<center>
<img src="png/trigger_user_defined.png" alt="Trigger condition user-defined" width="30%" />
</center>
<center>
Figure 14: Trigger condition user-defined
</center>

#### LET 语义
- Assumption: time-triggered system, the executor is activated periodically
- When the trigger fires, reads all input data and makes a local copy
- Processes all callbacks in sequential order
- Write output data at the end of the executor's period (Note: this is not implemented yet)

Additionally we have implemented the current rclcpp Executor semantics ("RCLCPP"):
- waiting for new data for all handles (rcl_wait)
- using trigger condition ANY
- if trigger fires, start processing handles in pre-defined sequential order
- request from DDS-queue the new data just before the handle is executed (rcl_take)

The selection of the Executor semantics is optional. The default semantics is "RCLCPP".

#### 多线程和调度配置

The rclc Executor has been extended for multi-threading. It supports the assignment of scheduling policies, like priorities or more advanced scheduling algorithms as reservation-based scheduling, to subscription callbacks. [[Pull Request](https://github.com/ros2/rclc/pull/87), Pre-print [SLD2021](#SLD2021)]. The overall architecture is shown in Figure 15. One Executor thread is responsible for checking for new data from the DDS queue. For every callback, a thread is spawned with the dedicted scheduling policy provided by the operating system. The Executor then dispatches new data of a subscription to its corresponding callback function, which is then executed in its own thread by operating system.

<center>
<img src="png/rclc_executor_multi_threaded.png" alt="Multi-threaded rclc Executor" width="90%" />
</center>
<center>
Figure 15: multi-threaded rclc-Executor
</center>

### 执行器 API
The API of the rclc Executor can be divided in two phases: Configuration and Running.
#### 配置阶段
During the configuration phase, the user shall define:
- the total number of callbacks
- the sequence of the callbacks
- trigger condition (optional, default: ANY)
- data communcation semantics (optional, default ROS2)

As the Executor is intended for embedded controllers, dynamic memory management is crucial. Therefore at initialization of the rclc Executor, the user defines the total number of callbacks. The necessary dynamic memory will be allocated only in this phase and no more memory in the running phase. This makes this Executor static in the sense, that during runtime no additional callbacks can be added.

Then, the user adds handles and the corresponding callbacks (e.g. for subscriptions and timers) to the Executor. The order in which this takes place, defines later the sequential processing order during runtime.

For each handle the user can specify, if the callback shall be executed only if new data is available (ON_NEW_DATA) or if the callback shall always be executed (ALWAYS). The second option is useful when the callback is expected to be called at a fixed rate.

The trigger condition defines when the processing of these callbacks shall start. For convenience some default conditions have been defined:
- trigger_any(default) : start executing if any callback has new data
- trigger_all : start executing if all callbacks have new data
- trigger_one(&`data`) : start executing if `data` has been received
- user_defined_function: the user can also define its own function with more complex logic

With 'trigger_any' being the default, the current semantics of the rclcpp Executor is selected.

The data communication semantics can be
- ROS2 (default)
- LET

To be compatible with ROS2 rclcpp Executor, the existing rclcpp semantics is implemented as 'ROS2'. That is, with the spin-function the DDS-queue is constantly monitored for new data (rcl_wait). If new data becomes available, then is fetched from DDS (rcl_take) immediately before the callback is executed. All callbacks are processed in the user-defined order, this is the only difference to the rclcpp Executor, in which no order can be specified.

Secondly, the LET semantics is implemented such that at the beginning of processing all available data is fetched (rcl_take) and buffered and then the callbacks are processed in the pre-defined operating on the buffered copy.

#### 运行阶段

As the main functionality, the Executor has a `spin`-function which constantly checks for new data at the DDS-queue, like the rclcpp Executor in ROS2. If the trigger condition is satisfied then all available data from the DDS queue is processed according to the specified semantics (ROS or LET) in the user-defined sequential order. After all callbacks have been processed the DDS is checked for new data again.

Available spin functions are
- `spin_some`  - spin one time
- `spin_period` - spin with a period
- `spin` - spin indefinitly

### 示例
We provide the relevant code snippets how to setup the rclc Executor for the processing patterns as described above.

#### 机器人技术中的感知-规划-动作流水线示例

In this example we want to realise a sense-plan-act pipeline in a single thread. The trigger condition is demonstrated by activating
the sense-phase when both data for the Laser and IMU are available. Three executors are necessary `exe_sense`, `exe_plan` and `exe_act`. The two sensor acquisition callbacks `sense_Laser` and `sense_IMU` are registered in the Executor `exe_sense`.
The trigger condition ALL is responsible to activate the sense-phase only when all data for these two callbacks are available. Finally all three Executors are spinning using a `while`-loop and the `spin_some` function.

The definitions of callbacks are omitted.

```C
...
rcl_subscription_t sense_Laser, sense_IMU, plan, act;
rcle_let_executor_t exe_sense, exe_plan, exe_act;
// initialize executors
rclc_executor_init(&exe_sense, &context, 2, ...);
rclc_executor_init(&exe_plan, &context, 1, ...);
rclc_executor_init(&exe_act, &context, 1, ...);
// executor for sense-phase
rclc_executor_add_subscription(&exe_sense, &sense_Laser, &my_sub_cb1, ON_NEW_DATA);
rclc_executor_add_subscription(&exe_sense, &sense_IMU, &my_sub_cb2, ON_NEW_DATA);
rclc_let_executor_set_trigger(&exe_sense, rclc_executor_trigger_all, NULL);
// executor for plan-phase
rclc_executor_add_subscription(&exe_plan, &plan, &my_sub_cb3, ON_NEW_DATA);
// executor for act-phase
rclc_executor_add_subscription(&exe_act, &act, &my_sub_cb4, ON_NEW_DATA);

// spin all executors
while (true) {
  rclc_executor_spin_some(&exe_sense, RCL_MS_TO_NS(100));
  rclc_executor_spin_some(&exe_plan, RCL_MS_TO_NS(100));
  rclc_executor_spin_some(&exe_act, RCL_MS_TO_NS(100));
}
```

#### 多速率同步示例

The sensor fusion synchronizing the multiple rates with a trigger is shown below.

```C
...
rcl_subscription_t aggr_IMU, sense_Laser, sense_IMU;
rcle_let_executor_t exe_aggr, exe_sense;
// initialize executors
rclc_executor_init(&exe_aggr, &context, 1, ...);
rclc_executor_init(&exe_sense, &context, 2, ...);
// executor for aggregate IMU data
rclc_executor_add_subscription(&exe_aggr, &aggr_IMU, &my_sub_cb1, ON_NEW_DATA);
// executor for sense-phase
rclc_executor_add_subscription(&exe_sense, &sense_Laser, &my_sub_cb2, ON_NEW_DATA);
rclc_executor_add_subscription(&exe_sense, &sense_IMU, &my_sub_cb3, ON_NEW_DATA);
rclc_executor_set_trigger(&exe_sense, rclc_executor_trigger_all, NULL);

// spin all executors
while (true) {
  rclc_executor_spin_some(&exe_aggr, RCL_MS_TO_NS(100));
  rclc_executor_spin_some(&exe_sense, RCL_MS_TO_NS(100));
}
```

The setup for the sensor fusion using sequential execution is shown below.
Note that the sequetial order is `sense_IMU`, which will request the aggregated IMU message, and then `sense_Laser`
while the trigger will fire, when a laser message is received.

```C
...
rcl_subscription_t sense_Laser, sense_IMU;
rcle_let_executor_t exe_sense;
// initialize executor
rclc_executor_init(&exe_sense, &context, 2, ...);
// executor for sense-phase
rclc_executor_add_subscription(&exe_sense, &sense_IMU, &my_sub_cb1, ALWAYS);
rclc_executor_add_subscription(&exe_sense, &sense_Laser, &my_sub_cb2, ON_NEW_DATA);
rclc_executor_set_trigger(&exe_sense, rclc_executor_trigger_one, &sense_Laser);
// spin
rclc_executor_spin(&exe_sense);
```
#### 高优先级处理路径示例

This example shows the sequential processing order to execute the obstacle avoidance `obst_avoid`
after the callbacks of the sense-phase and before the callback of the planning phase `plan`.
The control loop is started when a laser message is received. Then an aggregated IMU message is requested,
like in the example above. Then all the other callbacks are always executed. This assumes that these callbacks
communicate via a global data structure. Race conditions cannot occur, because the callbacks
run all in one thread.

```C
...
rcl_subscription_t sense_Laser, sense_IMU, plan, act, obst_avoid;
rcle_let_executor_t exe;
// initialize executors
rclc_executor_init(&exe, &context, 5, ...);
// define processing order
rclc_executor_add_subscription(&exe, &sense_IMU, &my_sub_cb1, ALWAYS);
rclc_executor_add_subscription(&exe, &sense_Laser, &my_sub_cb2, ON_NEW_DATA);
rclc_executor_add_subscription(&exe, &obst_avoid, &my_sub_cb3, ALWAYS);
rclc_executor_add_subscription(&exe, &plan, &my_sub_cb4, ALWAYS);
rclc_executor_add_subscription(&exe, &act, &my_sub_cb5, ALWAYS);
rclc_executor_set_trigger(&exe, rclc_executor_trigger_one, &sense_Laser);
// spin
rclc_executor_spin(&exe);
```

#### 实时嵌入式应用示例

With sequential execution, the co-operative scheduling of tasks within a process can be modeled. The trigger condition is used to periodically activate the process which will then execute all callbacks in a pre-defined order. Data will be communicated using the LET-semantics. Every Executor is executed in its own thread, to which an appropriate priority can be assigned.

In the following example, the Executor is setup with 4 handles. We assume a process has three subscriptions `sub1`, `sub2`, `sub3`. The sequential processing order is given by the order as they are added to the Executor. A timer `timer` defines the period.  The `trigger_one` with the parameter `timer` is used, so that whenever the timer is ready, all callbacks are processed. Finally the data communication semantics LET is defined.
```C
#include "rcl_executor/let_executor.h"

// define subscription callback
void my_sub_cb1(const void * msgin)
{
  // ...
}
// define subscription callback
void my_sub_cb2(const void * msgin)
{
  // ...
}
// define subscription callback
void my_sub_cb3(const void * msgin)
{
  // ...
}

// define timer callback
void my_timer_cb(rcl_timer_t * timer, int64_t last_call_time)
{
  // ...
}

// necessary ROS 2 objects
rcl_context_t context;   
rcl_node_t node;
rcl_subscription_t sub1, sub2, sub3;
rcl_timer_t timer;
rcle_let_executor_t exe;

// define ROS context
context = rcl_get_zero_initialized_context();
// initialize ROS node
rcl_node_init(&node, &context,...);
// create subscriptions
rcl_subscription_init(&sub1, &node, ...);
rcl_subscription_init(&sub2, &node, ...);
rcl_subscription_init(&sub3, &node, ...);
// create a timer
rcl_timer_init(&timer, &my_timer_cb, ... );
// initialize executor with four handles
rclc_executor_init(&exe, &context, 4, ...);
// define static execution order of handles
rclc_executor_add_subscription(&exe, &sub1, &my_sub_cb1, ALWAYS);
rclc_executor_add_subscription(&exe, &sub2, &my_sub_cb2, ALWAYS);
rclc_executor_add_subscription(&exe, &sub3, &my_sub_cb3, ALWAYS);
rclc_executor_add_timer(&exe, &timer);
// trigger when handle 'timer' is ready
rclc_executor_set_trigger(&exe, rclc_executor_trigger_one, &timer);
// select LET-semantics
rclc_executor_data_comm_semantics(&exe, LET);
// spin forever
rclc_executor_spin(&exe);
```

#### ROS 2 执行器研讨会参考系统
The rclc Executor has been presented at the workshop 'ROS 2 Executor: How to make it efficient, real-time and deterministic?' at [ROS World 2021](https://roscon.ros.org/world/2021/) (i.e. the online version of ROSCon)[[S2021](#S2021)]. A [Reference System](https://github.com/ros-realtime/reference-system) for testing and benchmarking ROS Executors has been developed for this workshop. The application of the rclc Executor on the reference system with the trigger condition can be found in the [rclc-executor branch of the Reference System](https://github.com/ros-realtime/reference-system/tree/rclc_executor). 

<iframe width="560" height="315" src="https://www.youtube.com/embed/IazrPF3RN1U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The slides can be downloaded [here](https://ec2a4d36-bac8-4759-b25e-bb1f794177f4.filesusr.com/ugd/984e93_749e27b917a54b45b9ccb5be930841b8.pdf). All information and the videos and slides of the other talks of the workshop can be found at [www.apex.ai/roscon-21](https://www.apex.ai/roscon-21).

### 未来工作

- Full LET semantics (writing data at the end of the period)
  - one publisher that periodically publishes
  - if Executors are running in multiple threads,
    publishing needs to be atomic
- Multi-threaded executor with assignment of scheduling policies of underlying operating system. [[Pull Request](https://github.com/ros2/rclc/pull/87), pre-print [SLD2021](#SLD2021)].

### 下载
The rclc Executor can be downloaded from the [ros2/rclc repository](https://github.com/ros2/rclc). It is available for the ROS 2 versions Humble, Iron and Rolling. The repository provides several packages including the [rclc Executor](https://github.com/ros2/rclc/tree/master/rclc) and an [rclc_examples package](https://github.com/ros2/rclc/tree/master/rclc_examples) with several application examples.

## 回调组级执行器

The Callback-group-level Executor was an early prototype for a refined rclcpp Executor API developed in micro-ROS. It has been derived from the default rclcpp Executor and addresses some of the aforementioned deficits. Most important, it was used to validate that the underlying layers (rcl, rmw, rmw_adapter, DDS) allow for multiple Executor instances without any negative interferences.

As the default rclcpp Executor works at a node-level granularity – which is a limitation given that a node may issue different callbacks needing different real-time guarantees - we decided to refine the API for more fine-grained control over the scheduling of callbacks on the granularity of callback groups using. We leverage the callback-group concept existing in rclcpp by introducing real-time profiles such as RT-CRITICAL and BEST-EFFORT in the callback-group API (i.e. rclcpp/callback_group.hpp). Each callback needing specific real-time guarantees, when created, may therefore be associated with a dedicated callback group. With this in place, we enhanced the Executor and depending classes (e.g., for memory allocation) to operate at a finer callback-group granularity. This allows a single node to have callbacks with different real-time profiles assigned to different Executor instances - within one process.

Thus, an Executor instance can be dedicated to specific callback group(s) and the Executor’s thread(s) can be prioritized according to the real-time requirements of these groups. For example, all time-critical callbacks are handled by an "RT-CRITICAL" Executor instance running at the highest scheduler priority.

The following figure illustrates this approach with two nodes served by three Callback-group-level Executors in one process:

<center>
<img src="png/cbg-executor_sample_system.png" alt="Sample system with two nodes and three Callback-group-level Executors in one process" width="60%" />
</center>

The different callbacks of the Drive-Base node are distributed to different Executors (visualized by the color red, yellow and green).  For example the onCmdVel and publishWheelTicks callback are scheduled by the same Executor (yellow). Callbacks from different nodes can be serviced by the same Executor.

### API 更改

In this section, we describe the necessary changes to the Executor API:
*   [include/rclcpp/callback\_group.hpp](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/callback_group.hpp):

    * Introduced an enum to distinguish up to three real-time classes (requirements) per node (RealTimeCritical, SoftRealTime, BestEffort)
    * Changed association with Executor instance from nodes to callback groups.
*   [include/rclcpp/executor.hpp](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/executor.hpp)

    * Added functions to add and remove individual callback groups in addition to whole nodes.

    * Replaced private vector of nodes with a map from callback groups to nodes.

*   [include/rclcpp/memory\_strategy.hpp](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/memory_strategy.hpp)

    * Changed all functions that expect a vector of nodes to the just mentioned map.
*   [include/rclcpp/node.hpp](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/node.hpp) and [include/rclcpp/node_interfaces/node_base.hpp](https://github.com/ros2/rclcpp/blob/master/rclcpp/include/rclcpp/node_interfaces/node_base.hpp)

    * Extended arguments of create\_callback\_group function for the real-time class.
    * Removed the get\_associated\_with\_executor\_atomic function.

The callback-group-level executor has been merged into ROS 2 rclcpp in [pull request 1218](https://github.com/ros2/rclcpp/pull/1218/commits).

### 测试台

As a proof of concept, we implemented a small test bench in the present package cbg-executor_ping-pong_cpp. The test bench comprises a Ping node and a Pong node which exchange real-time and best-effort messages simultaneously with each other. Each class of messages is handled with a dedicated Executor, as illustrated in the following figure.

<center>
<img src="png/ping_pong_diagram.png" alt="Architecture for the Callback-group-level Executor test bench" width="100%" />
</center>

With the test bench, we validated the functioning of the approach.

<center>
<img src="png/cbg_executor_demo_plot.png" alt="Results from Callback-group-level Executor test bench" width="80%" />
</center>

In this example, the callback for the high priority task (red line) consumes 10ms and the low priority task (blue line) 40ms in the Pong Node. With a ping rate of 20 Hz, the CPU saturates (10ms\*20+40ms\*20=1000ms). With higher frequencies the high priorty task can continue to send its pong message, while the low priority pong task degrades. With a frequency of 100Hz the high priority task requires 100% CPU utilization. With higher ping rates it keeps sending pong messages with 100Hz, while the low priority task does not get any CPU resources any more and cannot send any messages.

The test bench is provided in the [cbg_executor_demo](https://github.com/ros2/examples/tree/master/rclcpp/executors/cbg_executor).

## 相关工作

In this section, we provide an overview to related approaches and link to the corresponding APIs.

### Fawkes 框架

[Fawkes](http://www.fawkesrobotics.org/) is a robotic software framework, which supports synchronization points for sense-plan-act like execution. It has been developed by RWTH Aachen since 2006. Source code is available at [github.com/fawkesrobotics](https://github.com/fawkesrobotics).

#### 同步
Fawkes provides developers different synchronization points, which are very useful for defining an execution order of a typical sense-plan-act application. These ten synchronization points (wake-up hooks) are the following (cf. [libs/aspect/blocked_timing.h](https://github.com/fawkesrobotics/fawkes/blob/master/src/libs/aspect/blocked_timing.h)):

*   WAKEUP\_HOOK\_PRE\_LOOP
*   WAKEUP\_HOOK\_SENSOR\_ACQUIRE
*   WAKEUP\_HOOK\_SENSOR\_PREPARE
*   WAKEUP\_HOOK\_SENSOR\_PROCESS
*   WAKEUP\_HOOK\_WORLDSTATE
*   WAKEUP\_HOOK\_THINK
*   WAKEUP\_HOOK\_SKILL   
*   WAKEUP\_HOOK\_ACT     
*   WAKEUP\_HOOK\_ACT\_EXEC
*   WAKEUP\_HOOK\_POST\_LOOP  

#### 编译时配置
At compile time, a desired synchronization point is defined as a constructor parameter for a module. For example, assuming that `mapLaserGenThread` shall be executed in SENSOR_ACQUIRE, the constructor is implemented as:

```C++
MapLaserGenThread::MapLaserGenThread()
  :: Thread("MapLaserGenThread", Thread::OPMODE_WAITFORWAKEUP),
     BlockedTimingAspect(BlockedTimingAspect::WAKEUP_HOOK_SENSOR_ACQUIRE),
     TransformAspect(TransformAspect::BOTH_DEFER_PUBLISHER, "Map Laser Odometry")
```

Similarly, if `NaoQiButtonThread` shall be executed in the SENSOR_PROCESS hook, the constructor is:

```C++
NaoQiButtonThread::NaoQiButtonThread()
  :: Thread("NaoQiButtonThread", Thread::OPMODE_WAITFORWAKEUP),
     BlockedTimingAspect(BlockedTimingAspect::WAKEUP_HOOK_SENSOR_PROCESS)
```

#### 运行时执行
At runtime, the *Executor* iterates through the list of synchronization points and executes all registered threads until completion. Then, the threads of the next synchronization point are called.

A module (thread) can be configured independent of these sense-plan-act synchronization points. This has the effect, that this thread is executed in parallel to this chain.

The high level overview of the Fawkes framework is shown in the next figure. At compile-time the configuration of the sense-plan act wakeup hook is done (upper part), while at run-time the scheduler iterates through this list of wakeup-hooks (lower part):

<center>
<img src="png/fawkes_executor_diagram.png" alt="Sequence diagram for Fawkes Executor" width="50%" />
</center>

Hence, at run-time, the hooks are executed as a fixed static schedule without preemption. Multiple threads registered in the same hook are executed in parallel.

Orthogonal to the sequential execution of sense-plan-act like applications, it is possible to define further constraints on the execution order by means of a `Barrier`. A barrier defines a number of threads, which need to have finished before the thread can start, which owns the barrier.

These concepts are implemented by the following main classes:

* *Wakeup hook* by `SyncPoint` and `SyncPointManager`, which manages a list of synchronization points.
* *Executor* by the class `FawkesMainThread`, which is the scheduler, responsible for calling the user threads.
* `ThreadManager`, which is derived from `BlockedTimingExecutor`, provides the necessary API to add and remove threads to wakeup hooks as well as for sequential execution of the wakeup-hooks.
* `Barrier` is an object similar to `condition_variable` in C++.

#### 讨论

All threads are executed with the same priority. If multiple sense-plan-act chains shall be executed with different priorities, e.g. to prefer execution of emergency-stop over normal operation, then this framework reaches its limits.

Also, different execution frequencies cannot be modeled by a single instance of this sense-plan-act chain. However, in robotics the fastest sensor will drive the chain and all other hooks are executed with the same frequency.

The option to execute threads independent of the predefined wakeup-hooks is very useful, e.g. for diagnostics. The concept of the Barrier is useful for satisfying functional dependencies which need to be considered in the execution order.

<!--
### Orocos

TODO INSERT DESCRIPTION ON PARTIAL ORDER SCHEDULING.


### CoSiMA

TODO INSERT DESCRIPTION ON MODEL-BASED APPROACH BY COSIMA (ON TOP OF OROCOS) FROM FOLLOWING PAPER:

D. L. Wigand, P. Mohammadi, E. M. Hoffman, N. G. Tsagarakis, J. J. Steil and S. Wrede, "An open-source architecture for simulation, execution and analysis of real-time robotics systems," 2018 IEEE International Conference on Simulation, Modeling, and Programming for Autonomous Robots (SIMPAR), Brisbane, QLD, 2018, pp. 93-100.
doi: 10.1109/SIMPAR.2018.8376277
URL: http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8376277&isnumber=8376259
-->


## 参考文献
* [S2021]<a name="S2021"></a> J. Staschulat, "Micro-ROS: The rclc Executor", in Workshop ROS 2 Executor: How to make it efficient, real-time and deterministic? at ROS World, Oct. 2021, [[slides](https://ec2a4d36-bac8-4759-b25e-bb1f794177f4.filesusr.com/ugd/984e93_749e27b917a54b45b9ccb5be930841b8.pdf)] [[Video](https://www.youtube.com/embed/IazrPF3RN1U)]

* [SLD2021]<a name="SLD2021"></a> J. Staschulat, R. Lange and D. N. Dasari, "Budget-based real-time Executor for Micro-ROS", arXiv Pre-Print, May 2021. [[paper](https://arxiv.org/abs/2105.05590)] 

* [L2020]<a name="L2020"></a> Ralph Lange: Advanced Execution Management with ROS 2, ROS-Industrial Conference, Dec 2020 [[Slides]](https://micro-ros.github.io/download/2020-12-16_Advanced_Execution_Management_with_ROS_2.pdf)

* [SLL2020]<a name="SLL2020"></a> J. Staschulat, I. Lütkebohle and R. Lange, "The rclc Executor: Domain-specific deterministic scheduling mechanisms for ROS applications on microcontrollers: work-in-progress," 2020 International Conference on Embedded Software (EMSOFT), Singapore, Singapore, 2020, pp. 18-19. [[Paper]](https://ieeexplore.ieee.org/document/9244014) [[Video]](https://whova.com/embedded/session/eswe_202009/1145800/)

* [CB2019]<a name="CB2019"> </a> D. Casini, T. Blaß, I. Lütkebohle, B. Brandenburg: Response-Time Analysis of ROS 2 Processing Chains under Reservation-Based Scheduling, in Euromicro-Conference on Real-Time Systems 2019. [[Paper]](http://drops.dagstuhl.de/opus/volltexte/2019/10743/) [[slides]](https://t-blass.de/talks/ECRTS2019.pdf)

* [L2018]<a name="L2018"></a> Ralph Lange: Callback-group-level Executor for ROS 2. Lightning talk at ROSCon 2018. Madrid, Spain. Sep 2018. [[Slides]](https://roscon.ros.org/2018/presentations/ROSCon2018_Lightning1_4.pdf) [[Video]](https://vimeo.com/292707644)

* [EK2018]<a name="EK2018"></a> R. Ernst, S. Kuntz, S. Quinton, M. Simons: The Logical Execution Time Paradigm: New Perspectives for Multicore Systems, February 25-28 2018 (Dagstuhl Seminar 18092). [[Paper]](http://drops.dagstuhl.de/opus/volltexte/2018/9293/pdf/dagrep_v008_i002_p122_18092.pdf)

* [NSP2018]<a name="NSP2018"></a> A. Naderlinger, S. Resmerita, and W. Pree: LET for Legacy and Model-based Applications,
Proceedings of The Logical Execution Time Paradigm: New Perspectives for Multicore Systems (Dagstuhl Seminar 18092), Wadern, Germany, February 2018.

* [BP2017]<a name="BP2017"></a> A. Biondi, P. Pazzaglia, A. Balsini,  M. D. Natale: Logical Execution Time Implementation and Memory Optimization Issues in AUTOSAR Applications for Multicores, International Worshop on Analysis Tools and Methodologies for Embedded and Real-Time Systems (WATERS2017), Dubrovnik, Croatia.[[Paper]](https://pdfs.semanticscholar.org/4a9e/b9a616c25fd0b4a4f7810924e73eee0e7515.pdf)

* [KZH2015]<a name="KZH2015"></a> S. Kramer, D. Ziegenbein, and A. Hamann: Real World Automotive Benchmarks For Free, International Workshop on Analysis Tools and Methodologies for Embedded adn Real-Time Sysems (WATERS), 2015.

* [HHK2001]<a name="HHK2001"></a> Henzinger T.A., Horowitz B., Kirsch C.M. (2001) Giotto: A Time-Triggered Language for Embedded Programming. In: Henzinger T.A., Kirsch C.M. (eds) Embedded Software. EMSOFT 2001. Lecture Notes in Computer Science, vol 2211. Springer, Berlin, Heidelberg

* [LL1973]<a name="LL1973"></a> Liu, C. L.; Layland, J.:Scheduling algorithms for multiprogramming in a hard real-time environment, Journal of the ACM, 20 (1): 46–61, 1973.

## 致谢

This activity has received funding from the European Research Council (ERC) under the European Union's Horizon 2020 research and innovation programme (grant agreement n° 780785).
