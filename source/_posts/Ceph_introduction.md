---
title: 第一篇：Ceph简介
comments: true
author: zuoyang
type: 原创
toc: true
categories:
  - ceph
tags: ceph
abbrlink: 3a6a8a8c
date: 2018-11-19 20:15:07
---

# Ceph架构简介

最近工作中要使用ceph作为底层存储架构，故对其进行了一番调研，本篇乃ceph系列的第一篇。

## Ceph

Ceph是一个统一的分布式存储系统，设计初衷是提供较好的性能、可靠性和可扩展性。

Ceph项目最早起源于Sage就读博士期间的工作（最早的成果于2004年发表），并随后贡献给开源社区。在经过了数年的发展之后，目前已得到众多云计算厂商的支持并被广泛应用。RedHat及OpenStack都可与Ceph整合以支持虚拟机镜像的后端存储。

Ceph is a distributed object, block, and file storage platform.

使用Ceph系统可以提供**对象存储**、**块设备存储**和**文件系统服务**.

Ceph底层提供了分布式的RADOS存储，用与支撑上层的librados和RGW、RBD、CephFS等服务。Ceph实现了非常底层的object storage，是纯粹的SDS，并且支持通用的ZFS、BtrFS和Ext4文件系统，能轻易得Scale，没有单点故障。

## Ceph特点

- 高性能
  a. 摒弃了传统的集中式存储元数据寻址的方案，采用CRUSH算法，数据分布均衡，并行度高。
  b.考虑了容灾域的隔离，能够实现各类负载的副本放置规则，例如跨机房、机架感知等。
  c. 能够支持上千个存储节点的规模，支持TB到PB级的数据。
- 高可用性
  a. 副本数可以灵活控制。
  b. 支持故障域分隔，数据强一致性。
  c. 多种故障场景自动进行修复自愈。
  d. 没有单点故障，自动管理。
- 高可扩展性
  a. 去中心化。
  b. 扩展灵活。
  c. 随着节点增加而线性增长。
- 特性丰富
  a. 支持三种存储接口：块存储、文件存储、对象存储。
  b. 支持自定义接口，支持多种语言驱动。

## Ceph架构

**支持三种接口：**

- Object：有原生的API，而且也兼容Swift和S3的API。
- Block：支持精简配置、快照、克隆。
- File：Posix接口，支持快照。

{% asset_img rados.jpg  rados %}

{% asset_img ceph-architectural.png  ceph-architectural %}

## Ceph核心组件及概念介绍

{% asset_img ceph_all_component.png  ceph-all-component %}

- Monitor   

  一个Ceph集群需要多个Monitor组成的小集群，它们通过Paxos同步数据，用来保存OSD的元数据。

- OSD 

  OSD全称Object Storage Device，也就是负责响应客户端请求返回具体数据的进程。一个Ceph集群一般都有很多个OSD。

- MDS 

  MDS全称Ceph Metadata Server，是CephFS服务依赖的元数据服务。

- Object 

  Ceph最底层的存储单元是Object对象，每个Object包含元数据和原始数据。

- PG 

  PG全称Placement Grouops，是一个逻辑的概念，一个PG包含多个OSD。引入PG这一层其实是为了更好的分配数据和定位数据。

- RADOS 

  RADOS全称Reliable Autonomic Distributed Object Store，是Ceph集群的精华，用户实现数据分配、Failover等集群操作。

- Libradio 

  Librados是Rados提供库，因为RADOS是协议很难直接访问，因此上层的RBD、RGW和CephFS都是通过librados访问的，目前提供PHP、Ruby、Java、Python、C和C++支持。

- CRUSH 

  CRUSH是Ceph使用的数据分布算法，类似一致性哈希，让数据分配到预期的地方。

- RBD 

  RBD全称RADOS block device，是Ceph对外提供的块设备服务。

- RGW 

  RGW全称RADOS gateway，是Ceph对外提供的对象存储服务，接口与S3和Swift兼容。

- CephFS 

  CephFS全称Ceph File System，是Ceph对外提供的文件系统服务。



# CEPH Filesystem

## 文件存储

{% asset_img ceph_file.jpg  file-system %}

**典型设备：** FTP、NFS服务器
为了克服块存储文件无法共享的问题，所以有了文件存储。
在服务器上架设FTP与NFS服务，就是文件存储。

**优点：**

- 造价低，随便一台机器就可以了。
- 方便文件共享。

**缺点：**

- 读写速率低。
- 传输速率慢。

**使用场景：**

- 日志存储。
- 有目录结构的文件存储。
- …

## Ceph 文件系统

Ceph 文件系统（ Ceph FS ）是个 POSIX 兼容的文件系统，它使用 Ceph 存储集群来存储数据。 Ceph 文件系统与 Ceph 块设备、同时提供 S3 和 Swift API 的 Ceph 对象存储、或者原生库（ librados ）一样，都使用着相同的 Ceph 存储集群系统。 

![cephfs](http://docs.ceph.org.cn/_images/ditaa-b5a320fc160057a1a7da010b4215489fa66de242.png)

Ceph 文件系统要求 Ceph 存储集群内至少有一个 Ceph 元数据服务器。

当前， CephFS 还缺乏健壮得像 ‘fsck’ 这样的检查和修复功能。存储重要数据时需小心使用，因为灾难恢复工具还没开发完。

cephfs目前发展比较慢，之前一直没有稳定版本，2016年4月21日官方发布的jewel V10.2.0才公布第一个稳定版本，当前在生产环节中使用很少，所以还是建议谨慎使用，如果要使用需要进行严格的测试后才能上线。



# CEPH Block Device

## 块存储

{% asset_img disk.jpg  block device %}

**典型设备：** 磁盘阵列，硬盘

主要是将裸磁盘空间映射给主机使用的。

**优点：**

- 通过Raid与LVM等手段，对数据提供了保护。
- 多块廉价的硬盘组合起来，提高容量。
- 多块磁盘组合出来的逻辑盘，提升读写效率。

**缺点：**

- 采用SAN架构组网时，光纤交换机，造价成本高。
- 主机之间无法共享数据。

**使用场景：**

- docker容器、虚拟机磁盘存储分配。
- 日志存储。
- 文件存储。
- …



## Ceph 块设备 (RBD)

块是一个字节序列（例如，一个 512 字节的数据块）。基于块的存储接口是最常见的存储数据方法，它们基于旋转介质，像硬盘、 CD 、软盘、甚至传统的 9 磁道磁带。无处不在的块设备接口使虚拟块设备成为与 Ceph 这样的海量存储系统交互的理想之选。

Ceph 块设备是精简配置的、大小可调且将数据条带化存储到集群内的多个 OSD 。 Ceph 块设备利用 RADOS 的多种能力，如快照、复制和一致性。 Ceph 的 RADOS 块设备（ RBD ）使用内核模块或 librbd 库与 OSD 交互。

![rbd](http://docs.ceph.org.cn/_images/ditaa-dc9f80d771b55f2daa5cbbfdb2dd0d3e6dfc17c0.png)

内核模块可使用 Linux 页缓存。对基于 librbd 的应用程序， Ceph 可提供 RBD 缓存。

客户端可以通过内核模块挂在rbd使用，客户端使用rbd块设备就像使用普通硬盘一样，可以对其就行格式化然后使用；客户应用也可以通过librbd使用ceph块，典型的是云平台的块存储服务（如下图），云平台可以使用rbd作为云的存储后端提供镜像存储、volume块或者客户的系统引导盘等。

{% asset_img ceph_rbd2.png ceph-rbd %}

Ceph 块设备靠无限伸缩性提供了高性能，如向内核模块、或向 abbr:KVM (kernel virtual machines) （如 Qemu 、 OpenStack 和 CloudStack等云计算系统通过 libvirt 和 Qemu 可与 Ceph 块设备集成）。你可以用同一个集群同时运行 Ceph RADOS gateway、 Ceph FS 文件系统、和 Ceph 块设备。

目前ceph rbd在云平台使用比较广泛而且也很稳定，社区的支持力度也非常大。

# CEPH Object Gateway

## 对象存储

对象存储是提供restful接口并数据组织形式扁平化的存储方法，对象存储同兼具块存储高速直接访问磁盘及文件存储的分布式共享特点。

{% asset_img ceph-object.jpg %}

**典型设备：** 内置大容量硬盘的分布式服务器(swift, s3)
多台服务器内置大容量硬盘，安装上对象存储管理软件，对外提供读写访问功能。

**优点：**

- 具备块存储的读写高速。
- 具备文件存储的共享等特性。

**使用场景：** (适合更新变动较少的数据)

- 图片存储。
- 视频存储。
- …

## Ceph 对象存储 (radosgw)

Ceph 对象网关是一个构建在 `librados` 之上的对象存储接口，它为应用程序访问Ceph 存储集群提供了一个 RESTful 风格的网关 。 Ceph 对象存储支持 2 种接口：

1. **兼容S3:** 提供了对象存储接口，兼容大部分 亚马逊S3 RESTful 接口。
2. **兼容Swift:** 提供了对象存储接口，兼容大部分 Openstack Swift 接口。

Ceph 对象存储使用 Ceph 对象网关守护进程（ `radosgw` ），它是个与 Ceph 存储集群交互的 FastCGI 模块。因为它提供了与 OpenStack Swift 和 Amazon S3 兼容的接口， RADOS 要有它自己的用户管理。 Ceph 对象网关可与 Ceph FS 客户端或 Ceph 块设备客户端共用一个存储集群。 S3 和 Swift 接口共用一个通用命名空间，所以你可以用一个接口写入数据、然后用另一个接口取出数据。

![radosgw](http://docs.ceph.org.cn/_images/ditaa-50d12451eb76c5c72c4574b08f0320b39a42e5f1.png)

Ceph 对象存储**不使用** Ceph 元数据服务器。

对象存储的应用场景：

1）资源分发下载

网站或者app需要上传、下载和分发视频图片等

分发和下载app安装包等

2）网盘

可以对用户提供网盘服务，用户可以通过网盘存储自己任何格式的文件

ceph对象存储目前已经有厂商在使用，但是大多会基于网关等做些优化以适应自己的使用场景。