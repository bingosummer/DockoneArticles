Docker网络和服务发现

【编者的话】 本文是《Docker网络和服务发现》一书的全文，作者是Michael Hausenblas。本文介绍了Docker世界中的网络和服务发现的工作原理，并提供了一系列解决方案。

#### 前言

当你开始使用Docker构建应用时，对于Docker的能力和它带来的机会，你会感到很兴奋。它可以同时在开发环境和生产环境中运行，只需要将一切打包进一个Docker镜像中，然后通过Docker Hub分发镜像。你会发现以下过程很令人满意：你可以快速地将现有的应用（如Python应用）移植到Docker上，将该容器与另一个数据库容器（PostgreSQL）连接；你不想手动启动Docker容器，也不想部署自己的系统来监控容器是否还在运行，并重启未运行的容器。

此时，你会意识到两个相互关联的挑战：网络和服务发现。说得好听一点，这两个领域都是新兴的课题，说得难听一点，在这两个领域里，还有很多不确定的内容，也缺乏最佳实践。幸运的是，还有大量的“秘方”分散在海量的博客和文章中。

##### 本书

因此，我对自己说：如果有人写本书，可以提供这些主题的一些基本指导，对于每项技术给读者指引正确的方向，那该多好啊。

那个“人”注定是我， 在本书里，我将介绍一下Docker容器的网络和服务发现领域中的挑战和现有的解决方案。 我想让大家了解以下三点：

* 服务发现和容器编排就像一枚硬币的两面。
* 对于Docker容器，若果没有正确的理解和健全的策略的话，那就糟糕了。
* 网络和服务发现领域还很年轻。你会发现，刚开始你还在学习一套技术，过一段时间，就会“换挡”，尝试另一套技术。不要紧张，这是很正常的。在我看来，在形成标准和市场稳定之前，还有两三年时间。

	编排和调度
	
	严格来讲，编排是比调度更广泛的一个概念：编排包含了调度，同时也包含其他内容，例如，在故障时重启容器（可能是因为容器本身不健康，也可能是宿主机遇到了麻烦）。同时，调度仅仅是指决定哪个容器运行在哪个宿主机上的过程。在本书中，我会无差别地使用这两个术语。
	
	我这么做的原因是：第一，在IETF RFC和NIST标准中没有官方定义；第二，在不同公司的市场宣传中，故意混淆了两者，因此我希望你能对此有所准备。然而，Joe Beda（前Google和Kubernetes的策划者）对于该问题发表了一篇相当不错的文章： [What Makes a Container Cluster?](http://dockone.io/article/173)，你可以更深入地了解一下。
	
##### 你

我希望，本书的读者是：

	* 开发者们，正在使用Docker
	* 网络ops，正在为热情的开发者所带来的冲击做准备
	* （企业）软件架构师，正在将现有负载迁移到Docker，或者使用Docker开始一项新项目
	* 最后但同样重要的是，我相信，分布式应用的开发者、SRE和后端工程师们也能从其中获取一些价值

需要注意的是，本书不是一本动手实践（hands-on）的书籍，除了第二章的Docker网络部分，更像是一本指导。当你实施基于Docker的部署时，你可以使用它来做出一个明智合理的决定。阅读本书的另一种方式是，添加大量注释和书签（a heavily annotated bookmark collection）。

##### 我

我在一个酷酷的创业公司Mesosphere, Inc.（Apache Mesos背后的商业公司）工作，主要的工作内容是帮助devops充分利用Mesos。虽然我有偏见地认为Mesos是目前进行大规模集群调度的最佳选择，我仍会尽我最大的努力确保这种偏爱不会影响本书介绍的各项技术。

##### 致谢

向Mesosphere同事们James DeFelice和Stefan Schimanski（来自Kubernetes组）致谢，他们很耐心地回答了我关于Kubernetes网络的问题。向我的Mesosphere同事们Sebastien Pahl和Tim Fall（原Docker同事）致谢，我非常感谢你们关于Docker网络的建议。也很感谢另一个Mesos同事Mohit Soni，在百忙之中他提供了很多反馈。

进一步地， 我要感谢Medallia的Thorvald Natvig，他的讲话[Velocity NYC 2015]（http://conferences.oreilly.com/velocity/devops-web-performance-ny-2015/public/schedule/detail/43771）促使我深入地思考了网络这一领域，他也很友善地和我讨论Medallia生产环境中使用Docker/Mesos/Aurora的经验和教训。

很感谢Adrian Mouat（容器解决方案）和Diogo Mónica（Docker, Inc.）通过Twitter回答问题，尤其是在普通人睡觉的时候，花了数小时回答了我的问题。

我也很感谢来自Chris Swan的反馈，他提供了明确可操作的评论。通过解决他的问题，我相信本书变得更有针对性了。

在本书的写作过程中，来自Google的Mandy Waite提供了很多有用的反馈，特别是Kubernetes，我很感谢这一点。我也很感激Tim Hockin（Google）的支持，他帮助我澄清了很多关于Docker网络特性和Kubernetes的疑惑。

感谢Matthias Bauer对于本书的草稿提出的宝贵意见。

非常感谢我的O’Reilly编辑Brian Anderson。从我们讨论草稿开始，你一直都非常支持我，很高效，也很有趣。非常荣幸和您一起工作。

最后重要的一点是，对于我的家庭要致以最深刻的谢意：我的“阳光”Saphira，我的“运动女孩”Ranya，我的儿子和“Minecraft大师”Iannis，以及一直支持我的妻子Anneliese。如果没有你们的支持，我是不可能完成本书的。

#### 动机

2012年2月，Randy Bias发表了一篇谈话，名为[开放和可扩展云架构]（），提出了pets和cattle两种文化基因的碰撞。

* 基础设施的pets方法（pets approach to infrastructure）： 你把机器当做一个个体，会给每一台机器或者虚拟机一个名字，把各个应用静态地部署在各个机器上。例如，db-prod-2是生产环境中的一台数据库服务器。应用是手动部署的，当一台机器出故障时，你会修复它，并把应用手动部署到另一台机器上。在非云原生时代（non-cloud-native era），这种方法通常占据着主导地位。

* 基础设施的cattle方法（cattle approach to infrastructure）： 你所有的机器都是无名的，它们是完全类似的（modulo hardware upgrades），它们只有编号，没有名字。应用是自动部署到机器群中的任意一台。当其中一台机器发生故障时，你不需要立刻担心它，只在需要的时候更换它（或者它的一部分，如一块损坏的HDD）。

尽管最初的文化基因是针对虚拟机的，我们这里要将cattle方法套用在容器基础架构上。

##### Go Cattle!

使用基础设施的cattle方法的一大好处就是，你可以在商用硬件上水平扩展。

这样一来，你就可以有很大的弹性来实施混合云。只要你愿意，一部分可以部署在premise环境中，一部分部署在公有云上，也可以部署在不同云提供商的IaaS架构上。

更重要的是，从一名运维者的角度上来看，cattle方法可以给你一个相当好的睡眠质量，因为你再也不会像pets方法中那样，凌晨3点被叫起来去更换一块坏了的HDD或者将应用重新部署到另一台服务器上。

然而，cattle方法也带来了一些挑战，主要包括以下两个方面：

* 社交挑战（Social challenges）

	我敢说，大部分挑战来自于社交方面：我应该怎么说服我的老板？我怎么让CTO买我的账？我的同事们会反对这种新的做法吗？这是不是意味着我们需要更少的人来维护基础架构？现在，对于这个问题，我还不能提供一个完整有效的解决方案。你可以买一本《凤凰项目》，从中你可能可以找到答案。

* 技术挑战（Technical challenges）

	这个方面主要包括：机器的准备机制（provisioning mechanism）如何选择，比如使用Ansible来部署Mesos Agents；在容器之间、在容器和外界之间如何通信；如何保证容器的自动部署并总能被找到。
	
##### Docker网络和服务发现栈

我们在图1-1中描述了整个技术栈，包括以下内容：

* 底层的网络层（The low-level networking layer）

	这一层包括一系列网络工具： iptables、路由、IPVLAN和Linux命名空间。你通常不需要知道其中的细节，除非你是在网络组，但是你需要意识到这一点。关于这一课题的更多信息，可以参考第二章。

* Docker网络层

	这一层封装了底层的网络层，提供各种抽象模型，例如单主机桥网络模式、多主机的每个容器一个IP地址的解决方案。在第2章和第3章中有相关介绍。

* 服务发现层/容器编排层

	这里，我们将容器调度定义为通过底层提供的基本原语（primitives）决定将容器放在哪里。第4章提到了服务发现的必要背景，第5章从容器编排的角度讲解了网络和服务发现。
	
	软件定义网络（SDN）
	
	SDN真的是一个市场术语（umbrella term或者marketing term），它可以提供VM带来的网络优势，就像裸金属服务器（bare-metal servers）一样。网络管理团队会变得更加敏捷，对商业需求的变化反应更加迅速。另一种看法是：SDN是一种使用软件来定义网络的配置方式，无论它是通过API还是NFV来构建。
	
	如果你是一名开发者或架构师，我建议你看一下Cisco对于该课题的[观点]()和SDxCentral的文章[What’s Software-Defined Networking (SDN)?]()。
	
图1-1

如果你在网络运维组的话，你可能已经准备好进入下一章了。然而，如果你是架构师或者开发者的话，你的网络知识可能有点生锈（rusty）了，我建议你读一下[Linux Network Administrators Guide]()来复习一下相关知识。

##### 我需要All-In吗？

在各种会议和用户组中，我经常遇到一些人，他们对于容器领域的机会非常兴奋，同时他们也担心在从容器中受益之前需要投入多少。以下的表格是对于我了解的各种部署的非正式的总结（按照阶段不同排序）。

图

需要注意的是，不是所有的例子都使用Docker容器（显而易见的是，Google并不使用Docker容器）。其中，某些正在着手使用Docker，例如，在ad-hoc阶段；某些正在转向full-down阶段，例如Uber，请看ContainerSched 2015 London中的[演讲]()。最后重要的一点是，这些阶段与部署的大小是没有必然关系的。例如，Gutefrage.de只有六台裸金属服务器，仍然使用Apache Mesos来管理它们。 

在继续之前，还有最后一点：到目前为止，你可能已经注意到了我们处理的是分布式系统。由于我们总是希望将容器部署到一个集群中，我强烈建议你读一下[分布式计算的谬论]()，以防你不熟悉这一主题。

现在言归正传，让我们进入Docker网络这一主题。

#### Docker网络基础

在我们进入网络细节之前，我们先来看一看在单主机上发生了什么。Docker容器需要运行在一台宿主机上，可以是一台物理机（on-premise数据中心中的裸金属服务器），也可以是on-prem或云上的一台虚拟机。就像图2-1中描述的那样，宿主机上运行了Docker的daemon进程和客户端，一方面可以与Docker registry交互，另一方面可以启动、关闭和审查容器。

图2-1

宿主机和容器的关系是1：N，这以为这一台宿主机上可以运行多个容器。例如，从Facebook的报告来看，取决于机器的能力，每台宿主机上平均可以运行10到40个容器。另一个数据是：在Mesosphere，我们发现，在裸金属服务器上的各种负载测试中，每台宿主机上不超过250个容器是可能的。

无论你是在单主机上进行部署，还是在集群上部署，你总得和网络打交道：

* 对于大多数单主机部署来说，问题归结于是使用共享卷进行数据交换，还是使用网络（基于HTTP或者其他的）进行数据交换。尽管Docker数据卷很容易使用，它也引入了紧耦合，这意味着很难将单主机部署转化为多主机部署。自然地，共享卷的优势是速度。

* 在多主机部署中，你需要考虑两个方面：单主机上的容器之间如何通信和多主机之间的通信路径是怎样的。性能考量和安全方面都有可能影响你的设计决定。多主机部署通常是很有必要的，原因是单主机的能力有限（请看前面关于宿主机上容器的平均数量和最大数量的讨论），也可能是因为需要部署分布式系统，例如Apache Spark、HDFS和Cassandra。

	分布式系统和数据本地化（Distributed Systems and Data Locality）
	
	使用分布式系统（计算或存储）的基本想法是想从并行处理中获利，通常伴随着数据本地化。数据本地化，我指的是将代码转移到数据所在地的原则，而不是传统的、其他的方式。考虑下以下的场景：如果你的数据集是TB级的，而代码是MB级的，那么在集群中移动代码比传输TB级数据更高效。除了可以并行处理数据之外，分布式系统还可以提供容错性，因为系统中的一部分可以相对独立地工作。
	
本章主要讨论单主机中的网络，在第3章中，我们将介绍多主机场景。

简单地说，Docker网络是原生的容器SDN解决方案。总而言之，Docker网络有四种模式：桥模式，主机模式，容器模式和无网络模式。我们会详细地讨论单主机上的各种网络模式，在本章的最后，我们还会讨论一些常规主题，比如安全。

##### bridge模式网络

在该模式（见图2-2）中，Docker守护进程创建了一个虚拟以太网桥`docker0`，附加在其上的任何网卡之间都能自动转发数据包。默认情况下，守护进程会创建一对对等接口，将其中一个接口设置为容器的eth0接口，另一个接口放置在宿主机的命名空间中，从而将宿主机上的所有容器都连接到这个内部网络上。同时，守护进程还会从网桥的私有地址空间中分配一个IP地址和子网给该容器。

	因为bridge模式是Docker的默认设置，所以你也可以使用`docker run -d -P nginx:1.9.1`。如果你没有使用-P（发布该容器暴露的所有端口）或者-p host_port:container_port（发布某个特定端口），IP数据包就不能从宿主机之外路由到容器中。
	
	在生产环境中，我建议使用Docker的host模式（将会在下一小节Host模式网路中讨论），并辅以第3章中的某个SDN解决方案。进一步地，为了控制容器间的网络通信，你可以使用flags参数：--iptables和-icc。
	
##### host模式网络

该模式将禁用Docker容器的网络隔离。因为容器共享了宿主机的网络命名空间，直接暴露在公共网络中。因此，你需要通过端口映射（port mapping）来进行协调。

我们可以从例2-2中看到：容器和宿主机具有相同的IP地址10.0.7.197。

在图2-3中，我们可以看到：当使用host模式网络时，容器实际上继承了宿主机的IP地址。该模式比bridge模式更快（因为没有路由开销），但是它将容器直接暴露在公共网络中，是有安全隐患的。

##### container模式网络

该模式会重用另一个容器的网络命名空间。通常来说，当你想要自定义网络栈时，该模式是很有用的。实际上，该模式也是Kubernetes使用的网络模式，你可以在这里了解更多内容。

结果（例2-3）显示：第二个容器使用了--net=container参数，因此和第一个容器admiring_engelbart具有相同的IP地址172.17.0.3。

##### none模式网络

该模式将容器放置在它自己的网络栈中，但是并不进行任何配置。实际上，该模式关闭了容器的网络功能，在以下两种情况下是有用的：容器并不需要网络（例如只需要写磁盘卷的批处理任务）；你希望自定义网络，在第3章中有很多选项使用了该模式。

在例2-4中可以看到，恰如我们所料，网络没有任何配置。

你可以在Docker官方文档中读到更多关于Docker网络的配置。

	本书中的所有Docker命令都是在CoreOS环境中执行的，Docker客户端和服务端的版本都是1.7.1。
	
##### 更多话题

在本章中，我们了解了Docker单主机网络的四种基本模式。现在我们讨论下你应该了解的其他主题（这与多主机部署也是相关的）：

* 分配IP地址

	频繁大量的创建和销毁容器时，手动分配IP地址是不能接受的。bridge模式可以在一定程度上解决这个问题。为了防止本地网络上的ARP冲突，Docker Daemon会根据分配的IP地址生成一个随机地MAC地址。在下一章中，我们会再次讨论分配地址的挑战。

* 分配端口

	你会发现有两大阵营：固定端口分配（fixed-port-allocation）和动态端口分配（dynamically-port-allocation）。每个服务或者应用可以有各自的分配方法，也可以是作为全局的策略，但是你必须做出自己的判断和决定。请记住，bridge模式中，Docker会自动分配UDP或TCP端口，并使其可路由。

* 网络安全

	Docker可以开启容器间通信（意味着默认配置--icc=true），也就是说，宿主机上的所有容器可以不受任何限制地相互通信，这可能导致拒绝服务攻击。进一步地，Docker可以通过--ip_forward和--iptables两个选项控制容器间、容器和外部世界的通信。你应该了解这些选项的默认值，并让网络组根据公司策略设置Docker进程。可以读一下StackEngine的Boyd Hemphill写的文章Docker security analysis。
	
	另一个网络安全方面是线上加密（on-the-wire encryption），通常是指RFC 5246中定义的TLS/SSL。注意，在写本书时，这一方面还很少被讨论，实际上，只有两个系统（下一章会详细讨论）提供了这个特性：Weave使用NACI，OpenVPN是基于TLS的。根据我从Docker的安全负责人Diogo Mónica那里了解的情况，v1.9之后可能加入线上加密功能。
	
最后，可以读一下Adrian Mouat的Using Docker，其中详细地介绍了网络安全方面。

	自动Docker安全检查
	
	为了对部署在生产环境中的Docker容器进行安全检查，我强烈建议使用The Docker Bench for Security。
	
现在，我们对单主机场景有了一个基本的了解，让我们继续看一下更有效的案例：多主机网络环境。

#### Docker多主机网络

只要是在单主机上使用Docker的话，那么上一章中介绍的技术就足够了。然而，如果一台宿主机的能力不足以应付工作负载，要么买一个更高配置的宿主机（垂直扩展），要么你添加更多同类型的机器（水平扩展）。

对于后者，你会搭建一个集群。那么，就出现了一系列问题：不同宿主机上的容器之间如何相互通信？如何控制容器间、容器和外部世界之间的通信？怎么保存状态，如IP地址分配、集群内保持一致性等？如何与现有的网络基础架构结合？安全策略呢？

为了解决这些问题，在本章中，我们会讨论多主机网络的各种技术。

	对于本章中介绍的各种选项，请记住Docker信奉的是“batteries included but replaceable”，也就是说，总会有一个默认功能（如网络、服务发现），但是你可以使用其他方案来替代。
	
##### Overlay

2015年3月，Docker, Inc.收购了软件定义网络（SDN）的创业公司SocketPlane，并更名为Docker Overlay驱动，作为多主机网络的默认配置（在Docker 1.9以后）。Overlay驱动通过点对点（peer-to-peer）通信扩展了通常的bridge模式，并使用一个可插拔的key-value数据库后端（如Consul、etcd和ZooKeeper）来分发集群状态。

##### Flannel

CoreOS Flannel是一个虚拟网络，分配给每个宿主机一个子网。集群中的每个容器（或者说是Kubernetes中的pod）都有一个唯一的、可路由的IP地址，并且支持以下一系列后端：VXLAN、AWS VPC和默认的2层UDP overlay网络。flannel的优势是它降低了端口映射的复杂性。例如，Red Hat的Atomic项目使用的就是flannel。

##### Weave

Weaveworks Weave创建了一个虚拟网络，用来连接部署在多主机上的Docker容器。应用使用网络的方式就像是容器直接插入到同一个网络交换机上，不需要配置端口映射和连接。Weave网络上的应用容器提供的服务可以直接在公共网络上访问，无论这些容器在哪里运行。同样的，无论位置在哪，现有的内部系统都是暴露给应用容器的。Weave可以穿越防火墙，在部分连接的网络中运行。流量可以加密，允许主机跨越非授信网络进行连接。你可以从Alvaro Saurin的文章Automating Weave Deployment on Docker Hosts with Weave Discovery中学习到Weave的更多特性。

##### Project Calico

Metaswitch的Calico项目使用标准IP路由，严格的说是RFC 1105中定义的边界网关协议（Border Gateway Protocol，简称BGP），并能使用网络工具提供3层解决方案。相反，大多数其他的网络解决方案（包括Weave）是通过封装2层流量到更高层来构建一个overlay网络。主操作模式不需要任何封装，是为可以控制物理网络结构的组织的数据中心设计的。

##### Open vSwitch

Open vSwitch是一个多层虚拟交换机，通过可编程扩展来实现网络自动化，支持标准管理接口和协议，如NetFlow、IPFIX、LACP和802.1ag。除此之外，它还支持在多个物理服务器上分发，和VMware的vNetwork distributed vSwitch、Cisco的Nexus 1000V类似。

##### Pipework

Pipework由著名的Docker工程师Jérôme Petazzoni创建，称为Linux容器的软件定义网络。它允许你使用cgroups和namespace在任意复杂的场景中连接容器，并与LXC容器或者Docker兼容。由于Docker, Inc.收购了SocketPlane并引入了Overlay Driver，我们必须看一下这些技术如何融合。

##### OpenVPN

OpenVPN，另一个有商业版本的开源项目，运行你创建使用TLS的虚拟私有网络（virtual private networks，简称VPN）。这些VPN也可以安全地跨越公共网络连接容器。如果你想尝试一下基于Docker的配置，我建议看一下DigitalOcean很赞的教程How To Run OpenVPN in a Docker Container on Ubuntu 14.04。

##### 未来的Docker网络

在最近发布的Docker v1.9中，引入了一个新的命令docker network。容器可以动态地连接到其他网络上，每个网络都可以由不同的网络驱动来支持。默认的多主机网络驱动是Overlay。

为了了解更多实践经验，我建议看一下以下博客：

* Aleksandr Tarasov的Splendors and Miseries of Docker Network
* Calico项目的Docker libnetwork Is Almost Here, and Calico Is Ready!
* Weave Works的Life and Docker Networking – One Year On

##### 更多话题

在本章中，我们讨论了多主机网络中的各种解决方案。这一小节会简要介绍一下其他需要了解的话题：

* IPVLAN

	Linux内核v3.19引入了每个容器一个IP地址的特性，它分配给主机上的每个容器一个唯一的、可路由的IP地址。实际上，IPVLAN使用一个网卡接口，创建了多个虚拟的网卡接口，并分配了不同的MAC地址。这个相对较新的特性是由Google的Mahesh Bandewar贡献的，与macvlan驱动类似，但是更加灵活，因为它可以同时使用在L2和L3上。如果你的Linux发行版已经使用了高于3.19的内核，那么你很幸运；否则，你就无法享受这个新功能。
	
* IP地址管理（IPAM）

	多主机网络中，最大的挑战之一就是集群中容器的IP地址分配。

* 编排工具兼容性

	本章中讨论的大多数多主机网络解决方案都是封装了Docker API，并配置网络。也就是说，在你选择其中一个之前，你需要检查与容器编排工具的可兼容性。更多主题，请看第5章。

* IPv4 vs. IPv6

	到目前为止，大多数Docker部署使用的是标准IPv4，但是IPv6正在迎头赶上。Docker从v1.5（2015年2月发布）开始支持IPv6。IPv4的持续减少将会导致越来越多的IPv6部署，也可以摆脱NATs。然而，什么时候转变完成还不清楚。

现在，你已经对底层网络和Docker网络的解决方案有了充分理解，让我们进入下一个内容：服务发现。

#### 容器和服务发现

管理基础架构的cattle方法的最大挑战就是服务发现。服务发现和容器调度是一枚硬币的两面。如果你使用cattle方法的话，那么你会把所有机器看做相同的，你不会手动分配某台机器给某个应用。相反，你会使用调度软件来管理容器的生命周期。

那么，问题就来了：如何决定容器被调度到哪台宿主机上？答对了，这就是服务发现。我们会在第5章详细讨论硬币的另一面：容器编排。

##### 挑战

服务发现已经出现了一段时间了，被认为是zeroconf的一部分。

zeroconf 

zeroconf的想法是自动化创建和管理计算机网络，自动分配网络地址，自动分发和解析主机名，自动管理网络服务。

对于Docker容器来说，这个挑战归结于稳定地维护运行中容器和位置（location）的映射关系。位置这里指的是IP地址（启动容器的宿主机地址）和可被访问的端口。这个映射必须及时完成，并在集群中准确地重启容器。容器的服务发现解决方案必须支持以下两个操作：

* 注册（Register）
建立`container->location`的映射。因为只有容器调度器才知道容器运行在哪，我们可以把该映射当做容器位置的“绝对真理”（the absolute source of truth）。

* 查询（Lookup）
其他服务或应用可以查询我们存储的映射关系，属性包括信息的实时性和查询延迟。

让我们看一下在选择过程中相对独立的几点考虑：

* 除了简单地向一个特定方向发送请求之外，怎么从搜索路径中排除不健康的宿主机和挂掉的容器？你已经猜到了，这是和负载均衡高度相关的主题，因为这是很重要的，所以本章的最后一小节将讨论这一主题。

* 有人认为这是实现细节，也有人认为需要考虑CAP三要素。在选择服务发现工具时，需要考虑选择强一致性（strong consistency）还是高可用性（high availability）。至少你要意识到这一点。

* 可扩展性也会影响你的选择。当然，如果你只有少量的节点需要管理，那么上面讨论的解决方案就够用了。然而，如果你的集群有100多个节点，甚至1000个节点，那么在选择某一项特定技术之前，你必须做一些负载测试。

	CAP理论
	
	1998年，Eric Brewer提出了分布式系统的CAP理论。CAP代表了一致性（consistency）、可用性（availability）和分区容错性（partition tolerance）：
	
	* 一致性
		分布式系统的所有节点同时看到相同的数据。
	* 可用性
		保证每个请求都能得到响应，无论该响应是成功还是失败。
	* 分区容错性
		无论哪块分区由于网络故障而失效，分布式系统都可以继续运行。
		
	CAP理论在分布式系统的实践中已经讨论多次了。你会听到人们主要讨论强一致性 vs 最终一致性，我建议读一下Martin Kleppmann的论文A Critique of the CAP Theorem。该论文提出了一种不同的、更实际的思考CAP的方法，特别是一致性。
	
如果你想了解该领域更多的需求和根本的挑战，可以读一下Jeff Lindsay的Understanding Modern Service Discovery with Docker和Shopify的Simon Eskildsen在DockerCon分享的内容。
 
##### 技术

该小节介绍了各种技术和它们的优缺点，并提供了网上的更多资源（如果你想获得这些技术的实践经验，你可以看看Adrian Mouat的书《Using Docker》）。

###### ZooKeeper

Apache ZooKeeper是ASF的顶级项目，基于JVM的集中式配置管理工具，提供了与Google的Chubby相兼容的功能。ZooKeeper(ZK)将数据载荷组织成文件系统，成为znodes的层级结构。在集群中，选举出一个leader节点，客户端能够连接到服务器中的任意一个来获取数据。一个ZK集群中需要2n+1个节点。最常见的配置是3、5、7个节点。

ZooKeeper是经战场证明的、成熟的、可扩展的解决方案，但是也有一下缺点。有些人认为ZK集群的安装和管理不是一个令人愉快的体验。我碰到的大多数ZK问题都是因为某个服务（我想到了Apache Storm）错误地使用了它。它们可能在znodes里放入了太多数据，更糟糕的是，他们的读写率很不健康（unhealthy read-write ratio），特别是写得太快。如果你打算使用ZK的话，至少考虑使用高层接口。例如，Apache Curator封装了ZK，提供了大量的方法；Netflix的Exhibitor可以管理和监控一个ZK集群。

从图4-1可以看出，你可以看到两个组件：R/W(作为注册监控器（registration watcher），你需要自己提供)和NGINX（被R/W控制）。当一个容器被调度到一个节点上时，它会在ZK中注册，使用一个路径为/$nodeID/$containerID的znode，IP地址作为数据载荷。R/W监控znodes节点的变化，然后相应地配置NGINX。这种方法对于HAProxy和其他负载均衡器也同样有效。

###### etcd

etcd是由CoreOS团队使用Go语言编写的。它是一个轻量级的、分布式键值对数据库，使用Raft算法实现一致性（带有leader选举的leader-follower模型），在集群中使用复制日志（replicated log）向followers分发lead收到的写请求。从某种意义上说，在概念上，etcd和ZK是相当类似的。虽然负载数据是任意的，但是etcd的HTTP API是基于JSON的。就像ZK一样，你可以观察到etcd保存的值的变化。etcd的一个非常有用的特性是键的TTL，是服务发现的重要的一个结构单元。和ZK一样，一个etcd集群至少需要2n+1个节点。

etcd的安全模型支持基于TLS/SSL和客户端证书加密的线上加密（on-the-wire encryption），加密可以发生在客户端和集群之间，也可以在etcd节点之间。

在图4-2中，你可以看到etcd服务发现的搭建和ZK是相当类似的。主要区别在于etcd使用confd来配置NGINX，而不是使用自己编写的脚本。和ZK一样，这种搭建方法也适用于HAProxy或者其他负载均衡器。

###### Consul

Consul是HashiCorp的产品，也是用Go语言写的，功能有服务注册、服务发现和健康检查，可以使用HTTP API或者DNS来查询服务。Consul支持多数据中心的部署。

Consul的其中一个特性是一个分布式键值对数据库，与etcd相似。它也是用Raft一致性算法（同样需要2n+1个节点），但是部署方式是不同的。Consul有agent的概念，有两种运行方式：服务器模式（提供键值对数据库和DNS）和客户端模式（注册服务和健康检查），使用serf实现了成员和节点发现。

使用Consul，你有四种方式来实现服务发现（从最可取到最不可取）：
* 使用服务定义配置文件（service definition config file），由Consul agent解释。
* 使用traefik等工具，其中有Consul后端（backend）。
* 编写你自己的进程通过HTTP API注册服务。
* 让服务自己使用HTTP API来注册服务。

想要学习Consul来做服务发现吗？请阅读这两篇文章：Consul Service Discovery with Docker和Docker DNS & Service Discovery with Consul and Registrator。 

###### 纯基于DNS的解决方案

在互联网中，DNS经受了数十年的战场验证，是很健壮的。DNS系统的最终一致性、特定的客户端强制性地（aggressively）缓存DNS查询结果、对于SRV记录的依赖这些因素使你明白这是正确的选择。

本小节之所以叫做“纯基于DNS的解决方案”的原因是，技术上讲Consul也是用了DNS服务器，但这只是Consul做服务发现的其中一个选项。以下是著名的、常用的、纯粹的基于DNS的服务发现解决方案：

* Mesos-DNS
	该解决方案是专门用于Apache Mesos的服务发现的。使用Go编写，Mesos-DNS下拉任意任务的有效的Mesos Master进程，并通过DNS或HTTP API暴露IP:PORT信息。对于其他主机名或服务的DNS请求，Mesos-DNS可以使用一个外部的域名服务器或者你的现有DNS服务器来转发Mesos任务的请求到Mesos-DNS。
* SkyDNS
	使用etcd，你可以将你的服务通告给SkyDNS，SkyDNS在etcd中保存了服务定义，并更新DNS记录。你的客户端应用发送DNS请求来发现服务。因此，在功能层面，SkyDNS与Consul是相当类似的，只是没有健康检查。
* WeaveDNS
	WeaveDNS由Weave 0.9引入，作为Weave网络的服务发现解决方案，允许容器按照主机名找到其他容器的IP地址。在Weave 1.1中，引入了所谓的Gossip DNS，通过缓存和超时功能使得查询更加快速。在最新的实现中，注册是广播到所有参与的实例上，在内存中保存所有条目，并在本地处理查询。

###### Airbnb的SmartStack和Netflix的Eureka

在本小节中，我们将会看一下两个定做的系统，它们是用来解决特定的需求。这并不意味着你不能或者不应该使用它们，你只是需要意识到它们。

Airbnb的SmartStack是一个自动的服务发现和注册框架，透明地处理创建、删除、故障和维护工作。在容器运行的同一个宿主机上，SmartStack使用了两个分离的服务：Nerve（写到ZK）用于服务注册，Synapse（动态配置HAProxy）用于查询。这是一个针对非容器环境的完善的解决方案，随着实践推移，你会看到对于Docker，SmartStack也是有用的。

Netflix的Eureka则不同，它运行在AWS环境中（Netflix全部运行在AWS上）。Eureka是一个基于REST的服务，用于定位服务以便负载均衡和中间件层服务器的故障迁移；Eureka还有一个基于Java的客户端组件，可以直接与服务交互。这个客户端有一个内置的负载均衡器，可以做基本的round-robin的负载均衡。在Netflix，Eureka用于做red/black部署、Cassandra和memcached部署、承载应用中关于服务的元数据。

Eureka集群在参与的节点之间异步地复制服务注册信息；与ZK、etcd、Consul不同，Eureka相对于强一致性更倾向于服务可用性，让客户端自行处理过时的读请求，优点是在网络分区上更有弹性。你也知道的：网络是可靠的，才怪。

##### 负载均衡

服务发现的一个方面是负载均衡，有的时候负载均衡被认为是正交的，有的时候负载均衡被认为是服务发现的一部分。它可以在很多容器之间分散负载（入站服务请求）。在容器和微服务的语境下，负载均衡同时具有以下功能：

* 最大化吞吐量，最小化响应时间
* 防止热点（hotspotting），例如单一容器过载
* 可以处理过度激进的DNS缓存（overly aggressive DNS caching）

以下列举了一些有名的Docker中的负载均衡解决方案：
* NGINX
* HAProxy
* Bamboo
* Kube-Proxy
* vulcand
* Magnetic.io的vamp-router
* moxy
* HAProxy-SRV
* Marathon的servicerouter.py
* traefik

如果你想了解更多关于负载均衡的信息，请查看Mesos meetup视频和nginx.conf 2014上关于NGINX和Consul的负载均衡的演讲。

##### 更多话题

在本章的最后，我列举了服务发现解决方案的一览表。我并不想评出一个优胜者，因为我相信这取决于你的用例和需求。因此，把下表当做一个快速定位和小结：

最后请注意：服务发现领域在不断变化中，每周都会出现新工具。例如，Uber最近开源了它的内部解决方案Hyperbahn，一个路由器的overlay网络，用来支持TChannel RPC协议。因为容器服务发现在不断发展，因此你可能要重新评估最初的选择，直到稳定下来为止。

#### 容器和编排

就像上一章中介绍的那样，使用cattle方法来管理基础架构，你不必手动为特定应用分配特定机器。相反，你应该使用调度器来管理容器的生命周期。尽管调度是一个重要的活动，但是它其实只是另一个更广阔的概念——编排的一部分。

从图5-1，可以看到编排包括健康检查（health checks）、组织原语（organizational primitives，如Kubernetes中的labels、Marathon中的groups）、自动扩展（autoscaling）、升级/回滚策略、服务发现。有时候base provisioning也被认为是编排的一部分，但是这已经超出了本书的讨论范围，例如安装Mesos Agent或Kubernetes Kubelet。

服务发现和调度是同一枚硬币的两面。决定一个特定的容器运行在集群的哪个节点的实体，称之为调度器。它向其他系统提供了容器到位置（containers->locations）的映射关系，可以以各种方式暴露这些信息，例如像etcd那样的分布式键值对数据库、像Mesos-DNS那样的DNS。

本章将从容器编排解决方案的角度讨论服务发现和网络。背后的动机很简单：假设你使用了某个平台，如Kubernetes；然后，你主要的关注点是平台如何处理服务发现和网络。

	接下来，我会讨论满足以下两个条件的容器编排系统：开源的和相当大的社区。
	你可以看一下以下几个不同的解决方案：Fackebook的Bistro或者托管的解决方案，如Amazon EC2的容器服务ECS。
	另外，你如果想多了解一下分布式系统调度这个主题，我推荐阅读Google关于Borg和Omega的研究论文。
	
在我们深入探讨容器编排系统之前，让我们先看一下编排的核心组件——调度器到底是做什么的。

##### 调度器到底是做什么的？

分布式系统调度器根据用户请求将应用分发到一个或多个可用的机器上。例如，用户可能请求运行应用的100个实例（或者Kubernetes中的replica）。

在Docker中，这意味着：（a）相应的Docker镜像存在宿主机上；（b）调度器告诉本地的Docker Daemon基于该镜像开启一个容器。

在图5-2中，你可以看到，对于集群中运行的应用，用户请求了三个实例。调度器根据它对于集群状态的了解，决定将应用部署在哪里。比如，机器的使用情况、成功启动应用所必须的资源、约束条件（该应用只能运行在使用SSD的机器）等。进一步地，服务质量也是考量因素之一。

通过John Wilkes的Cluster Management at Google了解更多内容。

	对于调度容器的限制条件的语义，你要有概念。例如，有一次我做了一个Marathon的demo，没有按照预期工作，原因是我没有考虑布局的限制条件：我使用了唯一的主机名和一个特定的角色这一对组合。它不能扩展，原因是集群中只有一个节点符合条件。同样的问题也可能发生在Kubernetes的label上。

##### Vanilla Docker and Docker Swarm

创造性地，Docker提供了一种基本的服务发现机制：Docker连接（links）。默认情况下，所有容器都可以相互通信，如果它们知道对方的IP地址。连接允许用户任何容器发现彼此的IP地址，并暴露端口给同一宿主机上的其他容器。Docker提供了一个方便的命令行选项--link，可以自动实现这一点。

但是，容器之间的硬连接（hard-wiring of links）并不有趣，也不具扩展性。事实上，这种方法并不算好。长久来说，这个特性会被弃用。

让我们看一下更好的解决方案（如果你仍然想要或者必须使用连接的话）：ambassador模式。

###### Ambassadors 

如图5-3所示，这个模式背后的想法是使用一个代理容器来代替真正的容器，并转发流量。它带来的好处是：ambassador模式允许你在开发阶段和生产阶段使用不同的网络架构。网络运维人员可以在运行时重写应用，而不需要更改应用代码。

简单来说，ambassador模式的缺点是它不能有效地扩展。ambassador模式可以用在小规模的、手动的部署中，但是当你使用真正的容器编排工具（如Kubernetes或Apache Mesos）时，应该避免使用ambassador模式。
	如果你想要了解如何在Docker中部署ambassador模式，我再次建议你参考Adrian Mouat的书Using Docker。事实上，在图5-3中，我使用的就是他的amouat/ambassador镜像。

###### Docker Swarm

除了容器的静态链接（static linking），Docker有一个原生的集群工具Docker Swarm。Docker Swarm基于Docker API构建，并通过以下方式工作：有一个Swarm manager负载调度，在每一个宿主机上运行了一个agent，负责本地资源管理（如图5-4所示）。

Swarm中有趣的地方在于，调度器是插件式的（plug-able）,所以你可以使用除了内置以外的其他调度器，如Apache Mesos。在写本书的过程中，Swarm发布了1.0版本，并完成了GA（General Availability）；新的特性（如高可用）正在进行开发中。

###### 网络

在本书的第2章和第3章中，我们介绍了Docker中的单宿主机和多宿主机中的网络。

###### 服务发现

Docker Swarm支持不同的后端：etcd、Consul和Zookeeper。你也可以使用静态文件来捕捉集群状态。最近，一个基于DNS的服务发现工具wagl被引入到了Swarm中。

如果你想更多了解Docker Swarm，可以读一下Rajdeep Dua的幻灯片。

##### Kubernetes

Kubernetes(请看图5-5)是一个opinionated的开源框架，弹性地管理容器化的应用。简单来说，它吸取了Google超过十年的运行容器负载的经验，我们会简要地介绍一下。进一步地，你总是可以选择其他开源或闭源的方法来替换Kubernetes的默认实现，比如DNS或监控。

以下讨论假设你已经熟悉Kubernetes和它的技术。如果你还不熟悉Kubernetes的话，我建议看一下Kelsey HighTower的[Kubernetes Up and Running]()。

Kubernetes中，调度单元是一个pod，这是a tightly coupled set of containers that is always collocated。pod运行实例的数目是由Replication Controller定义和指定的。pods和services的逻辑组织是由labels定义的。

在每个Kubernetes节点上，运行着一个称为Kubelet的agent，负责控制Docker daemon，向Master汇报节点状态，设置节点资源。Master节点提供API（例如，图5-6中的web UI），收集集群现在的状态，并存储在etcd中，将pods调度到节点上。

###### 网络

在Kubernetes中，每个pod都有一个可路由的IP，不需要NAT，集群节点上的pods之间也可以相互通信。pod中的所有容器共享一个端口命名空间（port namespace）和同一个notion localhost，因此没有必要做端口代理（port brokering）。这是Kubernetes的基本要求，可以通过network overlay来实现。

在pod中，存在一个所谓的infrastructure容器，kubelet实例化的第一个容器，它会获得pod的IP，并设置网络命名空间。pod中的所有其他容器会加入到infra容器的网络和IPC命名空间。infra容器的网络启用了bridge模式（请参考第九页的bridge模式网络），pod中的所有其他容器使用container模式（请参考第11页的container模式网络）来共享它的命名空间。infra容器中的初始进程实际上什么也没做，因为它的目的只是提供命名空间的载体。关于端口转发的最近的work around可能在infra容器中启动额外的进程。如果infra容器死亡的话，那么Kubelet会杀死pod中所有的进程，然后重新启动进程。

进一步地，Kubernetes的命名空间会启动所有control points。其中一个网络命名空间的例子是，Calico项目使用命名空间来强化coarse-grained网络策略（policy）。

###### 服务发现

在Kubernetes的世界里，有一个服务发现的canonical抽象，这是service primitive。尽管pods随时可能启动和销毁因为他们可能失败（或者pods运行的宿主机失败），服务是长时间运行的：它们提供集群层面的服务发现和某种级别的负载均衡。它们提供了一个稳定的IP地址和持久化名字，compensating for the shortlivedness of all equally labelled pods。Kubernetes提供了两种发现机制：通过环境变量（限制于一个特定节点上）和DNS（集群层面上的）。

##### Apache Mesos

Apache Mesos （图5-7）是一个通用的集群资源管理器，抽象了集群中的各项资源（CPU、RAM等），通过这种方式，集群对于开发者来说就像一个巨大的计算机。

在某种程度上，Mesos可以看作是分布式操作系统的内核。因此，它从未单独使用，总是和其他框架一起工作，例如Marathon（用于long-running任务，如web服务器）、Chronos（用于批量任务）或者大数据框架（如Apache Spark或Apache Cassandra）。

Mesos同时支持容器化负载（Docker容器）和普通的可执行文件（包括bash脚本、Python脚本、基于JVM的应用、一个纯粹的老的Linux二进制格式），也支持无状态和有状态的服务。

接下来，我假设你已经熟悉了Mesos和它的相关技术。如果你不熟悉Mesos，我建议你阅读David Greenberg的书《Building Applications on Mesos》，该书介绍了Mesos，特别是对于分布式应用的开发者。

如图5-8所示，你可以看到Marathon的UI，使用Apache Mesos来启动和管理长期运行的服务和应用。

###### 网络

网络特性和能力主要取决于Mesos使用的容器方案：

* 对于Mesos containerizer来说，有一些要求，如Linux内核版本要大于3.16，并安装libnl。开启网络隔离功能之后，你可以构建一个Mesos Agent。启动之后，你可以看到以下输出：

mesos-slave --containerizer=mesos --isolation=network/port_mapping --resources=ports:[31000-32000];ephemeral_ports: [33000-35000]

Mesos Agent会配置成使用非临时（non-ephemeral）端口31000-32000,临时（ephemeral）端口33000-35000。所有容器共享宿主机的IP地址，端口范围被分配到各个容器上（使用目的端口和容器ID之间的1:1映射）。通过网络隔离，你可以定义网络性能（如带宽），使得你能够监控容器的网络流量。更多细节，请看MesosCon 2015 Seattle上的演讲Per Container Network Monitoring and Isolation in Mesos。

* 对于Docker containerizer，请看第二章。

###### 服务发现

尽管Mesos不提供服务发现功能，但是有一个Mesos特定的解决方案，在praxis中也很常用：Mesos-DNS（Pure-Play DNS-based Solutions）。然而，还有其他的解决方案，如traefik。如果你对于Mesos中的服务发现很感兴趣，我们的文档中有一个专门的章节介绍它。

##### Hashicorp Nomad

Nomad是HashiCorp开发的集群调度器，HashiCorp也开发了Vagrant。Nomad于2015年9月份引入，主要目的是简化性。Nomad易于安装和使用。据说，它的调度器设计受到Google的Omega的影响，比如维护了集群的全局状态、使用优化的、高并发的调度器。

Nomad的架构是基于agent的，它有一个单独的二进制文件，可以承担不同的角色，支持滚动升级（rolling upgrade）和draining nodes（为了重新平衡）。Nomad使用了一致性协议（强一致性）来实现所有的状态复制和调度，使用gossip协议来管理服务器地址，从而实现集群自动化（automatic clustering）和多区域联合（multiregion federation）。从图5-9可以看到，Nomad agent正在启动。

Jobs定义为HCL格式（HashiCorp-proprietary format）或JSON格式。Nomad提供了命令行接口和HTTP API，来和服务器进程进行交互。

接下来，我假设你已经熟悉了Nomad和它的术语，否则我不建议你读这一节。如果你不熟悉的话，我建议你读一下Nomad: A Distributed, Optimistically Concurrent Schedule: Armon Dadgar, HashiCorp（这是HashiCorp的CTO Armon Dadgar对于Nomad的介绍）和Nomad的文档。

###### 网络

Nomad中有几个所谓的任务驱动（task drivers），从通用的Java可执行程序到qemu和Docker。在之后的讨论中，我们将会专注后者。

在写这篇文章的时候，Nomad要求Docker的版本为1.8.2，并使用端口绑定技术，将容器中运行的服务暴露在宿主机网卡接口的端口空间中。Nomad提供了自动的和手动的映射方案，绑定Docker容器的TCP和UDP协议端口。

关于网络选项的更多细节，例如端口映射（mapping ports）和标签（labels），我建议你读一下该文章。

###### 服务发现

在v0.2中，Nomad提供了基于Consul的服务发现机制，请参考相关文档。其中包含了健康检查（health checks），并假设运行在Nomad中的任务能够与Consul agent通信（使用bridge模式网络），这是一个挑战。

##### 我应该用哪个呢？

以下内容当然只是我的一个建议。这是基于我的经验，自然地，我也偏向于我正在使用的东西。可能有各种原因（包括政治上的原因）导致你选择一个特定的技术。

从纯粹的可扩展性的角度来看，这些选项有以下特点：

对于少量的节点来说，自然是无所谓的：根据你的偏好和经验，选择以上四个选项中的任意一个都可以。但是请记住，管理大规模的容器是很困难的：

  * Docker Swarm可以管理1000个节点，请参考HackerNews和Docker的这篇博客。
  * Kubernetes 1.0据说可以扩展至100个节点，并在持续改进中，以达到和Apache Mesos同样的可扩展性。
  * Apache Mesos可以管理最多50000个节点。
  * 到目前为止，还没有资料表明Nomad可以管理多少个节点。

从工作负载的角度来看，这些选项有以下特点：

非容器化（Non-containerized)意味着你可以运行任何Linux Shell中可以运行的程序，例如bash或Python脚本、Java程序等。容器化（containerized）以为你需要生成一个Docker镜像。考虑到有状态服务，当今应用中的很大一部分需要一些调整。如果你想更多地学习编排工具，请参考：

  * Docker Clustering Tools Compared: Kubernetes vs Docker Swarm
  * O'Reilly Radar的Swarm v. Fleet v. Kubernetes v. Mesos

为了完整性，我想介绍这个领域的一个新成员Firmament，这是一个非常不错的项目。这个项目的开发者们也是Google的Omega和Borg的贡献者，这个新的调度器将任务和机器组成了一个流网络，并运行最小成本优化（minimum-cost optimization）。特别有趣的一点是，Firmament不仅可以单独使用，也可以与Kubernetes和Mesos整合。

###### 容器的一天

当选择使用哪种容器编排方案时，你需要考虑容器的整个生命流程。

Docker容器典型的生命周期包括以下几个阶段：

阶段1： dev

	容器镜像（你的服务和应用）从一个开发环境中开始它的生命流程，通常是从一名开发者的笔记本开始的。你可以使用生产环境中运行的服务和应用的特性请求（feature requests）作为输入。

阶段2： CI/CD

	然后，容器需要经历持续集成和持续发布（continuous integration and continous delivery），包括单元测试、整合测试和冒烟测试。

阶段3： QA/staging

	然后，你可以使用一个QA环境（企业内部或云中的集群）和/或staging阶段。

阶段4: prod

	最后，容器镜像是部署到生产环境中的。你也需要有一个策略去分发这些镜像。不要忘记以canaries的方式进行构建，并为核心系统（如Apache Mesos）、潜在的高层组件（如Marathon）和你的服务和应用的滚动式升级（rolling upgrade）做计划。

	在生产环境中，你会发现bugs，并收集相关指标，可以用来改善下一个迭代（返回到阶段1）。

这里讨论的大部分系统（Swarm、Kubernetes、Mesos和Nomad）提供了很多指令、协议和整合点来覆盖这些阶段。然而，在你选择任何一个系统之前，你仍然需要端到端地试用该系统。

###### 社区是很重要的

#### 参考

以下这些链接，有的包含了背景知识，有的包含了一些高级内容。

##### 网络参考

* Docker Networking
* Concerning Containers’ Connections: on Docker Networking
* Unifying Docker Container and VM Networking
* Exploring LXC Networking
* Letting Go: Docker Networking and Knowing When Enough Is Enough
* Networking in Containers and Container Clusters

##### 服务发现参考

* Service Discovery on p24e.io
* Understanding Modern Service Discovery with Docker
* Service Discovery in Docker Environments
* Service Discovery, Mesosphere
* Docker Service Discovery Using Etcd and HAProxy
* Service discovery with Docker: Part 1 and Part 2
* Service Discovery with Docker: Docker Links and Beyond

##### 其他高级话题

* What Makes a Container Cluster?
* Fail at Scale—Reliability in the Face of Rapid Change
* Bistro: Scheduling Data-Parallel Jobs Against Live Production Systems
* Orchestrating Docker Containers with Slack
* The History of Containers
* The Comparison and Context of Unikernels and Containers
* Anatomy of a Container: Namespaces, cgroups & Some Filesystem Magic - LinuxCon

**原文链接：[Docker networking and service discovery](https://www.oreilly.com/learning/docker-networking-service-discovery)（翻译：夏彬）**
