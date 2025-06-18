# 在 Risc-V 开发板上使用 ROS2

## 介绍

### ROS2 是什么

ROS 2 是一个开源的、灵活的框架，用于编写机器人软件。它并不是一个传统意义上的操作系统 (OS)，而更像是一个元操作系统 (Meta-Operating System) 或中间件 (Middleware)。它提供了一系列库、工具和约定，旨在简化跨各种机器人平台的复杂和健壮的机器人行为的创建。

### 为什么要用 ROS2

- 强大的数据分发系统 DDS(Data Distribution System)
    - 传统的嵌入式开发主流采用 UART，SPI，I2C，CAN 等通信协议，通常是点对点或主从的紧耦合模式，通信双方需要硬编码的连接信息，可维护性和可拓展性较差。
    - DDS 采用去中心化的 Publish/Subscribe 模型，节点内无须硬编码连接信息，只需关注 Topic 和数据类型，使得系统非常模块化，可以独立开发测试任意节点，而不影响其他部分。DDS 通信的核心是数据，它将字节流处理、数据帧格式、打包解包等工作量大且易出错的环节封装起来，我们只需关注数据内容本身。
- 易于构建分布式系统
    - 传统的嵌入式开发构建分布式系统需要大量的网络编程和协议设计工作。
    - 而 DDS 的设计天然支持分布式计算，节点可以在同一设备的不同进程中，也可以分布在网络连接的不同物理设备上，通信方式对开发者透明。这对于多机器人协作、传感器网络等应用至关重要。
- 模块化与标准化
    - 传统的嵌入式开发的软件模块化程度依赖于开发者经验和项目规范，没有统一的标准，导致在阅读中大型嵌入式项目代码时容易头晕。
    - ROS2 鼓励将系统功能分解为独立的、可执行的节点。每个节点负责特定任务，非常易于开发、测试和维护。而 Package 的概念提供了标准化的软件组织方式，方便共享和重用代码、库、消息定义等。

除此之外 ROS2 还有许多关键的架构特性，篇幅所限，在此不再赘述。

### 对 Risc-V 开发板适配良好的 OS

目前对 Risc-V 和 ROS2 适配较好的 OS 有两个，一个是使用 RPM（RHEL/CentOS）做包管理的 [OpenEuler](https://www.openeuler.org/zh/)，另一个是基于 Debian 的 [RevyOS](https://docs.revyos.dev/)。

## 前置任务

### 1.镜像刷写

关于镜像刷写请参考：

- RevyOS: [LicheePi4A镜像刷写教程](https://docs.revyos.dev/docs/Installation/licheepi4a/)

- OpenEuler: [RISC-V lpi4a 安装测试openEuler ROS Humble](https://openeuler-ros-docs.readthedocs.io/en/latest/index.html)

我所刷写的是 RevyOS 目前最新版本[镜像](https://fast-mirror.isrc.ac.cn/revyos/extra/images/lpi4a/20250526/)，因此后面教程没有特殊说明都是基于 RevyOS 展开。

请根据参考文档完成镜像刷写。

### 2.安装 ROS2
参考 RevyOS 文档：[Robot Operating System (ROS)](https://docs.revyos.dev/docs/desktop/software/ROS2/)






















