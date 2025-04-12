

# 1. free5GC的安装

## 1.1 前置条件

根据free5gc在youtube上的教程，在安装free5gc之前准备好ubuntu20.04的虚拟机，在虚拟机中安装go，gcc，git，cmake等前置软件。

```shell
# 1. 安装gcc
sudo apt install gcc
# 2. 安装CMake
wget https://cmake.org/files/v3.27/cmake-3.27.7-linux-x86_64.tar.gz
tar zxvf cmake-3.27.7-linux-x86_64.tar.gz
sudo mv cmake-3.27.7-linux-x86_64 /opt/cmake-3.27.7
sudo ln -sf /opt/cmake-3.27.7/bin/*  /usr/bin/
# 3. 安装git
sudo apt install git
# 4. 安装go
wget https://golang.google.cn/dl/go1.22.6.linux-amd64.tar.gz 
sudo tar -C /usr/local -zxvf go1.18.10.linux-amd64.tar.gz 
mkdir -p ~/go/{bin,pkg,src} 
# The following assume that your shell is bash: 
echo 'export GOPATH=$HOME/go' >> ~/.bashrc 
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc 
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc 
echo 'export GO111MODULE=auto' >> ~/.bashrc 
source ~/.bashrc
# 5. 安装其他必要依赖
sudo apt -y install g++ autoconf libtool pkg-config libmnl-dev libyaml-dev
# 6.安装mogodb
sudo apt -y update
sudo apt -y install mongodb wget git
sudo systemctl start mongodb
```

## 1.2 安装free5gc

使用

```shell
git clone --recursive -b v3.3.0 -j `nproc` https://github.com/free5gc/free5gc.git
# 设置go代理
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

命令，下载free5gc的源码。然后在free5gc的所在目录中使用`make`命令进行编译。

![Imgur](https://i.imgur.com/u0Ok8qd.png)

## 1.3 安装gtp5g

使用`

```shell
git clone https://github.com/free5gc/gtp5g.git
```

下载gtp5g的源码，然后在gtp5g目录下执行`make`命令，然后使用`sudo make install`将gtp5g安装到linux内核中。使用`lsmod | grep gtp`来查看是否已经将gtp加载进内核。正确的结果如下：

![Imgur](https://i.imgur.com/rnCFEwa.png)

## 1.4 WebConsole的构建

### 1.4.1 安装

使用如下命令安装yarn和nodejs

```shell
# 1. 安装yarn
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install -y yarn
# 2. 安装nodejs
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -  
sudo apt-get install -y nodejs
```

使用`make webconsole`构建webconsole，如果出现node版本过低的问题，可以使用nodesource的源来获取版本更高的nodejs。

### 1.4.2 使用webconsole新增UE

在webconsole目录下使用`go run server.go`运行webconsole，在浏览器中输入`192.168.56.101:5000`，可以看到如下窗口。

![Imgur](https://i.imgur.com/qoPF5vp.png)

登陆后出现如下界面

![Imgur](https://i.imgur.com/kc2jcqM.png)

点击Subscribers然后点击new subscribers出现如下界面

![Imgur](https://i.imgur.com/MHhGhed.png)

将Operation Code Type改为OP然后提交，结果如下：

![Imgur](https://i.imgur.com/Ye5FMex.png)

## 1.5 测试安装情况

在free5gc目录中使用` ./test.sh TestRegistration`来进行测试。测试会出现如下错误：

```shell
2023-10-22T08:57:16.332433391Z [ERRO][UPF][Main] UPF Cli Run Error: open Gtp5g: version mismatch: operation not supported
```

出现错误的原因为教程中下载的gtp5g版本过低，在gtp5g的README.md中可以看多最新的gtp5g地址在`https://github.com/free5gc/gtp5g`，在gtp5g的目录下使用`sudo make uninstall`将gtp模块从linux内核中删除，然后安装新版本的gtp5g，然后执行测试。会出现如下错误:

![Imgur](https://i.imgur.com/JDBidud.png)

```shell
2023-10-22T09:27:25.548198506Z [ERRO][UPF][PFCP][LAddr:10.200.200.101:8805] read udp4 10.200.200.101:8805: use of closed network connection
```

在[Test failed after pass message - Q/A - free5GC](https://forum.free5gc.org/t/test-failed-after-pass-message/1920/2)提到了这个问题，给到的解释是It says that it is normal to find out a error after shutdown a test.  If you got a same problem like me,please don’t be afraid and keep your testing.

下面继续进行测试，使用` ./test.sh TestGUTIRegistration`，用于验证 GUTI（Globally Unique Temporary Identifier）注册过程结果如下

![Imgur](https://i.imgur.com/Uc8yBID.png)

使用` ./test.sh TestServiceRequest`验证 Free5GC 是否能够成功处理服务请求消息，结果如下

![Imgur](https://i.imgur.com/0Y5BBTS.png)

使用`./test.sh TestXnHandover`测试移动设备的切换（Handover）流程，包括 S1 切换和 X2 切换。

![Imgur](https://i.imgur.com/071edor.png)

使用`./test.sh TestDeregistration`验证移动设备的去注册（Deregistration）流程，包括去附着、位置更新和注销等步骤。

![Imgur](https://i.imgur.com/7Q97Bpq.png)

使用`./test.sh TestPDUSessionReleaseRequest`验证 PDU（Packet Data Unit）会话释放请求的流程。

![Imgur](https://i.imgur.com/5KwlCIC.png)

使用`./test.sh TestPaging`测试寻呼（Paging）流程，用于唤醒休眠中的 UE（用户设备）。

![Imgur](https://i.imgur.com/xKtnjRi.png)

使用`./test.sh TestN2Handover`验证 N2 接口的切换（Handover）流程。

![Imgur](https://i.imgur.com/huu8I6P.png)

使用`./test.sh TestNon3GPP`验证非 3GPP 接入的功能，包括与 Wi-Fi 或其他非 3GPP网络的集成。

![Imgur](https://i.imgur.com/pVAL6xF.png)

使用`./test.sh TestReSynchronization`验证重新同步（ReSynchronization）流程的功能。

![Imgur](https://i.imgur.com/GPFCJyt.png)

使用`./test_ulcl.sh TestRequestTwoPDUSessions`验证同时请求建立两个 PDU（Packet Data Unit） 会话的流程。

![Imgur](https://i.imgur.com/PLJk9uX.png)

# 2. UERANSIM的安装

## 2.1 安装所需环境

使用如下命令安装UERANSIM所需环境

```shell
sudo apt install make
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```

## 2.2 安装UERANSIM

使用

```shell
git clone https://github.com/aligungr/UERANSIM
```

下载UERANSIM源码，然后再UERANSIM的目录中编译源码。结果如下

![Imgur](https://i.imgur.com/06ecXLI.png)

# 3. free5gc与UERANSIM进行连接

## 3.1 free5gc设置

修改`~/free5gc/config/amfcfg.yaml`文件，将ngapIPList修改为free5gc的IP地址。

![Imgur](https://i.imgur.com/peNypgW.png)

修改`~/free5gc/config/smfcfg.yaml`文件，将`userplaneInformation / upNodes / UPF / interfaces / endpoints`下的IP地址改为free5gc的IP地址。

![Imgur](https://i.imgur.com/tgk9k8l.png)

修改`~/free5gc/config/upfcfg.yaml`文件，将gtpu的IP地址改为free5gc的IP地址

![Imgur](https://i.imgur.com/oVw0fTD.png)

## 3.2 UERANSIM设置

修改`~/UERANSIM/config/free5gc-gnb.yaml`文件，将ngapIp的IP和gtpIp的IP地址改为UERANSIM的地址并将amfConfig的地址改为free5gc的IP地址。

![Imgur](https://i.imgur.com/EW0wZIw.png)

检查`~/UERANSIM/config/free5gc-ue.yaml`文件中UE的设置，查看是否与在free5gc中通过webconsole添加的UE是否一致。

![Imgur](https://i.imgur.com/I58u6E0.png)

## 3.3 测试UERANSIM与free5gc的连通性

首先在free5gc的VM中执行如下命令

```shell
sudo sysctl -w net.ipv4.ip_forward=1 
# <dn_interface>为连接到外网的网卡
sudo iptables -t nat -A POSTROUTING -o <dn_interface> -j MASQUERADE 
sudo systemctl stop ufw
```

在free5gc所在目录下执行`./run.sh`。

然后在UERANSIM的目录下执行`build/nr-gnb -c config/free5gc-gnb.yaml`

![Imgur](https://i.imgur.com/upPVO4D.png)

再启动一个UERANSIM所在VM的终端，然后再UERANSIM的目录下执行`sudo build/nr-ue -c config/free5gc-ue.yaml`

![Imgur](https://i.imgur.com/FOXk1RD.png)

再启动一个UERANSIM所在VM的终端，使用ping命令来确定free5gc是否存活。

![Imgur](https://i.imgur.com/zX65El1.png)

然后使用`ifconfig`可以看到`uesimtun0`被创建。

![Imgur](https://i.imgur.com/0Z7zQG5.png)

使用`ping -I uesimtun0 baidu.com`通过uesimtun0来访问外网。

![Imgur](https://i.imgur.com/g89iCIQ.png)

可以看到能够正确访问到baidu.com，此时free5gc正常运行。

# 4. 容器化

使用

```shell
git clone https://github.com/free5gc/free5gc-compose.git
```

下载free5gc-compose项目，在free5gc-compose\base目录下，使用

```bash
docker compose pull
```

拉取在dockerhub中的镜像。拉去后docker镜像如下所示：

![Imgur](https://i.imgur.com/onB1xNk.png)

然后使用`docker compose up`命令可以启动所有容器。结果如下

![Imgur](https://i.imgur.com/XnQQMnb.png)

这时可以在浏览器中看到webconsole。

![Imgur](https://i.imgur.com/rRW4tRg.png)

根据docker-compose.yaml文件可以看到，启动的容器中free5gc中的配置文件和free5gc-compose目录下文件的对应关系。

![Imgur](https://i.imgur.com/xgXzeFY.png)

![Imgur](https://i.imgur.com/AoUjR4i.png)

![Imgur](https://i.imgur.com/JDeI0I7.png)

![Imgur](https://i.imgur.com/MGqpdXz.png)

修改ueransim中的uecfg.yaml，使其与创建的UE属性相同。创建一个新的UE。

![Imgur](https://i.imgur.com/3x4jUpa.png)

然后进入ueransim所在的容器，执行`./nr-ue -c config/uecfg.yaml`可以得到如下结果:

![Imgur](https://i.imgur.com/zbjXOAS.png)

然后使用`ping -I uesimtun0 baidu.com`测试是否可以联通外网。

![Imgur](https://i.imgur.com/tExJdrC.png)

# 5. ULCL

## 5.1 网络架构

![github](https://github.com/s5uishida/free5gc_ueransim_ulcl_sample_config/raw/main/images/network-overview.png)

网络架构如图，创建五个虚拟机。虚拟机的配置如下

| VM # | SW & Role                      | IP address        | OS           | Memory (Min) | HDD (Min) |
| ---- | ------------------------------ | ----------------- | ------------ | ------------ | --------- |
| VM1  | free5GC  5GC C-Plane           | 192.168.56.141/24 | Ubuntu 20.04 | 2GB          | 20GB      |
| VM2  | free5GC  5GC U-Plane (I-UPF)   | 192.168.56.142/24 | Ubuntu 20.04 | 1GB          | 20GB      |
| VM3  | free5GC  5GC U-Plane (PSA-UPF) | 192.168.56.143/24 | Ubuntu 20.04 | 1GB          | 20GB      |
| VM4  | UERANSIM RAN (gNodeB)          | 192.168.56.131/24 | Ubuntu 20.04 | 1GB          | 10GB      |
| VM5  | UERANSIM UE                    | 192.168.56.132/24 | Ubuntu 20.04 | 1GB          | 10GB      |

## 5.2 修改配置文件

### 5.2.1 修改C-Plane的配置

修改VM1中的`/free5gc/config/amfcfg.yaml`文件，修改后如下所示

```yaml
info:
  version: 1.0.9
  description: AMF initial local configuration

configuration:
  amfName: AMF # the name of this AMF
  ngapIpList:  # the IP list of N2 interfaces on this AMF
    - 192.168.56.141 # 127.0.0.18
  ngapPort: 38412 # the SCTP port listened by NGAP
  sbi: # Service-based interface information
    scheme: http # the protocol for sbi (http or https)
    registerIPv4: 127.0.0.18 # IP used to register to NRF
    bindingIPv4: 127.0.0.18  # IP used to bind the service
    port: 8000 # port used to bind the service
    tls: # the local path of TLS key
      pem: cert/amf.pem # AMF TLS Certificate
      key: cert/amf.key # AMF TLS Private key
  serviceNameList: # the SBI services provided by this AMF, refer to TS 29.518
    - namf-comm # Namf_Communication service
    - namf-evts # Namf_EventExposure service
    - namf-mt   # Namf_MT service
    - namf-loc  # Namf_Location service
    - namf-oam  # OAM service
  servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
    # <GUAMI> = <MCC><MNC><AMF ID>
    - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
      amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
  supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
    - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
      tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
  plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
    - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
      snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
        - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
          sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
        - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
          sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
  supportDnnList:  # the DNN (Data Network Name) list supported by this AMF
    - internet
  nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
  security:  # NAS security parameters
    integrityOrder: # the priority of integrity algorithms
      - NIA2
      # - NIA0
    cipheringOrder: # the priority of ciphering algorithms
      - NEA0
      # - NEA2
  networkName:  # the name of this core network
    full: free5GC
    short: free
  ngapIE: # Optional NGAP IEs
    mobilityRestrictionList: # Mobility Restriction List IE, refer to TS 38.413
      enable: true # append this IE in related message or not
    maskedIMEISV: # Masked IMEISV IE, refer to TS 38.413
      enable: true # append this IE in related message or not
    redirectionVoiceFallback: # Redirection Voice Fallback IE, refer to TS 38.413
      enable: false # append this IE in related message or not
  nasIE: # Optional NAS IEs
    networkFeatureSupport5GS: # 5gs Network Feature Support IE, refer to TS 24.501
      enable: true # append this IE in Registration accept or not
      length: 1 # IE content length (uinteger, range: 1~3)
      imsVoPS: 0 # IMS voice over PS session indicator (uinteger, range: 0~1)
      emc: 0 # Emergency service support indicator for 3GPP access (uinteger, range: 0~3)
      emf: 0 # Emergency service fallback indicator for 3GPP access (uinteger, range: 0~3)
      iwkN26: 0 # Interworking without N26 interface indicator (uinteger, range: 0~1)
      mpsi: 0 # MPS indicator (uinteger, range: 0~1)
      emcN3: 0 # Emergency service support indicator for Non-3GPP access (uinteger, range: 0~1)
      mcsi: 0 # MCS indicator (uinteger, range: 0~1)
  t3502Value: 720  # timer value (seconds) at UE side
  t3512Value: 3600 # timer value (seconds) at UE side
  non3gppDeregTimerValue: 3240 # timer value (seconds) at UE side
  # retransmission timer for paging message
  t3513:
    enable: true     # true or false
    expireTime: 6s   # default is 6 seconds
    maxRetryTimes: 4 # the max number of retransmission
  # retransmission timer for NAS Deregistration Request message
  t3522:
    enable: true     # true or false
    expireTime: 6s   # default is 6 seconds
    maxRetryTimes: 4 # the max number of retransmission
  # retransmission timer for NAS Registration Accept message
  t3550:
    enable: true     # true or false
    expireTime: 6s   # default is 6 seconds
    maxRetryTimes: 4 # the max number of retransmission
  # retransmission timer for NAS Authentication Request/Security Mode Command message
  t3560:
    enable: true     # true or false
    expireTime: 6s   # default is 6 seconds
    maxRetryTimes: 4 # the max number of retransmission
  # retransmission timer for NAS Notification message
  t3565:
    enable: true     # true or false
    expireTime: 6s   # default is 6 seconds
    maxRetryTimes: 4 # the max number of retransmission
  # retransmission timer for NAS Identity Request message
  t3570:
    enable: true     # true or false
    expireTime: 6s   # default is 6 seconds
    maxRetryTimes: 4 # the max number of retransmission
  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
  sctp: # set the sctp server setting <optinal>, once this field is set, please also add maxInputStream, maxOsStream, maxAttempts, maxInitTimeOut
    numOstreams: 3 # the maximum out streams of each sctp connection
    maxInstreams: 5 # the maximum in streams of each sctp connection
    maxAttempts: 2 # the maximum attempts of each sctp connection
    maxInitTimeout: 2 # the maximum init timeout of each sctp connection
  defaultUECtxReq: false # the default value of UE Context Request to decide when triggering Initial Context Setup procedure

logger: # log output setting
  enable: true # true or false
  level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
  reportCaller: false # enable the caller report or not, value: true or false
```

其中的PLMN由全球唯一的PLMN代码标识，该代码由MCC（移动国家代码）和MNC（移动网络代码）组成。因此，它是一个五到六位数的数字，用于识别一个国家和该国家的移动网络运营商，通常以001-01或001–001的形式表示。这里修改为001和01是测试环境的意思。ngapIpList，根据注释是N2接口的IP地址，N2接口是AMF与RAN的接口。

修改`/free5gc/config/smfcfg.yaml`，修改后的配置文件如下：

```yaml
info:
  version: 1.0.7
  description: SMF initial local configuration

configuration:
  smfName: SMF # the name of this SMF
  sbi: # Service-based interface information
    scheme: http # the protocol for sbi (http or https)
    registerIPv4: 127.0.0.2 # IP used to register to NRF
    bindingIPv4: 127.0.0.2 # IP used to bind the service
    port: 8000 # Port used to bind the service
    tls: # the local path of TLS key
      key: cert/smf.key # SMF TLS Certificate
      pem: cert/smf.pem # SMF TLS Private key
  serviceNameList: # the SBI services provided by this SMF, refer to TS 29.502
    - nsmf-pdusession # Nsmf_PDUSession service
    - nsmf-event-exposure # Nsmf_EventExposure service
    - nsmf-oam # OAM service
  snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
      dnnInfos: # DNN information list
        - dnn: internet # Data Network Name
          dns: # the IP address of DNS
            ipv4: 8.8.8.8
            ipv6: 2001:4860:4860::8888
    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
      dnnInfos: # DNN information list
        - dnn: internet # Data Network Name
          dns: # the IP address of DNS
            ipv4: 8.8.8.8
            ipv6: 2001:4860:4860::8888
  plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
  pfcp: # the IP address of N4 interface on this SMF (PFCP)
    # addr config is deprecated in smf config v1.0.3, please use the following config
    nodeID: 192.168.56.141 # the Node ID of this SMF
    listenAddr: 192.168.56.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
    externalAddr: 192.168.56.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
  userplaneInformation: # list of userplane information
    upNodes: # information of userplane node (AN or UPF)
      gNB1: # the name of the node
        type: AN # the type of the node (AN or UPF)
      I-UPF: # the name of the node
        type: UPF # the type of the node (AN or UPF)
        nodeID: 192.168.56.142 # the Node ID of this UPF
        addr: 192.168.56.142 # the IP/FQDN of N4 interface on this UPF (PFCP)
        sNssaiUpfInfos: # S-NSSAI information list for this UPF
          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
            dnnUpfInfoList: # DNN information list for this S-NSSAI
              - dnn: internet
        interfaces: # Interface list for this UPF
          - interfaceType: N3 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - 192.168.56.142
            networkInstances:
              - internet # Data Network Name (DNN)
          - interfaceType: N9 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - 192.168.56.142
            networkInstances:
              - internet # Data Network Name (DNN)
      PSA-UPF:  # the name of the node
        type: UPF # the type of the node (AN or UPF)
        nodeID: 192.168.56.143 # the IP/FQDN of N4 interface on this UPF (PFCP)
        sNssaiUpfInfos: # S-NSSAI information list for this UPF
          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
            dnnUpfInfoList: # DNN information list for this S-NSSAI
              - dnn: internet
                pools:
                  - cidr: 10.60.0.0/16
        interfaces: # Interface list for this UPF
          - interfaceType: N9 # the type of the interface (N3 or N9)
            endpoints: # the IP address of this N3/N9 interface on this UPF
              - 192.168.56.143
            networkInstances:
              - internet # Data Network Name (DNN)
    links: # the topology graph of userplane, A and B represent the two nodes of each link
      - A: gNB1
        B: I-UPF
      - A: I-UPF
        B: PSA-UPF
  # retransmission timer for pdu session modification command
  t3591:
    enable: true     # true or false
    expireTime: 16s   # default is 6 seconds
    maxRetryTimes: 3 # the max number of retransmission
  # retransmission timer for pdu session release command
  t3592:
    enable: true     # true or false
    expireTime: 16s   # default is 6 seconds
    maxRetryTimes: 3 # the max number of retransmission
  nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
  ulcl: true
  #urrPeriod: 10 # default usage report period in seconds
  #urrThreshold: 1000 # default usage report threshold in bytes

logger: # log output setting
  enable: true # true or false
  level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
  reportCaller: false # enable the caller report or not, value: true or false
```

修改`free5gc/config/uerouting.yaml`文件，修改后文件为

```yaml
info:
  version: 1.0.7
  description: Routing information for UE

ueRoutingInfo: # the list of UE routing information
  UE1: # Group Name
    members:
    - imsi-208930000000003 # Subscription Permanent Identifier of the UE
    topology: # Network topology for this group (Uplink: A->B, Downlink: B->A)
    # default path derived from this topology
    # node name should be consistent with smfcfg.yaml
      - A: gNB1
        B: I-UPF
      - A: I-UPF
        B: PSA-UPF
    specificPath:
      - dest: 8.8.8.8/32 # the destination IP address on Data Network (DN)
        # the order of UPF nodes in this path. We use the UPF's name to represent each UPF node.
        # The UPF's name should be consistent with smfcfg.yaml
        path: [I-UPF]

  UE2: # Group Name
    members:
    - imsi-208930000007486 # Subscription Permanent Identifier of the UE
    topology: # Network topology for this group (Uplink: A->B, Downlink: B->A)
    # default path derived from this topology
    # node name should be consistent with smfcfg.yaml
      - A: gNB1
        B: BranchingUPF
      - A: BranchingUPF
        B: AnchorUPF1
    specificPath:
      - dest: 10.0.0.11/32 # the destination IP address on Data Network (DN)
        # the order of UPF nodes in this path. We use the UPF's name to represent each UPF node.
        # The UPF's name should be consistent with smfcfg.yaml
        path: [BranchingUPF, AnchorUPF2]

routeProfile: # Maintains the mapping between RouteProfileID and ForwardingPolicyID of UPF
  MEC1: # Route Profile identifier
    forwardingPolicyID: 10 # Forwarding Policy ID of the route profile

pfdDataForApp: # PFDs for an Application
  - applicationId: edge # Application identifier
    pfds: # PFDs for the Application
      - pfdID: pfd1 # PFD identifier
        flowDescriptions: # Represents a 3-tuple with protocol, server ip and server port for UL/DL application traffic
          - permit out ip from 10.60.0.1 8080 to any
```

这里需要记住UE1的imsi值，下面再Webconsole中设置UE时需要用到。

### 5.2.2 修改I-UPF的配置

修改VM2的`free5gc/config/upfcfg.yaml`文件，配置文件修改后如下：

```yaml
# The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
version: 1.0.3
description: UPF initial local configuration

# The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
pfcp:
  addr: 192.168.56.143   # IP addr for listening
  nodeID: 192.168.56.143 # External IP or FQDN can be reached
  retransTimeout: 1s # retransmission timeout
  maxRetrans: 3 # the max number of retransmission

gtpu:
  forwarder: gtp5g
  # The IP list of the N3/N9 interfaces on this UPF
  # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
  ifList:
    - addr: 192.168.56.142
      type: N3
    - addr: 192.168.56.142
      type: N9
      # name: upf.5gc.nctu.me
      # ifname: gtpif
      # mtu: 1400

# The DNN list supported by UPF
dnnList:
  - dnn: internet # Data Network Name
    cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
    # natifname: eth0

logger: # log output setting
  enable: true # true or false
  level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
  reportCaller: false # enable the caller report or not, value: true or fals
```

### 5.2.3 修改PSA-UPF的配置

修改VM3的`free5gc/config/upfcfg.yaml`文件，配置文件修改后如下：

```yaml
version: 1.0.3
description: UPF initial local configuration

# The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
pfcp:
  addr: 192.168.56.143   # IP addr for listening
  nodeID: 102.168.56.143 # External IP or FQDN can be reached
  retransTimeout: 1s # retransmission timeout
  maxRetrans: 3 # the max number of retransmission

gtpu:
  forwarder: gtp5g
  # The IP list of the N3/N9 interfaces on this UPF
  # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
  ifList:
    - addr: 192.168.56.143
      type: N9
      # name: upf.5gc.nctu.me
      # ifname: gtpif
      # mtu: 1400

# The DNN list supported by UPF
dnnList:
  - dnn: internet # Data Network Name
    cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
    # natifname: eth0

logger: # log output setting
  enable: true # true or false
  level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
  reportCaller: false # enable the caller report or not, value: true or false
```

### 5.2.4 修改gNB的配置

修改VM4的`UERANSIM/config/free5gc-gnb.yaml`配置文件，修改后如下：

```yaml
mcc: '208'          # Mobile Country Code value
mnc: '93'           # Mobile Network Code value (2 or 3 digits)

nci: '0x000000010'  # NR Cell Identity (36-bit)
idLength: 32        # NR gNB ID length in bits [22...32]
tac: 1              # Tracking Area Code

linkIp: 192.168.56.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
ngapIp: 192.168.56.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
gtpIp: 192.168.56.131    # gNB's local IP address for N3 Interface (Usually same with local IP)

# List of AMF address information
amfConfigs:
  - address: 192.168.56.141
    port: 38412

# List of supported S-NSSAIs by this gNB
slices:
  - sst: 0x1
    sd: 0x010203

# Indicates whether or not SCTP stream number errors should be ignored.
ignoreStreamIds: true
```

### 5.2.5 修改UE的配置

修改`UERANSIM\config\free5gc-ue.yaml`文件

```yaml
# IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
supi: 'imsi-208930000000003'
# Mobile Country Code value of HPLMN
mcc: '001'
# Mobile Network Code value of HPLMN (2 or 3 digits)
mnc: '01'
# SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
protectionScheme: 0
# Home Network Public Key for protecting with SUCI Profile A
homeNetworkPublicKey: '5a8d38864820197c3394b92613b20b91633cbd897119273bf8e4a6f4eec0a650'
# Home Network Public Key ID for protecting with SUCI Profile A
homeNetworkPublicKeyId: 1
# Routing Indicator
routingIndicator: '0000'

# Permanent subscription key
key: '8baf473f2f8fd09487cccbd7097c6862'
# Operator code (OP or OPC) of the UE
op: '8e27b6af0e692e750f32667a3b14605d'
# This value specifies the OP type and it can be either 'OP' or 'OPC'
opType: 'OP'
# Authentication Management Field (AMF) value
amf: '8000'
# IMEI number of the device. It is used if no SUPI is provided
imei: '356938035643803'
# IMEISV number of the device. It is used if no SUPI and IMEI is provided
imeiSv: '4370816125816151'

# List of gNB IP addresses for Radio Link Simulation
gnbSearchList:
  - 192.168.56.131

# UAC Access Identities Configuration
uacAic:
  mps: false
  mcs: false

# UAC Access Control Class
uacAcc:
  normalClass: 0
  class11: false
  class12: false
  class13: false
  class14: false
  class15: false

# Initial PDU sessions to be established
sessions:
  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 0x01
      sd: 0x010203

# Configured NSSAI for this UE by HPLMN
configured-nssai:
  - sst: 0x01
    sd: 0x010203

# Default Configured NSSAI for this UE
default-nssai:
  - sst: 1
    sd: 1

# Supported integrity algorithms by this UE
integrity:
  IA1: true
  IA2: true
  IA3: true

# Supported encryption algorithms by this UE
ciphering:
  EA1: true
  EA2: true
  EA3: true

# Integrity protection maximum data rate for user plane
integrityMaxRate:
  uplink: 'full'
  downlink: 'full'
```

注意supi的值要与在C-Plane中的UE设置相对应。

## 5.3 使用WebConsole设置UE

使用WebConsole创建如下UE。

![Imgur](https://i.imgur.com/cELW3F0.png)

## 5.4 运行

使用如下脚本运行C-Plane

```shell
#!/usr/bin/env bash

PID_LIST=()

NF_LIST="amf udr pcf udm nssf ausf"

export GIN_MODE=release

./bin/nrf &
PID_LIST+=($!)
sleep 1

./bin/smf -c config/smfcfg.yaml -u config/uerouting.yaml &
PID_LIST+=($!)
sleep 1

for NF in ${NF_LIST}; do
    ./bin/${NF} &
    PID_LIST+=($!)
    sleep 1
done

function terminate()
{
    sudo kill -SIGTERM ${PID_LIST[${#PID_LIST[@]}-2]} ${PID_LIST[${#PID_LIST[@]}-1]}
    sleep 2
}

trap terminate SIGINT
wait ${PID_LIST}
```

在I-UPF和PSA-UPF的虚拟机中使用如下命令启动UPF

```shell
sudo bin/upf
```

在gNb的虚拟机中使用如下命令，来模拟gNodeB

```shell
build/nr-gnb -c config/free5gc-gnb.yaml
```

在UE的虚拟机中使用如下命令来模拟UE

```shell
sudo build/nr-ue -c config/free5gc-ue.yaml
```

可以看到C-Plane的webconsole中打印结果如下：

![Imgur](https://i.imgur.com/x2sZQfw.png)

I-UPF和PSA-UPF的结果如下：

![Imgur](https://i.imgur.com/xhxjfeH.png)

gNB的终端打印结果如下：

![Imgur](https://i.imgur.com/wLPEQGz.png)

UE的终端打印结果如下：

![Imgur](https://i.imgur.com/XzsOHr0.png)

## 5.5 验证分流

在I-UPF和PSA-UPF中各自使用如下命令

```shell
iptables -t nat -A POSTROUTING -s 10.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

在I-UPF和PSA-UPF中使用tcpdump来监听虚拟出的upfgtp网卡

```shell
tcpdump -i upfgtp -n
```

在UE中使用如下命令来PING baidu.com

```shell
ping baidu.com -I uesimtun0 -n
```

可以在PSA-UPF的tcpdump中看到如下信息：

![Imgur](https://i.imgur.com/0az8IBo.png)

而在I-UPF中没有看到信息，说明通过PSA-UPF来链接到外网，

使用如下命令来PING 8.8.8.8

```shell
ping 8.8.8.8 -I uesimtun0 -n
```

可以在I-UPF中看到信息，而PSA-UPF中没有看到信息。

![Imgur](https://i.imgur.com/ouFE2Zm.png)
