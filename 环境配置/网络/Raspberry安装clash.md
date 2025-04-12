# 1. 下载可执行文件

clash 改头换面以原神的身份重生（不是），在mihomo这个项目（点击超链接跳转）的[relase](https://github.com/MetaCubeX/mihomo/releases)里找到对应系统架构的预编译文件，因为我的树莓派装的是官方最新的 64 位 raspberry os，所以这里选择mihomo-linux-arm64-alpha-0ab73a9.gz下载。因为自身系统的不同与项目 release 的不断更新，这里仅供参考。

此时我们的树莓派连接访问 github 的网络是不稳定的，所以我非常建议在已经科学的环境下载，然后通过 ssh 上传到树莓派。以下的步骤我是在已经科学的 Windows 环境下配合tssh完成。

# 2. 安装clash

```bash
gzip -d -f mihomo-linux-arm64-alpha-0ab73a9.gz
mv mihomo-linux-arm64-alpha-0ab73a9 clash
chmod +x
```

# 3. 初始化clash

```bash
./clash
```

初始化clash后，可以在`~/.config/mihomo`看到初始化的配置文件。

# 4. 配置config.yaml与country.mmdb

将订阅的config.yaml和conuntry.mmdb放到`~/.config/mihomo`中。

# 5. 配置开机自启

移动 clash 到/usr/local/bin/目录，注意命令中 clash 是树莓派中 clash 的路径位置

```bash
sudo mv clash /usr/local/bin/
```

在/etc 目录下创建 clash 目录

```bash
sudo mkdir /etc/clash
```

复制 config.yaml 与 Country.mmdb 文件到/etc/clash 目录

```bash
sudo cp -f ~/.config/mihomo/config.yaml ~/.config/mihomo/country.mmdb /etc/clash
```

创建 systemd 配置文件，实现开机自启

```bash
sudo touch /etc/systemd/system/clash.service
```

编辑配置文件（`sudo vim /etc/systemd/system/clash.service`）写入以下内容

```bash
[Unit]
Description=mihomo Daemon, Another Clash Kernel.
After=network.target NetworkManager.service systemd-networkd.service iwd.service

[Service]
Type=simple
LimitNPROC=500
LimitNOFILE=1000000
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME
Restart=always
ExecStartPre=/usr/bin/sleep 1s
ExecStart=/usr/local/bin/clash -d /etc/clash
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

启用 clash.service 服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable clash.service
sudo systemctl start clash.service
```

# 参考资料

[树莓派安装clash，实现科学上网](https://blog.panda74.fun/blog/practice/raspi/clash)

[mihomo 项目文档](https://wiki.metacubex.one/startup/)


