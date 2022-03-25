---
title: 6000O Cloud Computing Note
date: 2022-03-13 13:51:28
tags: 云计算
categories: 笔记
---

# 6000O Cloud Computing Note

## 1. Cloud Pricing and Economics

## 三种计费模式

- Reservation-based

  Cost(t) = Upfront + discount * R(On-demand rate) * t

  作用：Guaranteed availability，无论数据中心多么繁忙都能保证资源可用

  适合stable workload

- Spot pricing

  用户提交一个竞标（bid），云定期查看出价，出价最高的用户竞标成功

  通常比on-demand便宜很多，因为没有No service guarantee。用户的bid不会超过on-demand的价格。

  偶尔会有spot price很高的情况，是因为服务器把空间资源用spot pricing卖掉，用出价这种方式来回收。

- On-demand

为什么要有不同计费模式：

Market segmentation：每种模式适合的用户不同

供应方的问题：

- 资源有限

- 每种类型分配多少资源

- 每种类型如何定价

用户方的问题：

- 需要预测需求
- 需要预测spot price的价格，并且想办法活用

Cloud brokerage service：专门的公司来解决以上客户问题

## Total Cost of  Ownership (TCO)

决定要自己部署本地机房还是用云服务。

Total cost of ownership (TCO)：系统直接和间接成本的估算

## 提供者如何盈利

提供者使用资源池和multi-tenant model来服务客户。用不同的物理和虚拟资源来动态分配。

- Location Independence：用户不知道自己的服务是在哪里被实现的。

- 资源池可以实现很高的资源利用率。

- 规模经济：机房越大，单价越省
- Statistical multiplexing：灵活满足不同的用户在不同时间的资源需求

## 2. Service Models and Challenges

#### Cloud deployment models

- Public
- Private：安全、没有网络带宽和可用性问题
- Hybrid

#### Cloud Service Models

架构：硬件、虚拟化、基础设施、平台、应用

- IaaS

  提供者将硬件资源虚拟化，用户得到一台虚拟机，可以完全控制操作系统、存储和网络。典型案例：Netflix

- PaaS

  提供软件平台，或者中层，来供应用运行。用户要维护和部署他们的应用在平台上。

  硬件资源自动分配，用户无法指定。

  可以自动扩展，但是用户无法控制操作系统、存储和网络。

- SaaS

  提供软件和应用，并且来维护。用户只管使用。

从上到下，灵活性和定制性增强，但是便利性和易管理性减弱。

XaaS：

- FaaS（function）

  用户通过云函数的形式编写应用，规定触发函数执行的事件。然后云平台接管一切，包括资源分配、自动伸缩、容错等。用户只用根据**CPU时间**来付费。用户不管理服务器，因此也叫**serverless computing**

  好处：不用管理服务器，全由云提供者来维护；省钱，不用为空闲时间付费；灵活伸缩，由云提供者配置；高可用和高容错

- MLaaS

#### Issues of Cloud

- availability：停电等故障会导致损失
- 数据丢失
- Vendor lock-in：某个云中的应用无法迁移到另一个
- 安全性
- 隐私性：云服务商会偷窥数据吗

#### Challenges facing cloud  providers

- storage：大数据集无法存在本地，持久存储必须分布式；本地存储是脆弱的。
- Scale：
- Faults and failures：机器难免出问题
- Networking：带宽、公平性、如何保证网络又快又稳定
- Machine heterogeneity：每个机器的硬件都不同，如何保证性能；难以提供可预测和稳定的服务；难以监控系统，识别性能瓶颈，和掉队者的原因；难以达成用户之间的**公平共享**。

目标：

- 可以运行在各种规模上
- 容错
- 可预测的服务
- 高利用率
- 高上行和下行带宽
- 尽量少的人工干预

## 3. Virtualization

虚拟化是一个宽泛的术语。它可以应用于所有类型的资源（CPU、内存、网络等）

通过跨多个环境共享一台计算机的资源，允许一台计算机“看起来”像多台计算机，执行多个作业。

- Host：底层的物理机器

- Virtual Machine Manger (VMM) or hypervisor：通过提供与主机相同的接口创建和运行虚拟机（半虚拟化情况除外）

- virtual machine (VM)：基于软件实现的计算机，可用于运行通用的操作系统

- Guest：通常是操作系统

### Implementation of VMMs

- Type-0 hypervisors

  **基于硬件**的解决方案，通过固件（firmware）支持虚拟机的创建和管理。常见于大型机和大中型服务器中。

- Type-1 hypervisors

  类似操作系统的软件，直接在纯粹的基于x86的系统上提供虚拟化层。比如VMWare ESX。本身就类似于OS，其上可以有各种Guest OS。

  还包括提供标准功能和VMM功能的通用操作系统。通常没有专用的1型虚拟机监控程序功能丰富

- Type-2 hypervisors

  VMM本身是一个进程（程序），由主机运行和管理。比如Virtual Box，运行在Host OS之上。

  非常易用。由于是通过Host OS来间接访问硬件，性能不好，需要host OS授权，一般是个人使用。

- Other variations

  - Para-virtualization：将Guest OS修改为与VMM协作以优化性能的技术。Guest OS知道自己是虚拟机。
  - Programming-environment virtualization：VMM不会虚拟化真正的硬件，而是创建一个优化的虚拟系统，例如JVM
  - Emulators：允许为一个硬件环境（如iOS，Android）编写的应用程序在不同的硬件环境中运行

Type-1 hypervisors和Para-virtualization是云计算中最流行的。

### 历史

早期的大型机有很多问题：只能是batch-oriented；架构不兼容，升级硬件必须修改软件

MIT研发MAC (Multiple Access Computer)，是一种分时系统

- 隔离：主机系统受VM保护；虚拟机相互保护
- 冻结、暂停、运行虚拟机：VM可以暂停和拷贝
- 有利于研究OS和系统开发
- 实时迁移

计算机有一系列硬件：CPU，内存，IO设备，其他

OS：提供应用软件访问硬件资源的特殊软件层。与任何其他程序一样运行，但以特权（内核）CPU模式运行，将自己保护与用户程序中。可以访问硬件，执行敏感指令。

- 为应用程序提供高级编程接口（系统调用接口），OS实现接口来调用底层设备。
- 与硬件互动，代表硬件控制程序的指令

Application：

- 依赖于系统调用接口的程序，在用户态运行。通过特殊指令来调用OS
- MMU硬件允许操作系统给程序提供**虚拟地址空间**，让程序认为自己内存充足 

### How does virtualization work?

如果将OS（比如Windows）当做用户态的程序运行，大部分时间是可行的，但是：

- 如果需要进入内核态执行敏感指令？
- Guest OS该用那个硬件？
- 如何保护Guest OS上的程序不会伤害GuestOS

#### Trap-and-emulate

Guest OS必须运行在用户态。Guest VM需要virtual user mode 和  virtual kernel mode，让Guest以为自己在正常运行，但实际上这两个模式都运行在**用户态**。只有VMM可以运行在**内核态**。

GuestOS中导致切换到内核模式的操作必须导致VM切换到虚拟内核模式。

如何从虚拟用户模式切换到虚拟内核模式？

1. Trap：Guest在用户模式下尝试特权指令会导致错误——主机陷阱到内核模式
2. Emulate：VMM获得控制权，分析错误，模拟Guest尝试的指令的效果。VMM为Guest提供了一个虚拟硬件/软件接口
3. Return：VMM在用户模式下将控制权返回给Guest

#### Correctness requirement

指令有两种：

- Privileged instructions：当CPU处于用户态时，被trap（需要内核态）
- Sensitive instructions：修改（虚拟）硬件配置或资源的指令，以及行为依赖于（虚拟）硬件配置的指令，比如读写，设置寄存器

Emulation is only needed for sensitive instructions

Popek & Goldberg要求：如果敏感指令集是特权指令的子集，则可以高效且安全地构造VMM。这是对CPU架构的要求，但好多CPU不支持，有些敏感指令会运行在用户态。

#### How about the performance?

如果Popek & Goldberg要求达到，那么性能会如何？

- Non-sensitive instructions：几乎没有开销，虚拟机和真机几乎一样快。

- Sensitive instructions：如果引发trap，必须被引导到VMM并被其模拟。每一条指令可能需要数十条本机指令来模拟。

  I/O或系统调用密集型应用程序受到重创。

#### Trap-and-emulate并不总是管用（重点）

Trap-and-emulate依赖CPU error来捕获特权指令，如果所有的敏感指令都是特权指令，那么VMM可以提前预知到所有的敏感指令。

Intel架构不符合要求。popf指令，从堆栈内容加载CPU标志寄存器。

- 如果CPU在内核态：所有flag都被替换
- 如果CPU在用户态：只有部分flag被替换，并且不会陷入内核态。

Prof是敏感但不特权的指令，不能用Trap-and-emulate来虚拟化。



有些CPU没有明确区分特权指令和非特权指令，这使得一些特殊指令无法虚拟化。英特尔CPU直到1998年才被认为是可虚拟化的。

3种解决方法：

- Full virtualization：Emulate + binary translation

- Para-virtualization：修改GuestOS以避免不可虚拟化的指令
- Hardware-assisted virtualization：修复CPU

#### Full virtualization

**x86 protection rings**

Intel x86架构有4个保护级别，从高到底为ring0-3。ring0对应内核态，ring3对应用户态，ring1和ring2在现代OS中不使用。ring0可以执行所有指令。

另Guest OS kernel运行在ring1，每当Guest要运行敏感指令时，Hypervisor会发现，并用binary translation将CPU从ring1陷入ring0，模拟运行结果然后返回。

注意：Full virtualization并不依赖与x86架构

binary translation的思路很简单，但是实现很复杂。

- 如果Guest VCPU处于用户模式，guest可以以本机方式运行指令

- 如果Guest VCPU处于（虚拟）内核模式，hypervisor会检查Guest将要执行的每一条指令
  - 非特殊指令自然运行
  - 将特殊指令转换为新的指令集，在仿真硬件中执行等效任务

hypervisor向VM的Guest OS提供一整套模拟硬件。无论主机系统上的实际物理硬件是什么，模拟硬件都保持不变。

binary translation的步骤：

1. trapping I/O calls：每当GuestPS请求硬件时，例如请求BIOS提供硬件列表，它都会被hypervisor捕获
2. emulate/translate：无法虚拟化的指令被翻译为安全指令

GuestOS被欺骗，以为它在ring0中运行特权代码。它实际上运行在主机的ring1中，hypervisor模拟硬件并捕获特权代码。

优点：

- 不需要修改GuestOS
- 防止不稳定的虚拟机影响系统性能
- VM移植性

缺点：

- 没有优化，性能就不好。可能的解决方案是：缓存特殊指令的翻译，以避免将来再次翻译

#### Para-virtualization

旨在通过硬件仿真克服完全虚拟化带来的性能损失。要求修改GuestOS的内核。最著名的实现是**Xen**。云计算中**事实上**的虚拟化技术。

指出Full virtualization是错误方向：对Full virtualization的支持从来都不是x86体系结构设计的一部分。

- 为了实现正确的虚拟化，VMM必须处理某些监控器指令，但在权限不足的情况下执行这些指令会以静默方式失败，而不会造成方便的trap。
- 高效虚拟化x86 MMU也很困难（需要维护一个影子页表）
- 这些问题是可以解决的，但代价是复杂性增加和性能降低

在某些情况下，GuestOS希望看到真实的和虚拟的资源：

- 提供实时和虚拟时间可以让GuestOS更好地支持对时间敏感的任务，并正确处理TCP超时和RTT估计
- 公开真实的机器地址允许GuestOS通过使用超级页面或页面着色来提高性能

Paravirtualization：权衡对GuestOS的小改动，与性能和VMM简单性方面的大提升。

Modified GuestOS kernel也运行在ring1，通过hypercall来调用Xen Hypervisor。

##### VM Memory Interface

虚拟化内存很难，但如果架构有以下部分则会变简单：

- 一个由软件管理的TLB（转译后备缓冲区），可以有效地虚拟化
- 标记的TLB（带有地址空间标识符的TLB），不需要在每次转换时刷新

不幸的是，这两种功能在x86中都不受支持。VMware的解决方案（full virtualization）使用shadow page  tables，这可能非常慢！

Xen的解决方案（直接页表访问）

- GuestOS允许对**真实页表**进行**只读**访问。**对页表（PT）的更新必须由hypervisor验证**，这确保来GuestOS只能映射到分配给它的物理内存。

- GuestOS使用hypercall分配和管理自己的PTs
- **Xen存在于每个OS地址空间顶部的64MB区域中**，因此在进入和离开虚拟机监控程序时避免了TLB刷新

##### VM CPU Interface

trap/expection （系统调用、页面错误）处理程序向Xen注册以进行验证。

GuestOS可能会为系统调用安装一个“快速”异常处理程序，允许应用程序直接调用其GuestOS，并避免在每次调用时通过Xen间接调用

##### Control Transfer

在Xen中没有硬件中断，都是由软件事件来实现的。Xen用事件来告知GuestOS发生了什么。

Hypercalls：从GuestOS到Xen的同步调用（类似于系统调用）

#### Para vs. Full Virtualization

Full Virtualization

- 不要更改操作系统，除非在运行时（二进制翻译）
- 性能缓慢（有时不正确）

Para Virtualization：

- 对操作系统的更改最少
- 更好的性能和操作系统与虚拟硬件之间更快的交互

### Cloud infrastructures

云计算通常与虚拟化有关

- “高度弹性”
- 在虚拟化环境中启动新虚拟机既便宜又快速
- 将多个虚拟机整合到一台物理机器上可以提高利用率

云基础设施实际上是一个虚拟机管理基础设施

IaaS:

1. 客户端选择一个映像文件来启动VM
2. 控制器选择服务器来承载VM
3. 数据传输
4. 启动VM

公有云总是需要虚拟技术，对于私有云而言不一定，谷歌的集群都是建立在裸机之上的：高效而无性能损失。Container

## 4. Container Virtualization

Configure once, run anything。一次配置，到处运行。

VM vs. Containers：容器是隔离的，但是共享OS、bins/库

虚拟机系统调用路径：

- 虚拟机内部的应用程序进行系统调用
- 陷入Hypervisor（或主机操作系统）
- 把trap还给GuestOS

容器虚拟化系统调用路径

- 容器内的应用程序进行系统调用
- 陷入OS
- 操作系统将结果返回给应用程序

从高层看，容器就是轻量的VM

- 有自己的进程空间
- 自己的网络接口
- 可以以root运行
- 可以有自己的 /sbin/init

从底层看：容器就是一个隔离的进程

容器的实现利用Linux内核机制

- namespace：每个进程的资源独立
- cgroups：管理流程组的资源
- seccomp：限制可用的系统调用
- capabilities：限制可用权限
- CRIU：checkpoint/restore (w/ kernel support)

什么name需要虚拟化：

- 进程ID：`top`指令在容器中只显示容器中的进程。在容器外部，`top`指令可能会显示容器内的进程ID，但是可能不同。
- 文件名：容器内的进程可能对装载的文件系统有有限的不同视图。文件名可能会解析为不同的名称，容器外的一些文件名可能会被删除。
- 用户名：容器可能有不同的用户和不同的角色。容器内的`root`不一定是外部的`root`
- 主机名和IP地址：容器内的进程在执行网络操作时可能会使用不同的主机名和IP地址。

在进程粒度上限制内核端**名称**和数据结构的范围。

三个系统调用来管理：clone(), unshare(), setns(int fd, int nstype)

资源管理：操作系统可能希望确保整个容器，或其中运行的所有东西，消耗的能量不能超过一定量的CPU时间、内存、硬盘或者网络带宽。

**cgroups：Linux控制组**

控制组子系统为一组流程提供资源管理解决方案。
每个子系统都有一个层次结构（树），CPU、内存、块I/O有独立的层次结构。每个进程都位于每个层次结构中的一个节点中，每个节点都是一组共享资源的进程。

### Container

Container：一种基于cgroup和名称空间等内核机制的轻量级资源虚拟化。
多个容器在同一个内核上运行，并误以为它们是唯一使用资源的容器。

## 5. Distributed Storage System

就算能把单个硬盘做得足够大，也无法解决IO瓶颈，伸缩性很差。
CPU运算速度远高于硬盘读取速度。
建造一台高端超级计算机的成本非常非常高。
将所有数据存储在一个位置会增加硬件故障的风险。

Google的解决方式：向外扩展，而不是向上扩展。增加更多机器，而不是把机器做大。

许多廉价的、商品化的PC，每台都有磁盘和CPU。
高总存储容量，而不是增加单个硬盘的容量。
在多台机器上分散搜索处理：高I/O带宽，与机器的数量成比例；并行化数据处理。

注意：机器越多，每个时刻可能有机器出问题的概率越大。

在大范围内，超级花哨的可靠硬件仍然会失败，尽管失败的频率较低。

可靠性必须来自**软件**！

GFS：建立在高度不可靠硬件之上的高度可靠的存储系统

### Target environment（系统假设）

大量机器、分布式、出错时正常的
文件很大，但是数量不多。\>100M，一般是几个G大，几百万个文件。
一次写入，多次读取。文件通过追加来修改。大的流式读取和小的随机读取是典型的。
I/O带宽比延迟更重要。适用于批处理和日志分析。
如果文件系统为并发附件提供同步，这会很有帮助。

### Design Decisions

文件存储被分为数据块，大小为64MB。小于单个块的文件**不会**占用整个块的存储空间。

为什么使用大块：将磁盘搜索的成本降至最低；将文件传输速率提高到磁盘传输速率（传输时间远大于检索时间，2者相加等于文件传输时间 ）

**通过复制实现可靠性**
3-way replication：每个区块在3个以上的区块服务器上复制，同一个rack有一个副本，其他rack上有2+副本。

单master协调访问：维护元数据（文件名、权限、区块索引、文件夹层次结构等）。
文件数据存储在其他服务器上。访问数据需要向master申请。

添加**记录追加**操作，支持实时追加。

### General Architecture

Single master, Multiple chunkservers

1. 用file name和chunk index，向GFS Master查询
2. Master返回chunk的信息（chunk handle和locations）
3. 客户向chunk server发送chunk句柄和byte range
4. chunkserver返回data

可能的问题：

1. Single point of failure：如果master掉线怎么办、

   增加**shadow master**

2. Scalability bottleneck：如果master遇到瓶颈怎么办

   尽量减少主机参与，以解决可扩展性问题

   - 不参与数据传输，只用做元数据
   - 大chunk：最小化搜索/索引时间
   - **chunk leases**：master将权限委托给数据中的主副本

#### Metadata

三种元数据都保存在内存中

- 文件和chunk的命名空间
- 从文件到chunk的映射（每个chunk都有一个唯一的ID）
- 每个chunk副本的位置

前2种类型通过**操作日志**持久化。

第3种操作通过在启动时轮询chunkservers获得的区块副本位置。Chunkserver是其持有的区块的最终仲裁者 

#### Master的职责

- 元数据存储
- 命名空间管理/锁定
- 与服务器的定期通信：给出指示、收集状态、跟踪群集运行状况
- 区块创建、重新复制、重新平衡
  - 在racks上分散副本，减少错误
  - 如果冗余低于阈值，则重新复制数据
- 垃圾回收：比传统的文件删除更简单、更可靠
  - master记录删除，将文件重命名为隐藏名称，惰性回收隐藏文件
- 过时副本删除：使用ckunk版本号检测“过时”副本

#### Chunkserver

- 在本地磁盘上存储64 MB的文件块，每个块都有**版本号**和校验和。
- 读/写请求指定块**句柄**和**字节范围**。
- 在可配置数量的Chunkserver上复制的区块（默认：3-way复制）
- 无文件数据缓存

#### Client

- 向master发出控制（元数据）请求，比如`ls`命令
- 直接向服务器发出数据请求，比如`cat`命令
- 缓存的不是文件数据，而是元数据，这样可以减少与master的交互

### File read and write

文件读取：

1. 应用程序发起读取请求
2. GFS客户端翻译请求并将其发送给master 
3. Master使用块句柄和副本位置进行响应
4. 客户端选择“**最近**”的位置并发送请求（chunk句柄，比特范围）
5. Chunkserver将请求的数据发送到客户端
6. 客户端将数据转发给应用程序

文件写入：

1. 申请发起请求
2. GFS客户端翻译请求并将其发送给master 
3. Master使用块句柄和副本位置进行响应
4. 客户端将写入数据推送到所有位置。数据存储在区块服务器的内部缓冲区中。buffer满了就写入到disk。
   - 客户可能同时向3个服务器发送数据，也可能使用管道，向最近的服务器发送数据，该服务器再把数据复制到其他服务器。
5. 客户端向primary发送写命令
6. Primary确定其缓冲区中数据突变的顺序，并按顺序写入数据块。
7. Primary将串行命令发送给Secondary，并告诉它们执行写操作
8. Secondary回应primary
9. primary回应客户

### Fault Tolerance

如果是写入出错，就重写。通过检查checksum跳过不一致的文件区域。

chunkserver定时要向master报告状态。Master检测到chunkserver出现故障的“心跳”

如果chunkserver出错：

- Master会减少dead chunkserver上所有区块的副本数。
- Master将丢失副本的块重新复制到其他地方

特点：

- 高可用性
  - 快速恢复：master和纯看server可以在几秒内重启
  - 区块副本：默认情况下为3个副本
  - Shadow master
- 数据完整性：每个chunk中每64KB块就有一个checksum

### 性能

读取可以达到网络上限的75-80%，写入可以达到50%，客户越多带宽越大。

GFS演示了如何在商品硬件上支持大规模处理工作负载

- 容忍频繁的组件故障（故障是常态，而不是例外）
- 优化主要附加然后按顺序读取的大型文件
- 为许多并发读写器提供高聚合吞吐量

限制：

- 假设一次写入，多次读取
- 假设大文件数量适中：large chunk会导致，如果许多客户端访问同一个文件，小文件会在chunkserver上形成热点。



## 6. Hadoop Distributed File System

### HDFS overview

HDFS是GFS的**开源**实现，是一种**文件系统**，用于存储具有**流式数据访问模式**的**非常大的文件**，运行在**商用硬件**集群上

设计假设：

- 廉价的商品机器：在大型集群中，错误是常态而不是意外
- “适度”数量的非常大的文件：几百万个大于100MB的文件
- 批处理：
  - 一次写入，大部分追加（可能同时）
  - **流式读取**，而不是随机数据访问
  - **高持续吞吐量**优于低延迟

HDFS的设计和GFS很像：

- NameNode：用于管理文件系统元数据的单个主机

- DataNode (chunkserver)

  - 多个DataNode用于存储和检索数据

  - 向NameNode报告托管的块列表

- SecondaryNameNode (shadow master)：执行checkpointing

- 整个集群的单个命名空间

- 数据一致性：写入一次，读取多次

- 文件被分成块：每块**128MB**，每个块在多个数据节点上复制

- 智能客户端：客户端可以找到块的位置；客户端直接从数据节点访问数据

不同点：

- 早期版本的每个文件只有一个编写器
- 更早的版本不支持记录附加操作
- 开源，为不同的文件系统提供了许多接口和库

### Architecture 

#### NameNode

- 管理文件系统名称空间
  - 将文件名映射到一组块
  - 将块映射到其所在的数据
- 节点群集配置
- 管理复制引擎用于块

metadata：存储在内存中，记录了文件列表、每个文件的区块列表、文件属性、每个block在哪些DataNode。

事务日志：记录文件创建、文件删除等（metadata的前三项）

#### DataNode

- 在本地文件系统中保存数据
- 保存区块的metadata
- 向用户提供数据和元数据
- 定期向NameNode报告“心跳”
- 报告Block report：定期（默认情况下为1小时）向NameNode发送所有现有块的报告
- 促进数据的管道化：将数据转发到其他指定的DataNode

### Workflow

和GFS类似。但是如何选择最近的chunkserver呢？

不同的racks由Rack switch连接，Rack switch由Aggregation switch连接，形成一个树形结构。

- dist(/d1/r1/n1, /d1/r1/n1) = 0 (在同一个节点)
- dist(/d1/r1/n1, /d1/r1/n2) = 2 (同一个rack的不同节点)
- dist(/d1/r1/n1, /d1/r2/n3) = 4 (同一个数据中心的不同rack)
- dist(/d1/r1/n1, /d2/r3/n4) = 6 (不同的数据中心)

区块放置：当前策略（可替换为定制策略）

- 本地节点上有一个副本
- 同一远程机架的两个节点上有第二个和第三个副本
- 随机放置其他副本

一旦选择了复制副本位置，就会构建一个**管道**。

### Fault tolerance

错误处理：每三秒，DataNodes将心跳发送到NameNode。NameNode使用心跳来检测DataNode故障，10分钟内没有响应被视为失败。

**复制引擎**在检测到数据节点故障时，选择一个新的DataNode放置副本，平衡磁盘用量，平衡负载。

数据纠错：

- 验证数据的校验和（CRC32）
- 文件创建：客户端计算每512字节的校验和，DataNode保存校验和。
- 文件访问：客户端从DataNodes检索数据和校验和，如果验证失败就寻求其他副本。

NameNode failure：单点故障

- 事务日志存储在多个目录中：本地文件系统目录、远程文件系统目录
- 加入secondary NameNode
  - 非备用/备份名称节点：仅用于检查点，具有FSImage的非实时副本
  - 复制NameNode的FSImage和事务日志，将它们合并到一个新的FSImage
  - 将新的FSImage上载到NameNode并清除事务日志

### 限制（不满足）

- 低延迟数据访问：数十毫秒范围。HDFS更强调吞吐量而不是延迟
- 很多小文件：文件数量很多时，所有元数据都在内存中，发生溢出。
- 多个写入者，随机导致文件修改。

### Programming APIs   

`org.apache.hadoop.fs`

## 7. Map Reduce

并行化问题源于

- “工作人员之间的通信（如状态交换）”
- 对共享资源（如数据）的访问

因此，我们需要**同步机制**。

- semaphores（上锁，解锁）
- 条件变量（wait, notify, broadcast）
- barriers（一项工作在完成前置条件之前无法开始）

编程模型：shared memory (pthreads)、message passing (MPI)

设计模式：master-slaves、producer-consumer flows、shared work queues

解决典型的大数据问题，涉及5步：

- 对大量记录进行迭代
- 从每个迭代上提取感兴趣的东西
- 对中间结果进行洗牌和排序
- 汇总中间结果
- 生成最终输出

其中前2步是Map，后3步是Reduce。

关键思想：为这两个操作提供一个功能抽象

函数式编程：函数式操作从不修改现有的数据集，但会创建新的数据集。

[<img src="https://s1.ax1x.com/2022/03/13/bbnnvd.png" alt="bbnnvd.png" style="zoom:50%;" />](https://imgtu.com/i/bbnnvd)

如果中间有一个worker出问题了，那么它的计算就要重新进行一次，然后重新fold。

map函数可以任意，但是fold函数如果是有顺序要求的，那么不同节点的先后完成顺序会影响到结果。

因此要求fold函数有交换性（Commutativity）和结合性（Associativity）

**MapReduce的编程模型借鉴了函数式编程**

来自数据源的记录作为**键值对**提供

- map (k, v) → [<k2, v2>] 
- reduce (k2, [v2]) → [<k3, v3>]

具有**相同键**的**所有值**都被发送到同一个reducer。执行框架来处理**其他事情**

[<img src="https://s1.ax1x.com/2022/03/13/bbu2Y8.png" alt="bbu2Y8.png" style="zoom:50%;" />](https://imgtu.com/i/bbu2Y8)

什么是其他事情？

- 调度：分配worker、负载均衡
- 处理数据分布：将处理器移向数据、自动并行
- 处理同步：收集、排序、洗牌中间数据；网络和硬盘传输优化
- 处理错误：发现worker失败和重启

除了Map和Reduce函数，程序员还可以（可选）指定combiner和partitioner

`combine (k, [v]) → <k, v>`
mini-reducer，在内存中运行，在map后立刻运行，只处理本次map得到的数据。

`partition (k, # of partitions) → partition for k`
为并行reduce操作划分密钥空间。通常是密钥的简单散列，例如`hash(k) mod n`

**Barrier**在map和reduce之间。
map都完成了，reduce才能开始。但我们可以更早地开始将中间数据传输到执行map的管道shuffling中。

Key按照排序顺序到达每个reducer。reducer之间没有强制排序。类似于归并排序，每个reducer内按照key顺序到达和处理，但是互相没有排序关系。 

## 8. Hadoop

#### Mapper

- `void setup(Mapper.Context context)`：任务开始前初始化 
- `void map(K key, V value, Mapper.Context context)`: 对每个键值对调用
- `void cleanup(Mapper.Context context)`: 结束时调用

#### Reducer/Combiner

- `void setup(Reducer.Context context)`
- `void reduce(K key, Iterable value, Reducer.Context  context)`：每个key调用一次
- `void cleanup(Reducer.Context context)`

#### Partitioner

- `int getPartition(K key, V value, int numPartitions)`

#### terminology

Jobs：

- 打包Hadoop程序提交到集群
- 需要指定输入和输出路径
- 需要指定输入和输出格式
- 需要指定映射器、还原器、组合器、分区器
- 需要指定中间/最终键/值类
- 需要指定还原器的数量（但不是映射器，为什么？）

Task：在数据片上执行Map或Reduce，也称为Task-In-Progress（TIP）

Task attempt：试图在机器上执行任务的特定实例。某个特定任务将至少尝试一次，如果崩溃，可能会尝试更多次。

#### Data types

Writable -> WritableComparable -> IntWritable/LongWritable/Text

- Writable：定义反序列化/序列化协议。Hadoop中的每种数据类型都是Writable。
  如同java中`Object`是最原始的基类，`Writable`是Hadoop中的原始类。
- WritableComparable ：定义排序顺序。所有键都必须是这种类型（但不是值）。
- IntWritable等：不同数据类型的具体类。
- SequenceFile：键值对序列的二进制编码。Java自带的序列化太重了，Hadoop自己用了轻量的。

技巧：

- 尽可能避免对象创建——重用可写对象，更改有效负载
- 执行框架重用reducer中的值对象
- 通过类静态传递参数

如果遇到复杂数据类型，需要想办法用Hadoop支持的类型来表示，或者自己实现Writable接口。

## Anatomy of Hadoop

基本概念

每个cluster：

- NameNode：HDFS的master节点
- JobTracker：作业提交的master节点

每个从属机器需要设定：

- DataNode：作为HDFS的数据块
- TaskTracker：包含多个任务曹

Hadoop=HDFS+MapReduce

Hadoop有自己的RPC协议。

所有的交流都由slave发起：避免循环等待的死锁；slave周期性报告状态

所有的类都要提供显式的序列化方法，所以Hadoop数据类型必须继承自`Writable`。

Master节点运行JobTracker实例，接收client的作业申请。

Slave节点运行TaskTracker实例，分配java进程给任务。

Hadoop中的MapReduce程序 = Hadoop作业

作业被分为了map任务和reduce任务，多个任务可以整合为workflow。

作业提交：

- client创建作业，配置，提交到JobTracker。作业是jar文件和XML文件，包含序列化的程序配置选项。

运行MapReduce作业：

- 将jar和XML文件放入HDFS
- 通知TaskTracker从何处检索相关程序代码

计算Input splits
JobTracker将作业数据放在HDFS的共享位置，排队
TaskTrackers轮询任务。

#### Hadoop IO

输入格式：按行输入，键值对形式，序列化文件

输出格式：键值对，序列化文件

#### Shuffle 和sort

Map

- 映射输出缓冲在循环缓冲区中的内存中
- 当缓冲区达到阈值时，内容溢出到磁盘
- 溢出合并到单个分区文件中（在每个分区内排序）：合并器在合并期间运行

Reduce

- 映射输出被复制到reducer机器上
- “sort”是映射输出的多次合并（发生在内存和磁盘上）：合并器在合并过程中运行
- 最终的合并过程直接进入reducer

## 9. MapReduce Algorithm Design

框架负责：

- 调度任务，分配worker去做map和reduce任务
- 数据分发
- 同步：收集、排序、洗牌
- 发现和处理错误

程序员只用写mrcp四个函数，无法控制mapper和reducer

但是可以：

- 选择合适的数据类型
- 中间值key的排序顺序
- Partitioner：哪个reducer进程处理哪个key
- 保存mapper和reducer的状态

[<img src="https://s1.ax1x.com/2022/03/18/qkkGK1.png" alt="qkkGK1.png" style="zoom:50%;" />](https://imgtu.com/i/qkkGK1)

如何让Hadoop算法可伸缩

- 避免创建对象：昂贵的操作，垃圾回收影响效率
- 避免缓存：堆空间有限；用于小数据集

理想状态：运行时间和数据量成正比，和资源量成反比
为什么达不到：shuffle和sort需要同步机制，同步要求交流，交流影响性能

因此，要尽量避免交流

- 在本地reduce中间数据
- combiner帮忙（可选，不一定会用）

[<img src="https://s1.ax1x.com/2022/03/18/qkEQXR.png" alt="qkEQXR.png" style="zoom:50%;" />](https://imgtu.com/i/qkEQXR)

