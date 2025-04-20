NetworkManager是当前带有GUI的Linux发行版中，最常见的的网络配置工具，甚至arch的installer里面都有NetworkManager的选项。在网络配置方面，NetworkManager甚至比Systemd的Networkd还要全，但是喷NetworkManager的人并不多，很奇怪的现象。

# 1. 介绍

NetworkManager 是一个在 Linux 和其他类 Unix 操作系统上用于简化和自动化网络连接配置与管理的软件工具（通常作为一个系统守护进程运行）。

它的主要目标是让网络连接尽可能地“无缝”和“自动”，尤其是在桌面环境和移动计算场景下，用户可能经常在不同的网络（如有线、无线 Wi-Fi、移动宽带、VPN）之间切换。

NetworkManager 的一些关键特性和作用：

1. **自动检测和连接**：NetworkManager 能够自动检测可用的网络接口（如以太网卡、Wi-Fi 适配器）和网络（如可用的 Wi-Fi 热点）。它可以配置为自动连接到已知的或首选的网络，无需用户手动干预。

2. **支持多种网络类型**：
   
   * **有线网络 (Ethernet)**：管理以太网连接，支持 DHCP 或静态 IP 配置。
   * **无线网络 (Wi-Fi)**：管理 Wi-Fi 连接，支持各种安全协议（WEP, WPA/WPA2/WPA3 Personal/Enterprise）。
   * **移动宽带**：通过相应的硬件（如 USB 调制解调器）管理 3G, 4G LTE, 5G 等蜂窝网络连接。
   * **VPN**：通过插件支持多种 VPN 协议，如 OpenVPN, IPsec (strongSwan, libreswan), WireGuard, PPTP 等。
   * **其他**：还支持 DSL/PPPoE、网络绑定 (Bonding)、网络桥接 (Bridging)、VLAN 等高级网络配置。

3. **用户界面**：
   
   * **图形界面 (GUI)**：通常与桌面环境集成，提供系统托盘图标（如 `nm-applet`）和设置面板，让用户可以方便地查看网络状态、选择 Wi-Fi 网络、输入密码、配置 VPN 等。`nm-connection-editor` 是一个常用的图形化连接编辑器。
   * **文本用户界面 (TUI)**：提供 `nmtui` 命令，这是一个在终端中运行的基于文本菜单的界面，方便在没有图形环境的情况下进行配置。
   * **命令行界面 (CLI)**：提供强大的 `nmcli` 命令，允许用户通过命令行查看状态、管理设备和连接，非常适合脚本自动化和高级用户。

4. **连接配置文件管理**：将每个网络连接的配置（如 SSID、密码、IP 设置、VPN 参数等）保存为“连接配置文件”（通常存储在 `/etc/NetworkManager/system-connections/` 目录下），方便管理和重用。

5. **系统集成**：通过 D-Bus 系统消息总线与其他系统服务和应用程序通信，允许它们查询网络状态或请求网络操作。

6. **多连接管理**：可以同时管理多个活动的网络连接，例如，可以同时连接到有线网络和 VPN。
   
   

# 2. CLI命令介绍

NetworkManager将链接和设备分开，也就是connection是一套配置，device是一套配置，NetworkManager可以将connection的配置添加到device上，从而使设备增加链接。

NetworkManager的CLI为nmcli nmcli的常见操作如下：

```shell
# 查看所有设备
nmcli device status
# 查看当前所有链接
nmcli connection show
# 创建一个链接
```

## 2.1 链接以太网

```shell
# 创建一个链接
nmcli connection add type ethernet ifname enp5s0    # 创建一个连接。这里没有指定method，则默认使用auto，也就是自动配置。类型是以太网，类型有以太网、wifi，adsl等，具体参考文章头部给的url
# nmcli connection add ifname enp5s0 autoconnect yes type ethernet ip4 10.1.1.1/8 gw4 10.1.0.1    # 创建一个静态ip的以太网连接
nmcli connection modify  myEth +ipv4.dns 8.8.8.8    # 给myEth的配置添加dns
nmcli connection modify  myEth ipv4.method manual  ipv4.addresses "192.168.43.64/24,10.0.0.23/8"   #修改myEth连接为手动，ip地址设置为两个
nmcli con mod myEth  autoconnect no     # 设置myEth连接配置为不自动连接(重启操作系统或从起NetworkManager就能看到不会自动连接了)
```

## 2.2 链接WIFI

```shell

# 查看wifi状态
nmcli radio wifi

#打开wifi
nmcli radio wifi on


# 增加一个和wifi的配置链接
nmcli connection add \
type wifi \
con-name <CONNECTION NAME> \
ifname <INTERFACE NAME> \
ssid <SSID> \
wifi-sec.key-mgmt wpa-eap \
802-1x.eap ttls \
802-1x.phase2-auth mschapv2 \
802-1x.identity <USERNAME> \
802-1x.password <PASSWORD>
```

# 3. 配置文件介绍

NetworkManager的主配置文件`/etc/NetworkManager/NetworkManager.conf`,此文件可用于配置 NetworkManager 的行为。

Connection的文件在`/etc/NetworkManager/system-connections/`中





# 参考

[nmcli命令详解(创建热点，连接wifi，管理连接等) - 进取有乐 - 博客园](https://www.cnblogs.com/mind-water/p/12079647.html#general%E5%AF%B9%E8%B1%A1%E5%B8%B8%E8%A7%84%E4%BF%A1%E6%81%AF)

[Nmcli连接802 1x网络(EAP) - No Wander](https://sur.moe/post/nmcli%E8%BF%9E%E6%8E%A5802-1x%E7%BD%91%E7%BB%9Ceap/)
