# 分支、ROS 版本与 WSL2 硬件接入

## 选择分支

| 分支 | 目标环境 | 使用定位 |
| --- | --- | --- |
| [main](https://github.com/zay002/Arachne-armed-mobile-robot/tree/main) | Ubuntu 24.04 + ROS 2 Jazzy；Windows 11 / WSL2 | 本机开发、模型展示、离线验证，以及 WSL2 硬件接入 |
| [jetson](https://github.com/zay002/Arachne-armed-mobile-robot/tree/jetson) | Ubuntu 22.04 + ROS 2 Humble；Jetson | 真机集成、现场示教、相机/雷达与移动抓取功能的开发来源 |

main 已合入截至本次更新的 jetson 已提交功能，并在 Jazzy 上完成构建与离线验证。
两个分支的区别主要是目标 ROS / Ubuntu 环境及验证平台，不代表 main 永久自动同步 jetson，
也不代表任一分支中的全部硬件功能都已在另一平台验证。后续 jetson 新提交仍需整合后重新验证。

不要将 Humble 的 build/install 目录直接用于 Jazzy，或在一个终端混合 source 两个 ROS 发行版。
切换环境时使用独立工作目录和新的终端，重新构建对应分支。Jetson 原生 Linux 的硬件接入
不需要经过 Windows / WSL2 USB 穿透。

## WSL2 推荐搭配 hurry-porter

推荐使用作者的 [hurry-porter](https://github.com/zay002/hurry-porter)，
快速完成 Windows USB 设备扫描、诊断和向 WSL2 的 USB 穿透。
它基于 usbipd-win，为 USB 串口、USB-CAN、USB 手柄等提供统一入口，也支持 LAN 设备发现、
手柄桥接和 ROS 参数导出。它是可选的硬件接入辅助工具，Arachne 的纯模型预览不依赖它。

先按 [hurry-porter 的安装说明](https://github.com/zay002/hurry-porter#要求)
在 Windows 安装 usbipd-win，再在 WSL2 的 Ubuntu 24.04 / Jazzy 终端构建：

```bash
git clone https://github.com/zay002/hurry-porter.git ~/hurry-porter
cd ~/hurry-porter
source /opt/ros/jazzy/setup.bash
colcon build
source install/setup.bash

hurry doctor
hurry scan --json
# 将 4-3 替换成扫描结果中目标设备的 BUSID
hurry attach 4-3 --dry-run
hurry attach 4-3
```

穿透后确认 WSL2 已出现对应设备节点，再进入 Arachne 的 main 工作目录启动所需驱动。
首次共享尚未 bind 的设备时，按 hurry 的诊断提示在 Windows 管理员终端完成
`usbipd bind --busid 4-3`，再重试 attach；BUSID 同样以扫描结果为准。
USB 穿透解决的是设备接入；Linux 驱动、串口权限和设备参数仍需匹配。
Waveshare USB-CAN-A 使用厂商串口协议，不应当作 SocketCAN 的 can0 使用。
Aubo 的局域网连接也不需要 USB 穿透，可使用 hurry-porter 的 LAN 发现能力辅助诊断。

当前 hurry-porter 文档面向 Jazzy / Python 3.12；这里推荐的是 WSL2 + main 组合，
不将它作为 Jetson Humble 环境的必需依赖。具体设备兼容性、权限及重新连接操作以
[hurry-porter 文档](https://github.com/zay002/hurry-porter)为准。

## 本次验证范围

main：Jazzy 下 13 个包构建、离线入口检查及 dry-run 通过，RViz 模型与示教面板已实际运行。
该 WSL2 环境中的 Fast DDS 模型消息接收异常，通过 Cyclone DDS 恢复显示；
安装和启动方式见 [main README](https://github.com/zay002/Arachne-armed-mobile-robot/tree/main#jazzy--wsl2-model-preview)。

README 中 Jazzy / WSL2 截图只代表 main 的本机展示，不作为 jetson 真机测试证据。
本次未重新验证 Jetson 硬件、Humble 构建、Gazebo 或真实 USB attach。

