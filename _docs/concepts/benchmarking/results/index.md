---
title: 结果
redirect_from: /benchmarking/results/
permalink: /docs/concepts/benchmarking/results/
---

## 目录

* [从追踪到基准测试](#从追踪到基准测试)
* [通信结果](#通信结果)
* [实时结果](#实时结果)
* [执行情况](#执行情况)
* [函数调用统计](#函数调用统计)
* [静态内存使用](#静态内存使用)
* [动态内存使用](#动态内存使用)
* [功耗](#功耗)

## 从追踪到基准测试

低级 RTOS (NuttX) 以特定方式进行了插桩，以提供不同类别的基准测试测量（内存、执行时间等...）。收集的数据遵循[通用追踪格式](https://diamon.org/ctf/)。原始追踪随后使用 [Babeltrace API](https://babeltrace.org/) 追踪操作工具包进行处理。数据解释由用户决定。

有关基准测试结果和方法的更多信息在以下[文档](https://github.com/micro-ROS/benchmarking-results/blob/master/pdfs/OFERA_55_D5.4_Micro-ROS_benchmarks_-_Final.pdf)中处理。

解释所依据的结果可在 [benchmarking_results](https://github.com/micro-ROS/benchmarking-results/tree/master/aug2020) 仓库中找到。

### 一般方法

根据 micro-ROS 设置的通信介质类型，方法可能会有所不同。当然，差异很小且与传输协议相关。

为了实现基准测试，RTOS 和应用程序都进行了插桩。根据目标评估的不同，在 RTOS 的不同部分放置了不同的探针。数据格式遵循名为通用追踪格式 (V1.8) 的标准。该标准也在 Zephyr 中使用，Zephyr 是与 NuttX 一起被 micro-ROS 项目支持的 RTOS 之一。

事实上，CTF 核心是从 Zephyr 移植到 NuttX 的。

数据使用 babeltrace 和 babeltrace Python API 进行检索和分析。

所有事件都使用内部自由运行计时器计时（在 NuttX 运行于 Olimex STM32-E407 TIM2 的情况下）。因此，该设备可以拥有提供近 10 纳秒分辨率的定时器时钟。当前配置分辨率为 100 纳秒，这对于评估通信性能来说绰绰有余，考虑到 100Mbps 下的最小以太网（64字节）帧。每个测量都使用上述自由运行计时器打上时间戳。

软件配置可能会发生变化。然而，软件的角色将保持不变：

 * 在 PC 上运行的代理
 * 在一个 Olimex STM32-E407 上运行的订阅者
 * 在一个 Olimex STM32-E407 上运行的发布者

在硬件层面，USB-CDC/ACM 控制台将在 Olimex STM32-E407 开发板上用于以太网和串行基准测试。对于 6LoWPAN，串行 USART6 将用作控制台，以减少内存占用和执行影响。因此，两个 Olimex STM32-E407 开发板的 USB OTG1 应连接到计算机。必须执行额外的硬件设置，但这将取决于拓扑类型（以太网/串行/6LoWPAN）：

下面显示的结果是同一应用程序的成果：**发布者**。
只对 micro-ROS 相关函数进行了基准测试。因此，代码的所有串行/IP/无线电配置部分将不被考虑。

下面是数据处理描述。从序列化的二进制 CTF 数据到 Babeltrace API 的输出：

![](./images/bm_dataflow.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

## 通信结果

以下是通信比特率 RX/TX：

![](./images/bm_com.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}


_观察：_

  根据数据，以太网表现最佳，这符合预期。

## 实时结果

下面我们报告执行基准测试

从 Babeltrace 提取的数据显示关于 NuttX 调度器的以下信息。附加信息如下：

 * thread_id 0 是空闲线程
 * thread_id 3 是低优先级工作队列（RTOS kthread）
 * thread_id 7 是发布者

```
[01:00:21.445833238] (+0.000009524) 0 thread_resume: { thread_id = 7 }
[01:00:21.445993047] (+0.000159809) 0 thread_suspend: { thread_id = 7 }
[01:00:21.446002761] (+0.000009714) 0 thread_resume: { thread_id = 3 }
[01:00:21.446051904] (+0.000049143) 0 thread_suspend: { thread_id = 3 }
[01:00:21.446061428] (+0.000009524) 0 thread_resume: { thread_id = 0 }
[01:00:21.446085428] (+0.000024000) 0 thread_suspend: { thread_id = 0 }
[01:00:21.446095428] (+0.000010000) 0 thread_resume: { thread_id = 3 }
[01:00:21.446133047] (+0.000037619) 0 thread_suspend: { thread_id = 3 }
[01:00:21.446142571] (+0.000009524) 0 thread_resume: { thread_id = 0 }
[01:00:21.446273523] (+0.000130952) 0 thread_suspend: { thread_id = 0 }
[01:00:21.446283523] (+0.000010000) 0 thread_resume: { thread_id = 3 }
[01:00:21.446335809] (+0.000052286) 0 thread_suspend: { thread_id = 3 }
[01:00:21.446345333] (+0.000009524) 0 thread_resume: { thread_id = 7 }
[01:00:21.446505333] (+0.000160000) 0 thread_suspend: { thread_id = 7 }
[01:00:21.446514952] (+0.000009619) 0 thread_resume: { thread_id = 3 }
[01:00:21.446564190] (+0.000049238) 0 thread_suspend: { thread_id = 3 }
[01:00:21.446573714] (+0.000009524) 0 thread_resume: { thread_id = 0 }
[01:00:21.446597714] (+0.000024000) 0 thread_suspend: { thread_id = 0 }
[01:00:21.446607714] (+0.000010000) 0 thread_resume: { thread_id = 3 }
[01:00:21.446645333] (+0.000037619) 0 thread_suspend: { thread_id = 3 }
[01:00:21.446654857] (+0.000009524) 0 thread_resume: { thread_id = 0 }
[01:00:21.446779047] (+0.000124190) 0 thread_suspend: { thread_id = 0 }
[01:00:21.446789142] (+0.000010095) 0 thread_resume: { thread_id = 3 }
[01:00:21.446841333] (+0.000052191) 0 thread_suspend: { thread_id = 3 }
[01:00:21.446850857] (+0.000009524) 0 thread_resume: { thread_id = 7 }
[01:00:21.447010571] (+0.000159714) 0 thread_suspend: { thread_id = 7 }
[01:00:21.447020285] (+0.000009714) 0 thread_resume: { thread_id = 3 }
```

_观察：_

根据上述结果，软件以确定性方式运行。事实上，仔细观察可以看出，运行序列是相同的。

此外，在事件之间切换时，相关事件之间的时间增量变化非常小（连续的 thread_suspend/thread_resume）。

而且，调度器执行快速上下文切换，平均持续 10 微秒。

## 执行情况

下面是执行基准测试的描述。CPU 上花费的时间分为两部分（I/O 操作、订阅者操作）：

![](./images/bm_execution.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}


_观察：_
    根据数据，大部分时间都花在了 I/O 操作上。

## 函数调用统计

下面报告了每种通信介质的函数调用计数：

**以太网**
![](./images/fusage_eth.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

**串行**
![](./images/fusage_serial.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

**6LoWPAN**
 ![](./images/fusage_6lowpan.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

## 静态内存使用

下面是静态内存分析的表示：

![](./images/bm_max_static_memory.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

_观察：_

6LoWPAN 是消耗最多静态内存的介质，因为该协议运行在 IPv6 之上。

## 动态内存使用

下图显示动态分配的总数。每个条柱被分成一组内存块。每组中有不同大小的块。每组的大小介于最小尺寸（前一组的尺寸）和最大尺寸（图例中显示的尺寸）之间。例如，图例中黄色染色的块表示所有大于上一组内存块的块，这里是 16 字节。但它们小于或等于 32 字节。

![](./images/bm_allocation_nbr.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

_观察：_

无论通信介质如何分配的块都不大也不多。大多数分配发生在初始化期间。

## 功耗

下面按通信介质分类描述了能耗：

![](./images/bm_power.png){:.img-responsive and style="max-width: 100%; margin-left: auto; margin-right: auto;"}

_观察：_

通信介质对吞吐量有很大影响，进而影响功耗。以太网提供高吞吐量，但代价是更高的功耗。相比之下，串行通信介质提供较低的比特率，优势是能耗更低。
