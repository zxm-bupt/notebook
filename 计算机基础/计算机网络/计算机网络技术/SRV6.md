# 1. SRv6的定义

SRv6（Segment Routing over IPv6）是一种基于IPv6网络的源路由技术，旨在提供更灵活、高效和智能的网络管理和流量控制。SRv6通过在IPv6报文头部添加Segment Routing扩展头（SRH），实现了逐段路由，从而对数据流进行精确路径控制。同时，SRv6利用IPv6强大的扩展能力可以满足数据中心互联、5G网络切片等不同应用场景中的业务需求。SRv6技术为未来SDN网络架构的演进和发展奠定了基础。

# 2. SRv6的产生背景(必要性)

SRv6技术的产生是由多种因素共同驱动的结果：

* IPv4地址资源耗尽的驱动：作为IPv4的替代者，IPv6技术标准在上世纪90年代就已经提出，但当时IPv4加MPLS的网络已经相当成熟，基于成本等原因的考虑，网络服务提供商、企业和设备商等都没有足够的动力推动IPv4向IPv6网络的升级。直到21世纪10年代，IPv4地址耗尽的问题已经迫在眉睫，全面部署和升级IPv6也相应地加快了步伐。升级IPv6成为业界各厂商的共识。

* 新业务需求的驱动：随着5G和云计算的发展，各种各样的新业务不断出现，海量新业务对于地址和链接的需求不断增长，IPv4地址资源和MPLS的可扩展能力不足的缺点越发凸显。固定长度 [MPLS](https://info.support.huawei.com/info-finder/encyclopedia/zh/MPLS.html) 标签和有限的IPv4报文头中无法添加更多的扩展信息，以IPv4和MPLS为主的数据转发平面已经不能满足业务差异化标识等需求。IPv6的强大扩展能力可以满足新业务需求，IPv6网络的快速普及成为发展SRv6技术的基础。

* SRv6相较于OpenFlow能更好的兼容当前的分布式网络 **(为什么要兼容当前的分布式网络，不能激进一点重新做一套新的吗)**：传统网络中，每台设备既需要单独维护控制平面的路由协议和信令协议，也需要处理数据平面的报文转发，这种分布式的网络架构在跨域等大规模组网场景下的部署和运维极其复杂，网络的OPEX（Operating Expense，运维成本）较高。21世纪10年代新出现的SDN（Software Defined Network，软件定义网络）作为一种集中式的网络架构将控制平面和数据平面分离，并且实现网络设备的统一管理，很好地解决了上述问题。使用SRv6技术的SDN网络既可以兼容现有的分布式网络架构，又完美符合集中式架构SDN网络的思想，相较于使用OpenFlow等协议的SDN网络更适合现有网络的平滑演进。随着网络架构的变革和演进，SRv6成为了网络数据平面首选技术。

# 3. SRv6的技术优势(先进性)

SRv6技术作为一种基于IPv6网络的源路由技术，相较于传统网络技术具有一系列显著的优势和技术价值。

* 具有网络编程能力：SRv6技术具有强大的可编程能力，例如，通过SRH扩展头中的SID列表实现网络路径编排。又如，SRv6定义了多种类型的SID，根据不同SID的转发行为指令，能够执行不同的转发行为，从而满足不同业务诉求。它基于SDN架构设计，可以将应用程序信息带入到网络中，实现全局信息的网络调度和优化。

* 简化网络协议，部署运维简单：SRv6技术简化了网络协议，SRv6基于IGP和BGP扩展实现，无需使用LDP/RSVP-TE协议和MPLS标签，使得部署更简单。并且，网络中只需源节点控制和维护路径信息，其他节点不需要维护路径信息，大大简化了维护工作。

* 具备强大的扩展能力和兼容性：SRv6技术基于Native IPv6进行转发，充分利用了IPv6地址空间的扩展性，与普通IPv6设备兼容性良好，可以支持业务快速上线，平滑演进。此外，可以根据实际需要新定义SRv6 SID类型，具有很好的扩展性。

* 强大的流量工程能力：SRv6技术能够实现灵活的流量工程，支持多种路径来源和路径计算方式，可以从多路径中选择最优路径，支持基于权重的流量负载分担、路径间的热备份和快速切换，有助于网络运营商更好地满足不同业务场景的需求。

总之，SRv6技术能够提供更高效、灵活的网络管理方式，为未来网络架构的发展奠定了基础。

# 4. SRv6的工作方式

在介绍工作方式之前，首先介绍一下SRv6的SID。SID（Segment Identifier，段标识）是SRv6技术中的关键概念，它是一个128位的IPv6地址。SRv6 SID由Locator、Function、Arguments和MBZ（Must be zero，全0补位）四部分组成，可以自定义每部分的长度，其中Locator、Function部分为必选字段，而Arguments和MBZ为可选字段：

* locator：标识SID所属的IPv6网段。SRv6节点生成SID后，通过IGP等路由协议将Locator标识的IPv6网段发布到网络中，帮助其他设备将报文转发到该SRv6节点，因此Locator用于SRv6节点的路由和寻址。

* Function：标识与SID绑定的本地操作指令。报文通过Locator路由转发到SRv6节点后，由SRv6节点查找报文中Function字段值所对应的设备本地操作指令，SRv6节点再根据指令，执行对应的转发动作。设备的本地操作指令又称为SRv6 Endpoint Behavior，SRv6 Endpoint Behavior的类型非常丰富，业界不同厂商均可定义新的SRv6 Endpoint Behavior，并由IANA（Internet Assigned Numbers Authority，互联网号码分配局）统一分配取值。网络管理员只需要为SID的Function字段绑定不同类型的本地操作指令即可实现对报文转发行为的控制，达到对网络编程的效果。

* Arguments：是对Function的补充，定义报文的流和服务等信息，在SRv6 SID中为可选字段。

* MBZ：当Locator、Function和Arguments的位数之和小于128bits时，最低若干位使用0补齐。

<img src="http://123.57.190.49:12121/api/image/4LB6R24Z.png" title="" alt="SID" data-align="center">

## 4.1 基于SID的路径编程

<img src="http://123.57.190.49:12121/api/image/PHDJV24D.png" title="" alt="SRH结构" data-align="center">

<img src="http://123.57.190.49:12121/api/image/2J6DL4JT.png" title="" alt="SID转发过程" data-align="center">

在SRv6网络中，SID被添加到IPv6报文的Segment Routing扩展头（SRH）中。SID在SRH中的SID列表中按照顺序排列，即可指示数据包在网络中的行进路径。当数据包到达一个SRv6节点时，SRv6节点判断当前的IPv6目的地址是否等于当前表中的SRv6 SID，如果等于SRv6 SID，则根据本地SID对应的SRv6 Endpoint Behavior执行相应的转发动作，并将SRH中的下一个SID设置为目标地址，使数据包继续沿着指定的路径前进。这种机制使得SRv6网络能够实现精确的流量控制和路径编程管理。SID在从n-1开始匹配，到0结束。

## 4.2 SRv6 BE vs SRv6 TE

SRv6 BE（Best Effort）和SRv6 TE（Traffic Engineering）是SRv6技术中两种不同的业务转发策略。以下分别对它们进行简要介绍，并对比它们的优劣势和适用场景。

**SRv6 BE（Best Effort）**

SRv6 BE是一种尽力而为转发策略。它是由IGP根据业务的目的地址计算出一个SRv6 SID，报文将该SRv6 SID作为目的地址，并查找路由表转发到终点。从SRv6报文封装的角度看，在这种模式下，SRv6报文中不存在SRH扩展头，因此无法进行流量工程和路径控制。从数据转发的角度看，SRv6数据包在网络中的传输不提供任何质量保证，只根据网络状况和IGP自动选择最佳路径。SRv6 BE适用于对网络性能要求较低、可容忍一定程度的延迟和丢包的场景。

**SRv6 TE（Traffic Engineering）**

SRv6 TE则是一种基于流量工程原则的转发策略，在这种模式下，SRv6报文封装时存在SRH扩展头，可以通过SRH扩展头的SID列表来控制数据包在网络中的路径，实现对网络资源的优化利用。SRv6 TE适用于对网络性能要求较高、需要保证服务质量的场景，同时，SRv6 TE也更加契合SDN网络架构的要求。SRv6 TE可以通过创建Tunnel隧道接口的方式来实现，也可以通过SRv6 TE Policy来实现。由于SRv6 TE Policy的路径控制机制更加灵活可靠，目前通过SRv6 TE Policy实现的SRv6 TE已经成为业界主流，因此本文中的SRv6 TE等同SRv6 TE Policy。

 **SRv6 BE和SRv6 TE的对比**

* 路径选择：SRv6 BE由IGP自动选择最佳路径，而SRv6 TE可以控制数据包的行进路径。

* 服务质量：SRv6 BE不提供质量保证，适用于对网络性能要求较低的场景；而SRv6 TE可以根据路径的时延和抖动等信息来优选路径从而保证服务质量，适用于对网络性能要求较高的场景。

* 网络资源利用：SRv6 BE可能导致网络资源利用不均衡；而SRv6 TE可以实现网络资源的优化利用。

* 部署复杂性：SRv6 BE部署更加简单方便；而SRv6 TE可能需要更复杂的网络配置。

SRv6 BE和SRv6 TE分别适用于不同的网络场景和需求，根据实际应用场景选择合适的网络管理策略，可以更好地满足业务需求，实现网络资源的优化利用。

## 4.3 SRv6 BE

## 4.4 SRv6 TE

SRv6 TE Policy主要由以下几部分构成：

* Endpoint：目标节点的IPv6地址，表示报文的目的地。

* Color：SRv6 TE Policy的Color属性，用于在相同的源和目的节点之间区分多个SRv6 TE Policy，也可以区分不同的业务类型和优先级。

* Segment List：一个预先定义好的Segment Identifier（SID）列表，用于指示报文在网络中的行进路径。

* Candidate Path：SRv6 TE Policy的候选路径，SRv6 TE Policy中可以存在多条候选路径，根据候选路径的优先级流量选取最优的候选路径来转发流量，次优的候选路径可以作为最优候选路径的热备份。每条候选路径又可以存在多条Segment List列表，在候选路径转发流量时，根据每个Segment List列表的权重值可以实现基于权重的等价路径转发WECMP。)

<img src="http://123.57.190.49:12121/api/image/PPP08428.png" title="" alt="" data-align="center">

SRv6 TE Policy的工作机制也拆分为路由发布流程和数据转发流程。

以IPv4 L3VPN over SRv6 TE Policy为例，节点A和节点D分别作为SRv6 TE Policy的源节点和尾节点，同时作为PE设备，分别连接了属于VPN实例VPNA的CE设备。SRv6 TE Policy的Endpoint为节点D的Loopback口地址4::4，SRv6 TE Policy的Color属性为10，SRv6 TE Policy中仅存在一条候选路径且候选路径中只有一条SID List，即<2001:A::1,2001:B::1,2001:C::1>。路由发布的过程如下：

(1) 网络中的节点A、节点B、节点C和节点D通过IGP互相发布本地的SRv6 Locator D。

(2) 节点A和节点D建立了MP-BGP邻居，尾节点D通过BGP为VPN实例中的私网路由X分配SRv6 SID 2001:D::1，该SRv6 SID属于SRv6 Locator D范围。

(3) 节点A接收到携带SRv6 SID 2001:D::1的私网路由X后，根据VPN实例的RT值将X加入到本地VPNA的路由表中。节点A发现私网路由X的下一跳为节点D的Loopback口地址4::4等于SRv6 TE Policy的Endpoint，且私网路由X携带了Color扩展团体属性10等于SRv6 TE Policy的Color属性，因此，去往私网路由X的流量将被引入到SRv6 TE Policy中转发。

以IPv4 L3VPN over SRv6 TE Policy为例，报文转发流程如下：

(1) 节点A向VPNA的私网路由X发送业务报文，此时，节点A发现去往私网路由X被引入SRv6 TE Policy中转发。

(2) 节点A为原始报文封装IPv6报文头和SRH扩展报文头，SRH扩展报文头的SID列表中包含私网路由X的End.DT4 SID 2001:D::1，同时还包含SRv6 TE Policy的SID List，因此在SRv6报文中SID列表排列为{SID 2001:D::1,2001:C::1,2001:B::1,2001:A::1}，Segment Left=3。节点A将SID[3]=2001:A::1作为IPv6报文的目的地址，2001:A::1作为节点A的本地End.X SID绑定的出接口为A和B之间的链路接口，因此，无需查IPv6 FIB，直接将报文从出接口转发给节点B，同时将Segment Left-1=2，并且将IPv6报文的目的地址替换为SID[2]=2001:B::1。

(3) 节点B接收到SRv6报文，发现IPv6报文的目的地址2001:B::1为本地End.X SID，其绑定的出接口为B和C之间的链路接口，因此，无需查IPv6 FIB，直接将报文从出接口转发给节点C，同时将Segment Left-1=1，并且将IPv6报文的目的地址替换为SID[1]=2001:C::1。节点C重复节点B的处理流程，将报文转发给节点D。

(4) 节点D收到报文后，发现报文目的地址2001:D::1为本地的End.DT4 SID，按照该SID对应的Endpoint Behavior执行转发动作，即则解封装报文，去掉IPv6报文头，并根据End.DT4 SID匹配VPN实例VPNA，在VPN实例VPNA的路由表中，查表转发，将原始报文发送给CE。

<img src="http://123.57.190.49:12121/api/image/06024V46.png" title="" alt="" data-align="center">
