## 构建固件

配置步骤结束后，直接构建固件：

```bash
# 构建步骤
ros2 run micro_ros_setup build_firmware.sh
```

## 烧录固件

将固件烧录到平台的过程因硬件平台而异。
关于本教程的目标平台（**[Olimex STM32-E407](https://www.olimex.com/Products/ARM/ST/STM32-E407/open-source-hardware)**），将使用 JTAG 接口来烧录固件。

连接 [Olimex ARM-USB-TINY-H](https://www.olimex.com/Products/ARM/JTAG/ARM-USB-TINY-H/) 到开发板：

<img width="400" style="padding-right: 25px;" src="../imgs/2.jpg">

确保开发板电源跳线（PWR_SEL）处于 3-4 位置，以便通过 JTAG 接口为开发板供电：

<img width="400" style="padding-right: 25px;" src="../imgs/1.jpg">

将计算机通过 JTAG 适配器连接到 Olimex 开发板后，运行烧录步骤：

```bash
# 烧录步骤
ros2 run micro_ros_setup flash_firmware.sh
```
