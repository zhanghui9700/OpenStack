计算节点
================================


计算节点时OpenStack云计算架构中的资源核心，它为虚拟机实例提供了处理器、内存、网络以及存储资源。

**处理器的选择**


计算节点的处理器选择是非常重要的，首先，确保CPU支持VT-x(Intel处理器)或AMD-v(AMD处理器)等虚拟化技术，当然，CPU的核心数量也是重要的因素，目前处理器的核心通常高达12个，如果处理器还支持超线程的话，那么处理器相当于拥有24核。在同一台服务器上，若使用了多颗处理器，则再乘以相应的数目。

是否开启超线程，取决于您在什么需求下使用它们，我们建议您针对运行的事务在开启和关闭超线程的环境下，做性能上的对比测试，以便取得对处理器最优化的利用。


**虚拟机管理程序的选择**


OpenStack计算服务不同层次的支持多种虚拟机管理程序，包括KVM、LXC、QEMU、UMl、VMWare、ESX/ESXi、Xen、PowerVM以及Hyper-V等。

很多情况下，虚拟机管理程序的选择很大层面取决于您当前的使用和经验，除此之外，还有诸如特殊优势、文档支持或您在某个社区的经验等因素。例如，KVM是OpenStack社区中最有群众基础的，除了KVM，部署Xen、LXC、VMWare和Hyper-V的也比较多，但是，这些虚拟机管理程序的特性支持和在OpenStack上部署的指导性文档都太陈旧，缺乏更新。

如果您在选择虚拟机管理程序上存在疑虑，请参考OpenStack Wiki上这篇“虚拟机管理器支持矩阵”(https://wiki.openstack.org/wiki/HypervisorSupportMatrix),或者虚拟机管理程序参考手册(http://docs.openstack.org/folsom/openstack-compute/admin/content/ch_hypervisors.html)。

贴士：
可以在一个主机聚合单元或机柜中部署多种虚拟机管理程序，但一个独立计算节点只能一次运行一个虚拟机管理程序。


**虚拟机实例的存储方案**


在规划计算集群时，如何存储计算节点上的虚拟机也是必须要考虑到的。下面的三种存储方式如何选用，首先需要理解这三种方式都有什么特点：

	计算节点外存储 – 共享文件系统  
	计算节点内存储 – 共享文件系统
	计算节点内存储 – 非共享文件系统

通常，选择存储时都会面临如下问题：

	能管理多少磁盘？
	在网络访问的前提下，多磁盘是否能作到较好的I/O吞吐？
	在最佳性价比的情况下，哪一个符合您的目标？
	实际业务中，是如何管理这些存储的？


**计算节点外存储 – 共享存储**


很多云计算环境使用相对独立的计算与存储节点。计算服务与存储服务从硬件上来说需求是有差别的，计算节点通常比存储节点需要更强劲的处理器和更大的内存。因此，在预算内，我们需要更合理配置计算与存储节点，比如我们可以通过给计算节点配置较好的处理器和较多的内存，给存储节点配置较多的块存储来获得性价比。

另外，如果采用了相对独立的存储和计算主机，您的计算主机可处于“无状态“，相当于它与其他服务完全没有关联。这简化了计算节点的维护。只要计算节点上没有虚拟机实例运行，您可以将其离线甚至挂起，而这样的操作不会对您整个云系统造成任何影响。当然，如果您受物理主机数量的影响，又想运行尽可能多的虚拟机实例时，不妨考虑在购买主机时兼顾存储和计算能力，这样，就能将计算与存储放在同一台物理主机内。

存储与计算节点相对独立的方式确实有以下几个方面的优点：

	如果计算节点出错，虚拟机可以很方便的修复；
	操作和管理专用的存储系统会相对简单；
	可方便的扩展存储；
	可将额外的存储用于其他用处；

这种方法的两个主要缺点是：

	某一些实例的超量I/O会影响其他的实例对存储的影响，这是该架构的设计缺陷；
	网络传输的方式降低了效率；


**计算节点内存储 – 共享文件系统**


在这种模式下，每一个nova计算节点都可以有磁盘,由一个分布式的文件系统将各个计算节点的磁盘统一管理起来。这种方式的主要好处是：

	如果实例需要额外的磁盘份额，这种方式可通过整体的分布式文件系统进行磁盘分配；

这个方式也有它的缺点：

	与非共享存储相比，使用分布式文件系统可能会有数据丢失的风险；
	由于多台主机共同使用该文件系统，虚拟机的修复会相对复杂；
	机箱的固定大小从某种程度上来说限制了存储设备的容量；
	网络传输的方式降低了效率；


**计算节点内存储  - 非共享文件系统**


这种情况下，每一个计算节点都设定了足够的磁盘来存储它所管理的虚拟主机实例，有两个优势说明它其实是一个好方法：

	一台计算节点的大I/O不会影响到其他计算节点的虚拟机实例；
	直接的磁盘读取可获得较高的效率；

它的缺点是：

	如果一个计算节点故障，这个节点的虚拟机实例将出现问题；
	机箱的固定大小限制了一个节点可使用的存储容量；
	节点间实例的迁移将会比较复杂，有依赖的功能可能将无法继续展现；
	如果需要额外的空间，将无法扩展；


**动态迁移的问题**


我们将动态迁移当作是云计算系统操作特点的一部分，该功能可以将虚拟机实例从一台物理机无缝迁移到另一台物理主机上，但综合前文所说，该功能仅在共享存储的架构上可行。理论上，动态迁移也可以在非共享存储的架构上实现，可以采用一个叫KVM活块迁移的功能，但是，这个功能与动态迁移相比，知道的人很少，做过的测试也有限。KVM上游也计划弃用它。


**文件系统的选择**


如果您希望支持动态迁移,那么需要配置成如下的分布式文件系统，形成共享存储。它们是：

	NFS (linux默认采用)
	GlusterFS
	MooseFS
	Lustre

上述的文件系统我们都部署过，您可根据您熟悉的进行选择。


**资源配置**


OpenStack允许您给计算节点配置富裕的处理器和内存，这可以让您在云上尽可能多的运行虚拟机实例，成本会较低，虚拟机效率会较高。OpenStack官方对计算节点

有如下建议：

	CPU 配置比例: 16
	RAM 配置比例: 1.5

处理器的配置比例为16，意思是一个节点，调度程序在每一个物理核上将分配到16个虚拟核，例如，一个物理节点，它有12个处理器核心，每一个虚拟机实例使用4个

虚拟核，调度程序会分配最多192个 (16x12)虚拟核给虚拟机实例。(每个虚拟主机实例分配4个虚拟核，则可运行48个虚拟机实例)

类似的，内存的分配比例是1.5，意思是调度程序最多分配给虚拟机实例1.5倍的物理节点内存大小，例如，一个物理节点有内存48GB，调度程序会最多分配给虚拟主

机总共72GB内存(如6个虚拟机实例，每台虚拟机分配到8GB内存的情况下)

综上所述，您需要根据您的需求来选择合适的CPU和内存的分配比例。


**日志**


日志本身实际比**日志与监控**这一章节的内容要多得多，在您的云架构中要非常重视日志系统的设计，建议您可以考虑设置一个专门的中央日志服务器，记录和分析

OpenStack运营时产生的大量有价值的日志信息。(比如logstash)


**网络**


网络在OpenStack环境中是一个复杂、多面的挑战，我们会单独放在**网络设计**章节进行讨论。
