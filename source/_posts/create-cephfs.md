---
title: 第四篇：创建cephfs服务
comments: true
author: zuoyang
type: 原创
toc: true
categories:
  - ceph
tags:
  - ceph
  - cephfs
abbrlink: 4a310db5
date: 2018-11-23 12:06:35
---

基于[上一篇](https://www.zuoyangblog.com/post/28a4bfb3.html)，我们搭建好了一个健康的ceph集群：

- 3个mon节点组成的mon集群
- 9个osd节点组成的osd集群
- 3个mgr节点(ceph luminous版本才有的)
- 3个mds服务(cephfs使用)

```shell
[root@ceph-1 ceph]# ceph -s
  cluster:
    id:     c165f9d0-88df-48a7-8cc5-11da82f99c93
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph-1,ceph-2,ceph-3
    mgr: ceph-1(active), standbys: admin, ceph-2, ceph-3
    osd: 9 osds: 9 up, 9 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   965MiB used, 18.0TiB / 18.0TiB avail
    pgs:

```

本篇主要讲如何基于ceph集群搭建cephfs服务。

# cephFS简介

The [Ceph Filesystem](http://docs.ceph.com/docs/mimic/glossary/#term-ceph-filesystem) (Ceph FS) is a POSIX-compliant filesystem that uses a Ceph Storage Cluster to store its data. 

{% asset_img  cephfs.jpg  %}

Ceph文件系统（CephFS）提供符合POSIX标准的文件系统作为服务，该服务在基于对象的Ceph存储集群之上。 CephFS文件被映射到Ceph存储在Ceph存储集群中的对象。 Ceph客户端有两种挂载cephfs的方式：基于内核和FUSE.下图为cephfs的结构图：

{% asset_img  ceph_archite.jpg  %}

Ceph文件系统服务包括与Ceph存储集群一起部署的Ceph元数据服务器（MDS）。 MDS存储了文件系统里的所有元数据（目录，文件所有权，访问模式等）。 MDS（ceph-mds的守护进程）将一些简单的文件系统操作（例如列出目录或更改目录（ls，cd）等）与Ceph OSD守护进程隔离，这样可以减少对OSD进程造成不必要的负担。 因此，将元数据与数据分离意味着Ceph文件系统可以提供高性能服务，而不会对Ceph存储集群造成负担。

- MDS：存储metadata(元数据)
- OSD：存储file data(数据)

ceph-mds可以作为单个进程运行，也可以分发到多个物理机器，以实现高可用性或可伸缩性。

- 高可用性：集群中处于standby状态的ceph-mds进程随时可以取代失败的active状态的ceph-mds，当active状态的ceph-mds进程失败时，ceph-mon很容易就可以把standby状态的mds转换成active状态，从而维持集群的可用性
- 可扩展性：集群中可以有多个active状态的ceph-mds进程，它们会将目录树拆分为子树（以及单个繁忙目录的分片），从而有效地平衡所有活动服务器之间的负载。



# 新建ceph client虚拟机

根据[第二篇](https://www.zuoyangblog.com/post/ad293d8.html)的方法，新建一个虚拟机，我们命名为ceph-client.

安装好之后配置相应的网络，具体方法详见[这里](https://www.zuoyangblog.com/post/ad293d8.html)

## 修改ifcfg-enp0s3

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

将最后一行的`ONBOOT=no`改为`ONBOOT=yes`,添加IPADDR，NETMASK，这个是`网卡1`，用于给虚拟机上网。

## 修改ifcfg-enp0s8

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.56.200
NETMASK=255.255.255.0
```

## 重启网卡并检查联网状态：

```shell
systemctl restart network
```

## 修改主机名`hostname`:

```shell
echo ceph-client > /etc/hostname
```

## 重启，通过本机终端登录

```shell
ssh root@192.168.56.200
```

## 修改yum源

```shell
yum clean all
curl http://mirrors.aliyun.com/repo/Centos-7.repo >/etc/yum.repos.d/CentOS-Base.repo
curl http://mirrors.aliyun.com/repo/epel-7.repo >/etc/yum.repos.d/epel.repo 
sed -i '/aliyuncs/d' /etc/yum.repos.d/CentOS-Base.repo
sed -i '/aliyuncs/d' /etc/yum.repos.d/epel.repo
yum makecache
```

## 配置ceph源

在`/etc/yum.repos.d/`目录下新增一个ceph源文件`ceph.repo`,并写入下面内容：

```shell
vim /etc/yum.repos.d/ceph.repo
#写入以下内容
[ceph]
name=ceph
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/x86_64/
gpgcheck=0
[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/noarch/
gpgcheck=0
[ceph-source]
name=ceph-source
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/SRPMS/
gpgcheck=0
```

## 安装软件

```shell
yum clean all && yum makecache
yum -y install yum-plugin-priorities 
yum -y install openssh-server
```



# 创建cephFS

登录mon集群的那三台机器中的一台即可，我这里登录ceph-1

```shell
ssh root@192.168.56.101
```

## 创建cephfs

creatring pools

```shell
[root@ceph-1 ceph]# ceph osd pool create cephfs_data 64
pool 'cephfs_data' created
[root@ceph-1 ceph]# ceph osd pool create cephfs_metadata 64
pool 'cephfs_metadata' created
```

creating a filesystem

```shell
[root@ceph-1 ceph]# ceph fs new cephfs cephfs_metadata cephfs_data
new fs with metadata pool 2 and data pool 1
[root@ceph-1 ceph]# ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]
```

一旦文件系统创建好之后，mds的状态就会发生变化，如下所示：

```shell
[root@ceph-1 ceph]# ceph mds stat
cephfs-1/1/1 up  {0=ceph-2=up:active}, 2 up:standby
```

再看看集群状态：

```shell
[root@ceph-1 ceph]# ceph -s
  cluster:
    id:     c165f9d0-88df-48a7-8cc5-11da82f99c93
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph-1,ceph-2,ceph-3
    mgr: ceph-1(active), standbys: admin, ceph-2, ceph-3
    mds: cephfs-1/1/1 up  {0=ceph-2=up:active}, 2 up:standby
    osd: 9 osds: 9 up, 9 in

  data:
    pools:   2 pools, 128 pgs
    objects: 21 objects, 3.14KiB
    usage:   967MiB used, 18.0TiB / 18.0TiB avail
    pgs:     128 active+clean

```

- mds进入active状态
- 生成了两个pool，128个pg

## 挂载cephfs

cephfs支持两种方式挂载：

- 内核驱动
- 使用fuse

这是官网给出的两种挂载方式的对比：

The **FUSE client** is the most accessible and the easiest to upgrade to the version of Ceph used by the storage cluster, while the **kernel client** will often give better performance.

The clients do not always provide equivalent functionality, for example the **fuse client** supports client-enforced quotas while the **kernel client** does not.

### 内核驱动 (with kernel)

ceph一个集群只支持创建一个cephfs,所以只要创建一次cephfs之后，就可以在客户端机器上创建目录并挂载到集群：

在客户端机器上进行如下操作

```shell
[root@ceph-client ~]# mkdir -p /mycephfs
[root@ceph-client ~]# mount -t ceph 192.168.56.101,192.168.56.102,192.168.56.103:6789:/ /mycephfs
```

查看挂载情况：

```shell
[root@ceph-client ~]# df -h
文件系统                                             容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root                               50G  1.6G   49G    4% /
devtmpfs                                             485M     0  485M    0% /dev
tmpfs                                                496M     0  496M    0% /dev/shm
tmpfs                                                496M  6.8M  490M    2% /run
tmpfs                                                496M     0  496M    0% /sys/fs/cgroup
/dev/sda1                                           1014M  129M  886M   13% /boot
/dev/mapper/centos-home                              347G   35M  347G    1% /home
tmpfs                                                100M     0  100M    0% /run/user/0
192.168.56.101,192.168.56.102,192.168.56.103:6789:/  5.4T     0  5.4T    0% /mycephfs
```

最后一行可以看到挂载成功。

### 使用fuse (with fuse)

使用fuse方式进行挂载，需要安装ceph-fuse工具，

```shell
[root@ceph-client ~]# yum install ceph-fuse
 ........
已安装:
  ceph-fuse.x86_64 2:12.2.9-0.el7

作为依赖被安装:
  fuse-libs.x86_64 0:2.9.2-10.el7    gperftools-libs.x86_64 0:2.6.1-1.el7    libibverbs.x86_64 0:15-7.el7_5    pciutils.x86_64 0:3.5.1-3.el7
  rdma-core.x86_64 0:15-7.el7_5

完毕！
```

```shell
[root@ceph-client ~]# ceph-fuse -v
ceph version 12.2.9 (9e300932ef8a8916fb3fda78c58691a6ab0f4217) luminous (stable)
```

ceph-fuse安装完成。

挂载cephfs

```shell
[root@ceph-client ~]# mkdir /fusefs
[root@ceph-client ~]# ceph-fuse -m 192.168.56.101:6789 /fusefs
ceph-fuse[10854]: starting ceph client
ceph-fuse[10854]: starting fuse
```

```shell
[root@ceph-client ~]# df -h
文件系统                                             容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root                               50G  1.6G   49G    4% /
devtmpfs                                             485M     0  485M    0% /dev
tmpfs                                                496M     0  496M    0% /dev/shm
tmpfs                                                496M  6.8M  490M    2% /run
tmpfs                                                496M     0  496M    0% /sys/fs/cgroup
/dev/sda1                                           1014M  129M  886M   13% /boot
/dev/mapper/centos-home                              347G   35M  347G    1% /home
tmpfs                                                100M     0  100M    0% /run/user/0
192.168.56.101,192.168.56.102,192.168.56.103:6789:/  5.4T     0  5.4T    0% /mycephfs
ceph-fuse                                            5.4T     0  5.4T    0% /fusefs
```

查看最后一行，说明挂载成功。

# 总结

本篇主要介绍了cephfs的架构以及在ceph集群上挂载cephfs的流程。