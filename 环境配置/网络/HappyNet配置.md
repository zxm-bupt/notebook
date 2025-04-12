# 1. 服务器happynet配置

```shell
# 1. 下载安装包
wget "https://storage.happyn.cn/d/happyn/release/happynserver/happynserver-linux-dynamic-all-latest.tar.gz?sign=KDaAfo-lfSbacOsU6PNZid7-3rwOJnRXLjaqxWDPhpQ=:0" -O happynserver-linux-dynamic-all-latest.tar.gz
# 2. 解压软件
tar zxvf happynserver-linux-dynamic-all-latest.tar.gz
# 3.进入解压文件夹
cd happynserver
# 4.拷贝可执行文件
sudo cp bin/x64/happynsupernode /usr/local/bin/
sudo chmod +x /usr/local/bin/happynsupernode 
# 5.拷贝配置文件
sudo  cp etc/happynserver.conf /etc/
# 6.拷贝系统服务文件
sudo cp service/happynserver.service /etc/systemd/system/
# 7.载入服务
sudo systemctl daemon-reload
# 8.如果要设置为开机启动，请执行
sudo systemctl enable happynserver
```

配置文件

配置文件的格式非常简单: 基本格式：`服务ID 子网`

1. 服务ID: 用于验证客户端的ID，在下面的例子里，我们以`happyn001`为例
2. 子网: 一般是`10.x.x.x/24` 段的ip池; 在下面的例子里面，我们以`192.168.100.0/24`为例

```bash
happyn001 192.168.100.0/24
```

# 2. linux happynet客户端配置

```shell
# 1.下载安装包
wget https://storage.happyn.cn/d/happyn/release/linux/happynet-linux-x86-x64-dynamic-all-latest.tar.gz?sign=Wk_TrI-rj2PfKJjLMljk1J7vWsC8LxjEYQn_IoSH_T0=:0 -O happynet-linux-x86-x64-dynamic-all-latest.tar.gz
# 2.解压文件
tar zxvf happynet-linux-x86-x64-dynamic-all-latest.tar.gz
# 3.进入解压文件夹
cd happynlinux
# 4.拷贝可执行文件
sudo cp bin/x64/happynet /usr/local/bin/
sudo chmod +x /usr/local/bin/happynet
# 5.拷贝配置文件
sudo  cp etc/happynet.conf /etc/
# 6.拷贝系统服务文件
sudo cp service/happynet.service /etc/systemd/system/
# 7.载入服务
sudo systemctl daemon-reload
# 8.如果要设置为开机启动，请执行
sudo systemctl enable happynet
```

配置文件

```shell
$ sudo vim /etc/happynet.conf

#==============================
#必改参数:这里填入您获取的服务ID
-c=happyn001

#必改参数:这里填入您获取的服务密钥
-k=happynet 

#这里填入您指定的网卡MAC地址，如果不需要手工指定的话，直接用 `#` 注释掉即可
#-m=xx:xx:xx:xx:xx:xx 

#必改参数:这里填入您的合法子网IP地址，这个地址不能与其它连入设备相冲突，如果想要自动获取ip，直接用 `#` 注释掉即可
-a=10.251.56.100 

#必改参数:这里填入您的 `服务器地址:端口`
-l=vip00.happyn.cc:30002  

#启用数据压缩，节约传输流量，建议开启
-z1

#本地转发，默认关闭                                                                                                                                                                                                                                                                  
#-r  

#指定加密方法，联入同一网络的机器必须采用同一种加密方法
#-A2代表Twofish, -A3代表AES (happyn指定默认加密选项)
#-A4代表ChaCha20, -A5代表Speck-CTR
-A3

#可选参数：指定打洞对外监听端口，当您需要配置防火墙时，可以固定此端口，这样方便配置防火墙规则(UDP)
-p=0.0.0.0:55644

#调试信息输出，默认关闭                                                                                                                                                                                                                                                              
#-vvv
```

# 3. windows客户端配置

![Imgur](https://i.imgur.com/xMxAaJw.png)

# 4. 用户手册

[💡 happyn-您的组网专家 - 飞书云文档 (feishu.cn)](https://happyncn.feishu.cn/wiki/wikcnqJgp8j6qSKMPS9DdyrhLHc)
