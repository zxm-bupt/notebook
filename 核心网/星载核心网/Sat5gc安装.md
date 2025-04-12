# 1. 环境配置

```shell
# 1. 克隆代码
git clone https://codeup.aliyun.com/60a52fd5b8301d20d58b14cb/sat5gc_total.git
git clone https://codeup.aliyun.com/60a52fd5b8301d20d58b14cb/user-plane/upf-pfcp-server.git
git clone https://codeup.aliyun.com/60a52fd5b8301d20d58b14cb/user-plane/upf-xdp.git
git clone https://codeup.aliyun.com/60a52fd5b8301d20d58b14cb/modified_sat5gc.git
mv modified_sat5gc/ sat5gc_total/
mv upf-pfcp-server/ sat5gc-total/
mv upf-xdp/ sat5gc-total/
# 2. 安装控制面依赖
sudo apt install gcc make autoconf libtool
# 3. 安装用户面依赖
sudo apt install clang llvm libelf-dev gcc-multilib 
sudo apt install linux-tools-$(uname -r)
sudo apt install linux-headers-$(uname -r)
# 4. 安装UERANSIM依赖
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
# 5. 安装go
wget https://dl.google.com/go/go1.16.5.linux-amd64.tar.gz
sudo rm -rf /usr/local/go   # 该指令会删除原先版本的go
sudo tar -C /usr/local -zxvf go1.16.5.linux-amd64.tar.gz
mkdir -p ~/go/{bin,pkg,src}
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
echo 'export GO111MODULE=auto' >> ~/.bashrc
source ~/.bashrc
```

# 2. 编译项目

使用`build.sh`脚本自动编译项目，编译成功的结果如下图所示：

![Imgur](https://i.imgur.com/T0M2zMR.png)

编译UERANSIM。使用如下命令编译UERANSIM

```shell
cd UERANSIM
make
```

# 3. 运行项目

```shell
# 1. 启动项目
sudo ./bootstrap.sh
# 2. 终止项目
sudo ./stop.sh
```

运行成功后也可看到虚拟化的网卡

![Imgur](https://i.imgur.com/H7s63jZ.png)

可以进行ping通20.20.20.2

![Imgur](https://i.imgur.com/UaEDAnB.png)

但是ping不通baidu

![Imgur](https://i.imgur.com/vowNnkb.png)
