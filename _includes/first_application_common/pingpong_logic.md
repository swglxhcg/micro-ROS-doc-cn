本示例展示了一个 micro-ROS 节点，其中包含两个与 `ping` 和 `pong` 主题相关联的发布者-订阅者对。
该节点使用 `ping` 发布者发送一个具有唯一标识符的 `ping` 包。
如果 `ping` 订阅者收到来自外部节点的 `ping`，则 `pong` 发布者会响应一个 `pong` 来回复收到的 `ping`。为了测试此逻辑是否正确运行，我们实现了与 ROS 2 节点的通信：

* 监听 `ping` 订阅者发布的主题。
* 发布一个 `fake_ping` 包，该包由 micro-ROS `ping` 订阅者接收。
  因此，micro-ROS 应用程序上的 `pong` 发布者将发布一个 `pong`，以表示它正确收到了 `fake_ping`。

下图阐明了这些实体之间的通信流程：

![pingpong](https://www.plantuml.com/plantuml/png/ZOv1IyGm48Nl-HMFlPVI_W3PewSAwar4GZjsWvrCIAO7SVtlTc8FAxYmbydZyONl7Olwh2ilhdo4c7ps39OeuoaB4pIlv5oKYV0oRBTx1Np1qE7B0QDmaaXHSMWvZ5aU7vxQ5E9y02fdkRiAoWKe1dvVglfTrT-kwczLzQQguz0qT_lVUj7aC9_KoihLMw4wqGOgGIL1tZ6O43ZZML8OxVrCXFDU_jsv5KMdDovpQU_9JvJ_0-KAI762cPqxRd5LNdu3Bpy0)
