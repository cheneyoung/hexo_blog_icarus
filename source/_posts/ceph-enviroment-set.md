---
title: 第二篇：Ceph集群环境准备
comments: true
author: zuoyang
type: 原创
toc: true
categories:
  - ceph
tags:
  - ceph
  - virtualBox
abbrlink: ad293d8
date: 2018-11-20 18:28:27
---

第一篇简单介绍了Ceph的架构，让我们对Ceph有了一个初步的印象。

接下来，我将在MAC上介绍如何基于本机搭建ceph集群及cephfs、cephrgw、cephrbd服务。

集群规划：

- 生产环境
  - 至少3台物理机组成Ceph集群
  - 双网卡
- 测试环境
  - 1台主机也可以
  - 单网卡也可以

本文使用虚拟机搭建集群，集群设置如下：

- mon集群：
  - 3台虚拟机组成mon集群
- osd集群：
  - 3台虚拟机组成osd集群
  - 每台虚拟机上3个osd进程
- mgr集群：
  - 3个mgr进程

在部署之前，首先需要介绍前期环境的准备工作：

- 软件准备：VirtualBox，CentOS
- 安装虚拟机
- 克隆虚拟机
- 

# 软件准备

## 安装VirtualBox

点击 [VirtualBox Mac版本](http://download.virtualbox.org/virtualbox/5.1.6/VirtualBox-5.1.6-110634-OSX.dmg)，下载VirtualBox，然后双击下载好的软件，按照提示一直安装即可。

### 添加网络

安装好之后，需要添加一个网络，这样可以保证在虚拟机里也能正常上网。具体操作如下：

点击左上角的`VirtualBox->偏好设置->网络->仅主机(Host-Only)网络->`点击右边的绿色➕，一般会默认添加了`vboxnet0`，双击`vboxnet0`，可以看到网络的IP信息，默认为`192.168.56.1`，如下图：

{% asset_img  network.jpg 配置网络  %}

{% asset_img  ip.jpg 默认IP %}



## 下载CentOS镜像

点击[CentOS 7.5 镜像Minimal](<http://mirrors.aliyun.com/centos/7.5.1804/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso >) ，这里我们下载Minimal版本，后面操作的时候，我们直接通过配置虚拟机IP，使得本机可以直接ssh访问虚拟机进行操作。



# 安装CentOS虚拟机

## 创建虚拟机

- 打开VirtualBox，点击`新建`，这时会新建一个虚拟机，命名为`ceph-1`,类型选择`Linux`，版本选择`Linux 2.6/3.x/4.x 64bit`，具体见下图：

  {% asset_img new.jpg  新建虚拟机 %}

- 内存设置：默认值为1G，直接下一步即可，如下图:

  {% asset_img memory.jpg 设置内存大小 %}

- 创建虚拟硬盘：选择创建VDI，然后下一步，如下图:

  {% asset_img  disk.jpg  创建虚拟硬盘 %}

  {% asset_img virtual_disk.jpg  创建虚拟硬盘 %}

- 存储在物理盘：这里有两个选项-`动态分配`或`固定大小`，这里选择`动态分配`，见下图:

  {% asset_img  dynamic.jpg 动态分配 %}

  - `动态分配`，这种方式下，创建一个2T的磁盘，实际只会占用计算机几十MB的空间，实际使用多少空间，才会占用多少空间，相当于用时分配。
  - `固定大小`，这种方式下，创建多大的盘就会占用多大的空间。

- 文件位置与大小：将空间大小设置为100G，这是用于系统盘。然后点击创建即可:

  {% asset_img  disk_size.jpg 系统盘大小 %}

至此，虚拟机创建完成。



## 配置虚拟机

### 添加CentOS镜像

选择刚刚创建好的虚拟机，点击`设置`->`存储`->`控制器:IDE`->`没有盘片`，点击右侧的光盘按钮，将刚刚下载的CentOS的镜像添加进来，如下图所示：

{% asset_img addios.jpg  添加CentOS镜像 %}

### 添加3个2T磁盘

点击`控制器:SATA`旁边的方形加号，添加SATA盘，`创建新的虚拟盘-> VHD-> 动态分配 -> 2TB`.

步骤如下图：

{% asset_img  newvhd.jpg  添加虚拟磁盘 %}

{% asset_img  newvhd2.jpg  添加虚拟磁盘 %}

{% asset_img finish.jpg  完成虚拟盘创建 %}



### 配置网络

点击`设置-> 网络-> 网卡 1-> 连接方式 -> 网络地址转换(NAT)`，用于给VM上网，如下图所示：

{% asset_img adapter1.jpg 配置网卡1 %}

点击`网卡2 -> 勾选启用网络连接 -> 连接方式 -> 仅主机(Host-only)网络 -> 界面名称 -> vboxnet0`这里的`vboxnet0`是在上一步中添加的，如下图所示：

{% asset_img adapter2.jpg 配置网络 %}

虚拟机配置完成，接下来进行安装CentOS。



### 安装CentOS

启动刚刚创建的虚拟机，Install CentOS 7，一直往下点，然后出现下面的页面：

{% asset_img install_os1.jpg 安装CentOS1 %}

点击红框中的选项，然后Begin Install：

{% asset_img install_os2.jpg  安装CentOS2 %}

之后再设置root密码：

{% asset_img install_os3.jpg  设置root密码 %}

{% asset_img  user.jpg  添加用户 %}

{% asset_img user_set.jpg 设置账号密码 %}

之后一直等就可以了，最后重启机器，安装完成。



### 配置网卡

#### 修改ifcfg-enp0s3

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

将最后一行的`ONBOOT=no`改为`ONBOOT=yes`,添加IPADDR，NETMASK，这个是`网卡1`，用于给虚拟机上网。

#### 修改ifcfg-enp0s8

```shell
vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
#修改以下几个配置项
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.56.101 #因为vboxnet0的IP为192.168.56.1
NETMASK=255.255.255.0
```

重启网卡并检查联网状态：

```shell
systemctl restart network
ping www.baidu.com
PING www.a.shifen.com (115.239.211.112) 56(84) bytes of data.
64 bytes from 115.239.211.112 (115.239.211.112): icmp_seq=1 ttl=63 time=2.79 ms
64 bytes from 115.239.211.112 (115.239.211.112): icmp_seq=2 ttl=63 time=25.7 ms
64 bytes from 115.239.211.112 (115.239.211.112): icmp_seq=3 ttl=63 time=3.58 ms
64 bytes from 115.239.211.112 (115.239.211.112): icmp_seq=5 ttl=63 time=3.62 ms
64 bytes from 115.239.211.112 (115.239.211.112): icmp_seq=6 ttl=63 time=2.97 ms
64 bytes from 115.239.211.112 (115.239.211.112): icmp_seq=7 ttl=63 time=2.99 ms
```

修改主机名`hostname`:

```shell
echo ceph-1 > /etc/hostname
```

**然后重启机器，这样配置的hostname跟IP就会生效**

然后就可以通过本机终端直接ssh到虚拟机进行操作，可以方便的进行复制黏贴。

登录到虚拟机：

```shell
ssh root@192.168.56.101
The authenticity of host '192.168.56.101 (192.168.56.101)' can't be established.
ECDSA key fingerprint is SHA256:uuwVZ9O8+0KypxxJUgZLANVYMOFKY2QAd1Jv7Fa2fQE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.101' (ECDSA) to the list of known hosts.
root@192.168.56.101's password:
Last login: Wed Nov 21 01:08:21 2018
[root@ceph-1 ~]#
```

登录成功之后开始进行下面的操作。

### 修改yum源

这里将yum源修改成aliyun的源，指令如下:

```shell
yum clean all
curl http://mirrors.aliyun.com/repo/Centos-7.repo >/etc/yum.repos.d/CentOS-Base.repo
curl http://mirrors.aliyun.com/repo/epel-7.repo >/etc/yum.repos.d/epel.repo 
sed -i '/aliyuncs/d' /etc/yum.repos.d/CentOS-Base.repo
sed -i '/aliyuncs/d' /etc/yum.repos.d/epel.repo
yum makecache
```

安装软件：

```shell
yum -y install wget ntp vim 
```



## 虚拟机ceph软件安装

### 配置ceph源

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

### 安装ceph

执行以下命令安装ceph:

```shell
yum clean all && yum makecache
yum -y install yum-plugin-priorities 
yum -y install openssh-server
yum -y install ceph ceph-radosgw
```

验证ceph是否安装完成：

```shell
[root@ceph-1 ~]# ceph -v
ceph version 12.2.9 (9e300932ef8a8916fb3fda78c58691a6ab0f4217) luminous (stable)
```

说明ceph安装成功。

至此，第一台虚拟机创建完成，然后关机。



# 克隆虚拟机

## 克隆机器

点击刚刚创建好的虚拟机，然后右键点击复制，

重命名虚拟机为`ceph-2`，并勾选重新初始化MAC和网卡选项。

{% asset_img  clone2.jpg  %}

点击继续，然后勾选完全复制

{% asset_img  clone3.jpg  %}



## 修改机器配置

在VM上登陆`ceph-2`，修改`enp0s8`的IP。

```shell
vim /etc/sysconfig/network-scripts/ifcfg-enp0s8
IPADDR=192.168.56.102
```

修改`hostname`

```shell
echo ceph-2 > /etc/hostname
```

修改好之后重启，就可以通过本机终端ssh访问了

```shell
ssh root@192.168.56.102
The authenticity of host '192.168.56.102 (192.168.56.102)' can't be established.
ECDSA key fingerprint is SHA256:uuwVZ9O8+0KypxxJUgZLANVYMOFKY2QAd1Jv7Fa2fQE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.102' (ECDSA) to the list of known hosts.
root@192.168.56.102's password:
Last login: Wed Nov 21 02:14:58 2018
[root@ceph-2 ~]#
```

说明克隆成功。

同样的方法克隆`ceph-3`，将修改IP为`192.168.56.103`。

最后，将各个主机的IP加入各自的`/etc/hosts`中:

```
vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.101 ceph-1
192.168.56.102 ceph-2
192.168.56.103 ceph-3
```

最后重启所有主机。

本篇结束，下一篇将重点介绍ceph集群的手工搭建。