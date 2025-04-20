众所周知的，linux的网络配置有很多工具，例如NetworkManager、Netplan等等，他们千奇百怪。在这做一个整理。



# 1.工具的分层

<img title="" src="http://123.57.190.49:12121/api/image/PNPPJ6NJ.png" alt="Imgur" data-align="inline">

linux的网络底层是Linux网络子系统，Linux网络子系统核心组件包括:

- 网络接口（Network Interfaces）: 如 eth0、wlan0，由内核管理。
- 网络协议栈: 包括 TCP/IP、ARP 等，负责数据包的处理。
- 路由表（Routing Table）: 由内核维护，决定数据包的转发路径。
- Netlink: 内核和用户空间通信的接口，用于动态配置网络。

内核通过 系统调用 和 Netlink 协议 接受来自用户空间的指令，完成网络接口的启用、IP地址的分配、路由规则的设置等。

内核本身不负责持久化，所有配置在重启后会丢失。持久化是由用户空间工具通过配置文件或服务实现的。可见在设计时，子系统在设计之初就不考虑持久化的问题，如果需要持久化，需要使用其余工具实现，非常符合Kiss原则。

linux网络子系统使用NetLink协议来与用户态程序通信。linux网络子系统和NetLink的有关知识，在这里不重要，有机会我们在写一篇。

在linux子系统之上，有同样不支持持久化的iproute2工具包，他提供了一组指令来配置网络的ip，路由等内容。

除了iproute2，还有负责无线接入的wpa-supplicant(这个之后会介绍，可能需要单独写一个如何进行wifi接入的文章，以处理恶心人的校园网认证方式)，

对于持久化方案，目前常见的有networkd、netplan和netmanager，当然还有什么dhcpcd这种不常用的东西。

# 2.iproute2

iproute2是net-tools的继任者，显然iproute2的命令更符合直觉且提供的功能更多，所以这里不在记录net-tools如何使用，如果您一定要使用，我建议执行如下操作：

```shell
sudo apt remove net-tools
sudo apt install iproute2
```

iproute2的常用指令就像他的名字一样，配置ip和路由

如果要增加一个ip

```shell
ip addr add 192.168.100.1/24 dev eth0
```

删除可以使用del

如果要配置一个路由，那么可以使用

```shell
ip route add 192.168.100.0/24 via 192.168.100.1 dev eth0
```

via 确实比gw好理解，不知道为什么把网关起名为gateway。

# 3./etc/network/interfaces

该配置文件由ifupdown软件来使用，配置文件。

下面给出一个常见的配置文件：

```shell
# /etc/network/interfaces
auto eth0
iface eth0 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.1
```

# 4.networkd

systemd-networkd 是 systemd 项目中的一个组件，用于管理网络配置。它是一个轻量级的守护进程，旨在取代传统的网络管理工具（如 ifupdown 或老的 /etc/network/interfaces），提供现代化、系统化的网络配置方案。

networkd配置文件位于

* /usr/lib/systemd/network，非持久化的运行时网络配置目录位于
* /run/systemd/network ，本地管理网络配置位于
* /etc/systemd/networket中的配置文件具有最高优先级

使用 Netlink 协议 直接与 Linux 内核的网络子系统通信（类似 iproute2）。

# 5.NetworkManager

NetworkManager 是一个开源的网络管理工具，最初由 Red Hat 于 2004 年开发，现由 GNOME 项目托管。它旨在简化 Linux 系统中的网络配置和管理，特别适用于动态网络环境（如无线网络、移动宽带）和桌面用户。

**主要功能**

1. 动态网络管理:
* 自动检测和连接可用网络（如 Wi-Fi 热点）。
* 在有线和无线网络之间无缝切换（优先使用有线连接）。
* 支持热插拔设备（如插入网线或 USB 网卡时自动配置）。
2. 多种连接类型:
* 有线网络（Ethernet）。
* 无线网络（Wi-Fi，配合 wpa_supplicant 处理认证）。
* 移动宽带（通过 ModemManager）。
* VPN（支持 OpenVPN、PPTP 等插件）。
* DSL/PPPoE。
3. 用户界面:
* nmcli: 命令行工具，适合脚本化和高级管理。
* nmtui: 文本用户界面（TUI），适合无图形环境的配置。
* 图形界面: 如 nm-applet（GNOME/KDE 的系统托盘工具）。
4. 配置持久化:将网络配置存储在 /etc/NetworkManager/system-connections/ 下的 .nmconnection 文件中（INI 格式）。
5. API 支持:
* 通过 D-Bus 提供丰富的 API，允许其他程序查询和控制网络状态。
* 提供 libnm 库，支持高级语言绑定（如 Python）。

NetworkManger直接使用Netlink与内核交互, 其组成十分复杂。

# 6.Netplan

Netplan 是一个网络配置抽象工具，它通过 YAML 格式的配置文件定义网络设置，并将这些配置交给后端（如 systemd-networkd 或 NetworkManager）执行。它并不是一个独立的网络管理服务，而是充当“前端”角色，简化了复杂的网络配置过程。Netplan 解析 YAML 文件，生成后端所需的配置文件。

例如：

* 为 systemd-networkd 生成 .network 文件。
* 为 NetworkManager 生成 .nmconnection 文件。

通过 netplan apply 命令将配置传递给后端，后端再与内核交互。
