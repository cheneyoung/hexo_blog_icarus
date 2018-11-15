---
title: 手动部署ceph集群
tags: ceph
categories:
  - ceph
abbrlink: c3d7e91e
date: 2018-11-15 15:54:19
---

## 1、机器选择

### 1.1 系统要求

ceph 最新 LTS 版本 (luminous) 推荐 linux 内核版本 `4.1.4` 及以上, 最低版本要求 `3.10.*`。


### 1.2 服务器

这里选择三台服务器来部署ceph集群，一台Mon+五台OSD

------

| 节点            | 服务             | cluster network  | public network   |
| --------------- | ---------------- | ---------------- | ---------------- |
| 192.168.226.20  | osd.1,mon.node2  | 192.168.226.0/24 | 192.168.226.0/24 |
| 192.168.226.21  | osd.4            | 192.168.226.0/24 | 192.168.226.0/24 |
| 192.168.226.22  | osd.2, mon.node1 | 192.168.226.0/24 | 192.168.226.0/24 |
| 192.168.226.96  | osd.3,mon.node3  | 192.168.226.0/24 | 192.168.226.0/24 |
| 192.168.226.106 | osd.0            | 192.168.226.0/24 | 192.168.226.0/24 |

每个节点只能使用1块磁盘部署osd。所以，集群共有5个`osd`进程，3个`monitor`进程。

cluster network 是处理osd间的数据复制，数据重平衡，osd进程心跳检测的网络，其不对外提供服务，只在各个osd节点间通信，本文使用eth1网卡作为cluster network，三个节点网卡eth1桥接到同一个网桥br1上



## 2、环境配置 

配置每个节点的host文件，在 `/etc/hosts`文件中添加如下内容：

```shell
192.168.226.20 ceph-1
192.168.226.22 ceph-2
192.168.226.96 ceph-3
```

### 2.2 ceph节点安装

你的管理节点必须能够通过 SSH 无密码地访问各 Ceph 节点。如果 `ceph-deploy` 以某个普通用户登录，那么这个用户必须有无密码使用 `sudo` 的权限。

#### 2.2.1 安装 NTP

我们建议在所有 Ceph 节点上安装 NTP 服务（特别是 Ceph Monitor 节点），以免因时钟漂移导致故障，详情见[时钟](http://docs.ceph.org.cn/rados/configuration/mon-config-ref#clock)。

```shell
sudo yum install ntp ntpdate ntp-doc
```

确保在各 Ceph 节点上启动了 NTP 服务，并且要使用同一个 NTP 服务器，详情见 [NTP](http://www.ntp.org/) 。

#### 2.2.2 安装 SSH 服务器

在**所有 Ceph** 节点上执行如下步骤：

1. 在各 Ceph 节点安装 SSH 服务器（如果还没有）

   ```shell
   sudo yum install openssh-server
   ```

2. 确保**所有** Ceph 节点上的 SSH 服务器都在运行。

#### 2.2.3 安装ceph

由于蚂蚁内部物理机不能访问外网，使用以下步骤安装ceph。

在**所有Ceph**节点上执行如下步骤：

下载ceph所有的依赖rpm，并解压缩

```shell
sudo wget http://qianli-lzh.oss-cn-hangzhou-zmf.aliyuncs.com/bill_inference_public%2Fceph.tar
sudo tar -xvf bill_inference_public%2Fceph.tar
```

手动安装所有的rpm

```shell
sudo rpm -ivh --force --nodeps ceph/*.rpm
```

验证ceph是否正确安装

```shell
ceph -v
ceph version 12.2.8 (ae699615bac534ea496ee965ac6192cb7e0e07c0) luminous (stable)
```

#### 2.2.4 关闭防火墙

```shell
sudo sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sudo setenforce 0
sudo systemctl stop firewalld 
sudo systemctl disable firewalld
```

## 3、集群搭建

### 3.1 搭建Mon集群 (使用admin账户)

**创建配置文件**

在**每台节点机器**上创建配置文件`/etc/ceph/ceph.conf`：

```shell
[global]
fsid = 932XXXXX-fba7-XXXX-9526-a858c613f468
mon initial members = e15p13447.ew9
mon host = 192.168.226.20,192.168.226.22,192.168.226.96
rbd default features = 1
auth_cluster_required = none
auth_service_required = none
auth_client_required = none
public network = 192.168.226.0/24
cluster network = 192.168.226.0/24
osd journal size = 1024
osd pool default size = 2
osd pool default min size = 1
osd pool default pg num = 128
osd pool default pgp num = 128
osd crush chooseleaf type = 1
mon_max_pg_per_osd = 200

[mds.ceph-1]
host = ceph-1
[mds.ceph-2]
host = ceph-2
[mds.ceph-3]
host = ceph-3

[mon]
mon allow pool delete = true
```

其中 `fsid` 是为集群分配的一个 uuid, 初始化 mon 节点其实只需要这一个配置就够了。
`mon host` 配置 ceph 命令行工具访问操作 ceph 集群时查找 mon 节点入口。
ceph 集群可包含多个 mon 节点实现高可用容灾, 避免单点故障。
`rbd default features = 1` 配置 rbd 客户端创建磁盘时禁用一些需要高版本内核才能支持的特性。

#### 3.1.2 主mon节点 （192.168.226.20）

1、为此集群创建密钥环、并生成Monitor密钥 (3台机器一样)

```shell
sudo ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
```

2、生成管理员密钥环，生成 `client.admin` 用户并加入密钥环 (3台机器一样)

```shell
sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *' 
```

3、把 `client.admin` 密钥加入 `ceph.mon.keyring`  (3台机器一样)

```shell
sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
```

4、用规划好的主机名、对应 IP 地址、和 FSID 生成一个Monitor Map，并保存为 `/tmp/monmap`

```shell
host_name=`hostname`
sudo monmaptool --create --add $host_name 192.168.226.20  --fsid 932XXXXX-fba7-XXXX-9526-a858c613f468 /tmp/monmap --clobber
```

5、在Monitor主机上分别创建数据目录

```shell
host_name=`hostname`
#在admin账户下
sudo mkdir /var/lib/ceph/mon/ceph-$host_name/
```

6、用Monitor Map和密钥环组装守护进程所需的初始数据

```shell
sudo ceph-mon --mkfs -i $host_name --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
```

7、建一个空文件 `done` ，表示监视器已创建、可以启动了

```shell
sudo touch /var/lib/ceph/mon/ceph-$host_name/done
```

8、启动Monitor

```shell
#sudo ceph-mon -f --cluster ceph --id $host_name &
sudo cp /usr/lib/systemd/system/ceph-mon@.service /usr/lib/systemd/system/ceph-mon@$host_name.service
sudo systemctl start ceph-mon@$host_name
sudo systemctl enable ceph-mon@$host_name
```

9、确认下集群在运行

```shell
ceph -s
```

事例：

```shell
  cluster:
    id:     932XXXXX-fba7-XXXX-9526-a858c613f468
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-1,ceph-2,ceph-3
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   0B used, 0B / 0B avail
    pgs:     
```

#### 3.1.2 从mon节点 (192.168.226.22 & 192.168.226.96)

```shell
host_name=`hostname`
sudo ceph mon getmap -o /tmp/monmap
sudo rm -rf /var/lib/ceph/mon/ceph-$host_name
sudo ceph-mon -i $host_name --mkfs --monmap /tmp/monmap
sudo chown -R ceph:ceph /var/lib/ceph/mon/ceph-$host_name/
#nohup ceph-mon -f --cluster ceph --id $host_name --setuser ceph --setgroup ceph &
#ceph-mon -f --cluster ceph --id $host_name &
sudo cp /usr/lib/systemd/system/ceph-mon@.service /usr/lib/systemd/system/ceph-mon@$host_name.service
sudo systemctl start ceph-mon@$host_name
sudo systemctl enable ceph-mon@$host_name
```



### 3.2 创建ceph-mgr

#### 3.2.1 创建用户 openstack 用于 MGR 监控

```shell
ceph auth get-or-create mgr.openstack mon 'allow *' osd 'allow *' mds 'allow *'
输出：
[mgr.openstack]
        key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxugvXkLfgauLA==
```

需要将之前创建的用户密码存放至对应位置

```shell
mkdir /var/lib/ceph/mgr/ceph-openstack
ceph auth get mgr.openstack -o  /var/lib/ceph/mgr/ceph-openstack/keyring
exported keyring for mgr.openstack
```

#### 3.2.2 启动mgr

```shell
ceph-mgr -i openstack
```

监控状态

```shell
$ceph -s
  cluster:
    id:     932e88a6-fba7-45a9-9526-a858c613f468
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-1,ceph-2,ceph-3
    mgr: openstack(active)
    mds: cephfs-1/1/1 up  {0=2=up:active}, 2 up:standby
    osd: 3 osds: 3 up, 3 in
 
  data:
    pools:   2 pools, 256 pgs
    objects: 21 objects, 3.04KiB
    usage:   3.32GiB used, 1.17TiB / 1.17TiB avail
    pgs:     256 active+clean
```

当 mgr 服务被激活之后, service 中 mgr 会显示 mgr-$name(active) 
data 部分信息将变得可用

### 3.3 手动搭建osd集群(三台机器上做相同的操作，注意osd_id的变化)

添加一个新osd，`id`可以省略，ceph会自动使用最小可用整数，第一个osd从0开始

```shell
#ceph osd create {id}
ceph osd create
0
```

#### 3.3.1 初始化osd目录

创建osd.0目录，目录名格式`{cluster-name}-{id}`

```shell
#mkdir /var/lib/ceph/osd/{cluster-name}-{id}
sudo mkdir /var/lib/ceph/osd/ceph-0
```

挂载osd.0的数据盘/dev/sdb2

```shell
sudo mkfs.xfs /dev/sdb2
sudo mount /dev/sdb2 /var/lib/ceph/osd/ceph-0
```

初始化osd数据目录

```shell
# sudo ceph-osd -i {id} --mkfs --mkkey
sudo ceph-osd -i 0 --mkfs --mkkey
#--mkkey要求osd数据目录为空
#这会创建osd.0的keyring /var/lib/ceph/osd/ceph-0/keyring
```

初始化后，默认使用普通文件/var/lib/ceph/osd/ceph-3/journal作为osd.0的journal分区，普通文件作为journal分区性能不高，若只是测试环境，可以跳过更改journal分区这一步骤

#### 3.3.2 创建journal

生成journal分区，一般选ssd盘作为journal分区，这里使用ssd的/dev/sdb1分区作为journal

使用fdisk工分出磁盘/dev/sdb1,

```shell
#清除磁盘所有分区(重新添加时需要)
#sgdisk --zap-all --clear --mbrtogpt /dev/sdb
#生成分区/dev/sdb1的uuid
#uuidgen
#b3897364-8807-48eb-9905-e2c8400d0cd4
#创建分区
#1:0:+100G 表示创建第一个分区，100G大小
#sudo sgdisk --new=1:0:+100G --change-name=1:'ceph journal' --partition-guid=1:b3897364-8807-48eb-9905-e2c8400d0cd4 --typecode=1:b3897364-8807-48eb-9905-e2c8400d0cd4 --mbrtogpt -- /dev/vdf
#格式化
sudo mkfs.xfs /dev/sdb1
sudo rm -f /var/lib/ceph/osd/ceph-4/journal 
#查看分区对应的partuuid， 找出/dev/sdb1对应的partuuid
sudo blkid
sudo ln -s /dev/disk/by-partuuid/b3897364-8807-48eb-9905-e2c8400d0cd4 /var/lib/ceph/osd/ceph-0/journal

sudo chown ceph:ceph -R /var/lib/ceph/osd/ceph-0
sudo chown ceph:ceph /var/lib/ceph/osd/ceph-0/journal
#初始化新的journal
sudo ceph-osd --mkjournal -i 0
sudo chown ceph:ceph /var/lib/ceph/osd/ceph-0/journal
```



#### 3.3.3 注册osd.{id}，id为osd编号，默认从0开始

```shell
# sudo ceph auth add osd.{id} osd 'allow *' mon 'allow profile osd' -i /var/lib/ceph/osd/ceph-{id}/keyring
sudo ceph auth add osd.0 osd 'allow *' mon 'allow profile osd' -i /var/lib/ceph/osd/ceph-0/keyring
#ceph auth list 中出现osd.0
```

#### 3.3.4 加入crush map

这是m1上新创建的第一个osd，CRUSH map中还没有m1节点，因此首先要把m1节点加入CRUSH map，同理，m2/m3节点也需要加入CRUSH map

```shell
#ceph osd crush add-bucket {hostname} host
sudo ceph osd crush add-bucket `hostname` host
```

然后把三个节点移动到默认的root `default`下面

```shell
sudo ceph osd crush move `hostname` root=default
```

添加osd.0到CRUSH map中的m1节点下面，加入后，osd.0就能够接收数据

```shell
#ceph osd crush add osd.{id} 0.4 root=sata rack=sata-rack01 host=sata-node5
sudo ceph osd crush add osd.4 1.7 root=default host=`hostname`
#0.4为此osd在CRUSH map中的权重值，它表示数据落在此osd上的比重，是一个相对值，一般按照1T磁盘比重值为1来计算，这里的osd数据盘1.7，所以值为1.7  
```

此时osd.0状态是`down`且`in`，`in`表示此osd位于CRUSH map，已经准备好接受数据，`down`表示osd进程运行异常，因为我们还没有启动osd.0进程

#### 3.3.5 启动ceph-osd进程

需要向systemctl传递osd的`id`以启动指定的osd进程，如下，我们准备启动osd.0进程

```shell
#systemctl start ceph-osd@{id}  id表示osd编号，从数字0开始
sudo cp /usr/lib/systemd/system/ceph-osd@.service /usr/lib/systemd/system/ceph-osd@0.service
sudo systemctl start ceph-osd@0
sudo systemctl enable ceph-osd@0
#sudo ceph-osd -i 0
```

上面就是添加osd.0的步骤，然后可以接着在其他`hostname`节点上添加osd.{1,2}，添加了这3个osd后，可以查看集群状态 ceph -s。

### 3.4 搭建MDS

创建目录：

```shell
sudo mkdir /var/lib/ceph/mds/ceph-`hostname`
sudo chown ceph:ceph -R /var/lib/ceph/mds/ceph-`hostname`
```

在ceph.conf中添加如下信息：

```shell
[mds.{id}]
host = {id}
例如：
[mds.0]
host = 0
```

启动mds

```shell
#ceph-mds --cluster {cluster-name} -i {id} -m {mon-hostname}:{mon-port} [-f]
sudo cp /usr/lib/systemd/system/ceph-mds@.service /usr/lib/systemd/system/ceph-mds@`hostname`.service 
sudo systemctl start ceph-mds@`hostname`
sudo systemctl enable ceph-mds@`hostname`
#ceph-mds --cluster ceph -i 0 -m e15p13447.ew9:6789
```

查看mds状态

```shell
ceph mds stat
cephfs-1/1/1 up  {0=1=up:active}, 2 up:standby
```

至此ceph集群搭建完成。