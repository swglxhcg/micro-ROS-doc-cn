---
title: 特性和架构
permalink: /docs/overview/features/
redirect_from:
  - /docs/
  - /docs/overview/
  - /architecture/
---

<script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
<script>
function updateVisibilityOfFeatureDescriptions() {
  $('.feature').not('feature_is_active').find('.three_dots').show();
  $('.feature').not('feature_is_active').find('.feature_description').slideUp(500);
  $('.feature_is_active').find('.three_dots').hide();
  $('.feature_is_active').find('.feature_description').slideDown(500);
}

$(document).ready( function() {
  $('.feature_description').hide();
  updateVisibilityOfFeatureDescriptions();

  $('.feature').click( function() {
    if ($(this).hasClass("feature_is_active")) {
      $(this).removeClass("feature_is_active");
    } else {
      $('.feature').removeClass("feature_is_active");
      $(this).addClass("feature_is_active");
    }
    updateVisibilityOfFeatureDescriptions();
  });
});
</script>

<style>
  .three_dots {
    color: #BBBBBB;
  }
  .feature_title {
    font-weight: bold;
    margin: 8pt 0 2pt 0;
  }
  .feature_description {
    margin-left: 3em;
  }
  .feature_description > p {
    margin: 0 0 2pt 0;
  }
</style>

Micro-ROS 提供**七个关键特性**，使其可以随时用于您的基于微控制器的机器人项目：

<div class="feature feature_is_active">
 <div class="feature_title">&#10004; 针对微控制器优化的客户端 API，支持所有主要 ROS 概念<span class="three_dots"> (...)</span></div>
 <div class="feature_description">
  <p>Micro-ROS 将所有主要核心概念（如节点、发布/订阅、客户端/服务、节点图、生命周期等）带到微控制器 (MCU) 上。Micro-ROS 的客户端 API（采用 C 编程语言）基于标准 ROS 2 客户端支持库 (rcl) 和一组扩展及便捷函数 (rclc)。</p>
  <p>rcl+rclc 组合针对 MCU 进行了优化。初始化阶段后，它可以无需任何动态内存分配即可使用。rclc 包提供高级执行机制，允许实现嵌入式系统工程中经过验证的调度模式。</p>
 </div>
</div>

<div class="feature">
 <p class="feature_title">&#10004; 与 ROS 2 无缝集成<span class="three_dots"> (...)</span></p>
 <div class="feature_description">
  <p>Micro-ROS 代理将 MCU 上的 micro-ROS 节点（即组件）与标准 ROS 2 系统无缝连接。这允许使用已知的 ROS 2 工具和 API 访问 micro-ROS 节点，就像普通 ROS 节点一样。</p>
 </div>
</div>

<div class="feature">
 <p class="feature_title">&#10004; 资源受限但灵活的中间件<span class="three_dots"> (...)</span)</p>
 <div class="feature_description">
  <p>eProsima 的 Micro XRCE-DDS 满足深度嵌入式系统中间件的所有要求。这就是为什么 micro-ROS 成为这一针对极端资源受限环境 (XRCE) 的新 DDS 标准实现的应用程序之一。为了与 micro-ROS 堆栈中的 ROS 中间件接口 (rmw) 集成，引入了静态内存池以避免运行时的动态内存分配。</p>
  <p>该中间件内置支持串行传输、以太网 UDP、Wi-Fi 和 6LoWPAN，以及蓝牙。此外，Micro XRCE-DDS 源代码提供了用于实现更多传输支持的模板。</p>
 </div>
</div>

<div class="feature">
 <p class="feature_title">&#10004; 多 RTOS 支持，通用构建系统<span class="three_dots"> (...)</span></p>
 <div class="feature_description">
  <p>Micro-ROS 支持三种流行的开源实时操作系统 (RTOS)：FreeRTOS、Zephyr 和 NuttX。它可以移植到任何带有 POSIX 接口的 RTOS 上。</p>
  <p>RTOS 特定的构建系统集成到少数通用设置脚本中，这些脚本作为 ROS 2 包提供。因此，ROS 开发者可以使用他们常用的命令行工具。此外，micro-ROS 提供了与 RTOS 特定工具链的精选集成（例如，用于 ESP-IDF 和 Zephyr）。</p>
 </div>
</div>

<div class="feature">
 <p class="feature_title">&#10004; 宽松的许可证<span class="three_dots"> (...)</span></p>
 <div class="feature_description">
  <p>Micro-ROS 采用与 ROS 2 相同的宽松许可证，即 Apache License 2.0。这适用于 micro-ROS 客户端库、中间件层和工具。</p>
  <p>在使用底层 RTOS 创建项目时，请注意 RTOS 项目或供应商的许可证，如<a href="../license/">许可证</a>页面上的进一步说明。</p>
 </div>
</div>

<div class="feature">
 <p class="feature_title">&#10004; 充满活力的社区和生态系统<span class="three_dots"> (...)</span></p>
 <div class="feature_description">
  <p>Micro-ROS 由一个不断增长的、自组织的社区开发，该社区由嵌入式工作组（正式的 ROS 2 工作组）支持。社区分享入门级教程，通过 Slack 和 GitHub 提供支持，并在每月一次的公开工作组视频会议中会面。当然，eProsima 为 Micro XRCE-DDS 提供商业支持。</p>
  <p>该社区还围绕 micro-ROS 创建工具。例如，为了将基于 micro-ROS 的应用优化到 MCU 硬件，开发了特定的基准测试工具。这些工具可以检查内存使用、CPU 时间消耗和整体性能。</p>
 </div>
</div>

<div class="feature">
 <p class="feature_title">&#10004; 长期可维护性和互操作性<span class="three_dots"> (...)</span></p>
 <div class="feature_description">
  <p>Micro-ROS 由成熟的组件组成：著名的开源 RTOS、标准化的中间件和标准 ROS 2 客户端支持库 (rcl)。通过这种方式，为了长期可维护性，最小化了特定于 micro-ROS 的代码量。同时，micro-ROS 堆栈保留了标准 ROS 2 堆栈的模块化。Micro-ROS 可以与自定义中间件层一起使用——因此是标准的——或自定义 ROS 客户端库。</p>
  <p>此外，通过<a href="https://soss.docs.eprosima.com/">系统综合器</a> (SOSS)，这是一个快速轻量级的<a href="https://www.omg.org/spec/DDS-XTypes">OMG DDS-XTYPES 标准</a>集成工具，可以连接更多中间件协议。例如，我们开发了 SOSS-FIWARE 和 SOSS-ROS2 System-Handle，它们通过利用 SOSS 核心的集成能力，将 ROS 2 和 micro-ROS 与<a href="https://www.fiware.org/">FIWARE Context Broker</a>通过 NGSIv2（下一代服务接口）标准连接起来。</p>
 </div>
</div>

## 分层和模块化架构

Micro-ROS 遵循 [ROS 2 架构](https://docs.ros.org/en/rolling/Concepts/Advanced/About-Internal-Interfaces.html)，并利用其中间件可插拔性使用针对微控制器优化的 [DDS-XRCE](https://www.omg.org/spec/DDS-XRCE/)。此外，它使用基于 POSIX 的 RTOS（FreeRTOS、Zephyr 或 NuttX）而不是 Linux。

<img src="/img/micro-ROS_architecture.png" style="display: block; margin: auto; width: 100%; max-width: 500px;"/>

深蓝色组件是专门为 micro-ROS 开发的。浅蓝色组件来自标准 ROS 2 堆栈。我们尽可能多地将代码贡献回 ROS 2 主线代码库。
