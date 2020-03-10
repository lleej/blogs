# 环境

- 操作系统：macOS High Sierra 10.13.6
- 虚机软件：VMWare Fusion 11.5.1
- 虚拟机：Win7 64位
- USB设备：移动硬盘 1T

# 问题

- 启动Win7虚拟机后，提示无法识别的USB设备

# 无效处理

- 找到`VMWare Fusion安装包/Library/Services`中的`Open VMWare USB Arbitrator Service`，打开后依然无法识别

# 有效处理

- 关闭Win7虚拟机后，打开该虚拟机的设置界面，设置"USB和蓝牙"
- 设置“USB高级选项”中的“USB兼容性”为“USB 3.0”
- 系统提示，需要额外安装[驱动程序](https://downloadcenter.intel.com/download/22824/USB-3-0-Driver-Intel-USB-3-0-eXtensible-Host-Controller-Driver-for-Intel-8-9-100-Series-and-C220-C610-Chipset-Family)
- 下载该驱动程序，并在虚拟机（注意：不支持xp和vista）中安装该驱动
- 可以访问“USB设备”

