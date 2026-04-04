执行命令后，工作空间中必须存在一个名为 `firmware` 的文件夹。

此步骤负责（除其他外）为您正在使用的特定平台下载一组 micro-ROS 应用程序。
在 Zephyr 的情况下，这些应用程序位于 `firmware/zephyr_apps/apps`。
每个应用程序由一个文件夹表示，其中包含以下文件：

* `src/main.c`：此文件包含应用程序的逻辑。
* `app-colcon.meta`：此文件包含特定于 micro-ROS 应用程序的 colcon 配置。有关如何通过此文件配置 RMW 的详细信息，请参阅[此处](/docs/tutorials/advanced/microxrcedds_rmw_configuration/)。
* `CMakeLists.txt`：这是包含编译应用程序脚本的 CMake 文件。
* `<transport>.conf`：这是特定于 Zephyr 且依赖于传输的应用程序配置文件。`<transport>` 可以是 `serial`、`serial-usb` 和 `host-udp`。

为了让用户创建其自定义应用程序，需要在此位置注册一个名为 `<my_app>` 的文件夹，其中包含刚才描述的四个文件。
