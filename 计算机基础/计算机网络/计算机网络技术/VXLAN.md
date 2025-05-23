# 1. VXLAN简介

## 1.1 定义

RFC7348定义了VLAN扩展方案VXLAN（Virtual eXtensible Local Area Network，虚拟扩展局域网）。VXLAN采用MAC in UDP（User Datagram Protocol）封装方式，是NVO3（Network  Virtualization over Layer 3）中的一种网络虚拟化技术。

## 1.2 挑战

随着网络技术的发展，云计算凭借其在系统利用率高、人力/管理成本低、灵活性/可扩展性强等方面表现出的优势，已经成为目前企业IT建设的新趋势。而服务器虚拟化作为云计算的核心技术之一，得到了越来越多的应用。

服务器虚拟化技术的广泛部署，极大地增加了数据中心的计算密度；同时，为了实现业务的灵活变更，虚拟机VM（Virtual Machine）需要能够在网络中不受限迁移，这给传统的“二层+三层”数据中心网络带来了新的挑战。

* 虚拟机规模受网络设备表项规格的限制
  在传统二层网络环境下，数据报文是通过查询MAC地址表进行二层转发。服务器虚拟化后，VM的数量比原有的物理机发生了数量级的增长，伴随而来的便是VM网卡MAC地址数量的空前增加。而接入侧二层设备的MAC地址表规格较小，无法满足快速增长的VM数量。

* 网络隔离能力有限
  VLAN作为当前主流的网络隔离技术，在标准定义中只有12比特，因此可用的VLAN数量仅4096个。对于公有云或其它大型虚拟化云计算服务这种动辄上万甚至更多租户的场景而言，VLAN的隔离能力无法满足。

* 虚拟机迁移范围受限
  由于服务器资源等问题（如CPU过高，内存不够等），虚拟机迁移已经成为了一个常态性业务。
  虚拟机迁移是指将虚拟机从一个物理机迁移到另一个物理机。为了保证虚拟机迁移过程中业务不中断，则需要保证虚拟机的IP地址、MAC地址等参数保持不变，这就要求虚拟机迁移必须发生在一个二层网络中。而传统的二层网络，将虚拟机迁移限制在了一个较小的局部范围内。

* 针对虚拟机规模受设备表项规格限制
  VXLAN将管理员规划的同一区域内的VM发出的原始报文封装成新的UDP报文，并使用物理网络的IP和MAC地址作为外层头，这样报文对网络中的其他设备只表现为封装后的参数。因此，极大降低了大二层网络对MAC地址规格的需求。

* 针对网络隔离能力限制
  VXLAN引入了类似VLAN ID的用户标识，称为VXLAN网络标识VNI（VXLAN Network Identifier），由24比特组成，支持多达16M的VXLAN段，有效得解决了云计算中海量租户隔离的问题。

* 针对虚拟机迁移范围受限
  VXLAN将VM发出的原始报文进行封装后通过VXLAN隧道进行传输，隧道两端的VM不需感知传输网络的物理架构。这样，对于具有同一网段IP地址的VM而言，即使其物理位置不在同一个二层网络中，但从逻辑上看，相当于处于同一个二层域。即VXLAN技术在三层网络之上，构建出了一个虚拟的大二层网络，只要虚拟机路由可达，就可以将其规划到同一个大二层网络中。这就解决了虚拟机迁移范围受限问题。

# 2. VXLAN的基本原理

<img src="http://123.57.190.49:12121/api/image/02LHZP6F.png" title="" alt="" data-align="center">

* VXLAN/NVGRE/STT是三种典型的NVO3技术。

* 是通过**MAC In IP技术**在IP网络之上构建逻辑二层网络。

* 同一租户的VM彼此可以二层通信、跨三层物理网络进行迁移。

* 相比传统L2 VPN等Overlay技术，NVO3的CE侧是虚拟或物理主机，而不是网络站点。

* 此外主机具有可移动性。

* 目前，一般是IT厂商主导，通过服务器的Hypervisor来构建Overlay网络。




