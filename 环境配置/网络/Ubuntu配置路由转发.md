# 背景

今天想让一台装有Ubuntu20.04系统的服务器在局域网中起到路由器的功能。
**目标：** 让局域网中所有的电脑都通过这台服务器连接外网。

服务器网卡:

* eth0：192.168.1.xxx/24 连接局域网 
* eth1：117.22.22.xxx/24 连接外网

# 基本配置

1、在服务器上开启内核路由转发参数

临时生效：

```shell
echo "1" > /proc/sys/net/ipv4/ip_forward
```

永久生效的话，需要修改`/etc/sysctl.conf`：

```shell
net.ipv4.ip_forward = 1
```

执行sysctl -p马上生效

开启成功：执行该条命令，`cat /proc/sys/net/ipv4/ip_forward`,如果输出1则说明开启成功！

2、服务器开启iptables转发

```shell
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

执行之后，只会临时起效，重启之后就失效了。

永久保存：`iptables-save > /etc/sysconfig/iptables`

**TIP：注意对应网卡。**

3、修改其他电脑的网关
将局域网中想要访问外网的电脑的网关改成服务器的局域网IP192.168.1.xxx。
