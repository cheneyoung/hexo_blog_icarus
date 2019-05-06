---
title: cephfs_performance_optimization
comments: true
author: zuoyang
type: 原创
toc: true
categories:
  - ceph
tags:
  - cephfs
abbrlink: 63586
date: 2019-01-09 00:02:07
---

# 背景

最近由于工作需要，需要使用cephfs替换nas，因此搭建了ceph集群，并在此基础上创建了cephfs文件系统供其他同学使用，刚开始的时候还好，但是随着使用的用户数及文件数目越来越多，cephfs的性能问题就逐渐凸显出来，他们反馈在cephfs客户端上操作时会出现以下两种情况：

1. 通过vim编辑文件时十分卡顿，打开或者保存文件有很明显的延时；
2. 训练模型时，读取数据的速度特别慢，读取4000张图片大约需要120s的时间，IO性能非常差。



# 本次优化参数如下：

```shell
osd journal size = 20000 #默认5120                      #osd journal大小
osd mkfs type = xfs                                     #格式化系统类型

filestore xattr use omap = true                         #默认false#为XATTRS使用object map，EXT4文件系统时使用，XFS或者btrfs也可以使用
filestore min sync interval = 10                        #默认0.1#从日志到数据盘最小同步间隔(seconds)
filestore max sync interval = 15                        #默认5#从日志到数据盘最大同步间隔(seconds)
filestore queue max ops = 25000                        #默认500#数据盘最大接受的操作数
filestore queue max bytes = 1048576000      #默认100   #数据盘一次操作最大字节数(bytes
filestore queue committing max ops = 50000 #默认500     #数据盘能够commit的操作数
filestore queue committing max bytes = 10485760000 #默认100 #数据盘能够commit的最大字节数(bytes)
filestore split multiple = 16000 #默认值2                  #前一个子目录分裂成子目录中的文件的最大数量
filestore merge threshold = 40 #默认值10               #前一个子类目录中的文件合并到父类的最小数量
filestore fd cache size = 204800 #默认值128  #对象文件句柄缓存大小
filestore omap header cache size = 204800
filestore fiemap = true    #默认值false  #读写分离
filestore ondisk finisher threads = 2  #默认值为1
filestore apply finisher threads = 2   #默认值为1

ms_async_op_threads = 5     #默认值3 
ms_dispatch_throttle_bytes = 104857600000  #默认值100 << 20

journal max write bytes = 1073714824 #默认值1048560    #journal一次性写入的最大字节数(bytes)
journal max write entries = 10000 #默认值100         #journal一次性写入的最大记录数
journal queue max ops = 50000  #默认值50            #journal一次性最大在队列中的操作数
journal queue max bytes = 10485760000 #默认值33554432   #journal一次性最大在队列中的字节数(bytes)
osd max write size = 512 #默认值90                   #OSD一次可写入的最大值(MB)
osd client message size cap = 2147483648 #默认值100    #客户端允许在内存中的最大数据(bytes)
osd deep scrub stride = 131072 #默认值524288         #在Deep Scrub时候允许读取的字节数(bytes)
osd op threads = 16 #默认值2                         #并发文件系统操作数
osd disk threads = 4 #默认值1                        #OSD密集型操作例如恢复和Scrubbing时的线程
osd map cache size = 1024 #默认值500                 #保留OSD Map的缓存(MB)
osd map cache bl size = 128 #默认值50                #OSD进程在内存中的OSD Map缓存(MB)
osd mount options xfs = "rw,noexec,nodev,noatime,nodiratime,nobarrier" #默认值rw,noatime,inode64  #Ceph OSD xfs Mount选项
osd recovery op priority = 2 #默认值10              #恢复操作优先级，取值1-63，值越高占用资源越高
osd recovery max active = 10 #默认值15              #同一时间内活跃的恢复请求数 
osd max backfills = 4  #默认值10                  #一个OSD允许的最大backfills数
osd min pg log entries = 30000 #默认值3000           #修建PGLog是保留的最大PGLog数
osd max pg log entries = 100000 #默认值10000         #修建PGLog是保留的最大PGLog数
osd mon heartbeat interval = 40 #默认值30            #OSD ping一个monitor的时间间隔（默认30s）
ms dispatch throttle bytes = 1048576000 #默认值 104857600 #等待派遣的最大消息数
objecter inflight ops = 819200 #默认值1024           #客户端流控，允许的最大未发送io请求数，超过阀值会堵塞应用io，为0表示不受限
osd op log threshold = 50 #默认值5                  #一次显示多少操作的log
```

## 查看各进程下的参数

### [mon]

```shell
sudo ceph daemon mon.`hostname` config show | grep parametername
或者
sudo ceph daemon mon.`hostname` get parametername
```

example：

```shell
$sudo ceph daemon mon.`hostname` config show | grep filestore_op
    "filestore_op_thread_suicide_timeout": "180",
    "filestore_op_thread_timeout": "60",
    "filestore_op_threads": "32",
 
$sudo ceph daemon mon.`hostname` config get filestore_op_threads
{
    "filestore_op_threads": "32"
}
```

### [mds]

```shell
sudo ceph daemon mds.`hostname` config show | grep parametername
或者
sudo ceph daemon mds.`hostname` get parametername
```

example：

```shell
$sudo ceph daemon mds.`hostname` config show | grep mds_cache
    "mds_cache_memory_limit": "1073741824",
    "mds_cache_mid": "0.700000",
    "mds_cache_reservation": "0.050000",
    "mds_cache_size": "0",

$sudo ceph daemon mds.`hostname` config get mds_cache_memory_limit
{
    "mds_cache_memory_limit": "1073741824"
}
```

### [osd]

```shell
sudo ceph daemon ods.$id config show | grep parametername
或者
sudo ceph daemon ods.$id config get parametername
```

example：

```shell
$sudo ceph daemon osd.3 config show | grep osd_op_th
    "osd_op_thread_suicide_timeout": "150",
    "osd_op_thread_timeout": "15",

$sudo ceph daemon osd.3 config get osd_op_thread_suicide_timeout
{
    "osd_op_thread_suicide_timeout": "150"
}
```

###  

## 修改各进程下的参数

### 修改所有进程的参数值(使用tell命令)

#### [mon]

```shell
sudo ceph tell mon.* injectargs '--parametername=value'
```

example:

修改所有mon进程的osd_pool_default_size参数值为3

```
sudo ceph tell mon.* injectargs '--osd_pool_default_size = 3'
```

#### [mds]

```shell
sudo ceph tell mds.* injectargs '--parametername=value'
```

example:

修改所有mds进程的mds_cache_memory_limit参数值为1073741824

```
sudo ceph tell mds.* injectargs '--mds_cache_memory_limit=1073741824'
```

#### [osd]

```shell
sudo ceph tell osd.* injectargs '--parametername=value'
```

example:

修改所有osd进程的filestore_op_threads参数值为32

```
sudo ceph tell osd.* injectargs '--filestore_op_threads=32'
```

####  

### 修改单个进程的参数值

#### [mon]

```shell
sudo ceph daemon mon.`hostname` set parametername value
```

example:

修改mon.`hostname`进程的osd_pool_default_size参数值为3

```
sudo ceph daemon mon.`hostname` set osd_pool_default_size  3
```

#### [mds]

```shell
sudo ceph daemon mds.`hostname` set parametername value
```

example:

修改mds.`hostname`进程的mds_cache_memory_limit参数值为1073741824

```
sudo ceph daemon mds.`hostname` set mds_cache_memory_limit 1073741824
```

#### [osd]

```shell
sudo ceph daemon osd.$id set parametername value
```

example:

修改所有osd2进程的filestore_op_threads参数值为32

```shell
sudo ceph daemon osd.2 set filestore_op_threads 32
```



# 各参数介绍

## osd配置优化

### [osd] - filestore

| 参数名                               | 描述                                                         | 默认值    | 建议值      |
| ------------------------------------ | ------------------------------------------------------------ | --------- | ----------- |
| filestore xattr use omap             | 为XATTRS使用object map，EXT4文件系统时使用，XFS或者btrfs也可以使用 | false     | true        |
| filestore max sync interval          | 从日志到数据盘最大同步间隔(seconds)                          | 5         | 15          |
| filestore min sync interval          | 从日志到数据盘最小同步间隔(seconds)                          | 0.1       | 10          |
| filestore queue max ops              | 数据盘最大接受的操作数                                       | 500       | 25000       |
| filestore queue max bytes            | 数据盘一次操作最大字节数(bytes)                              | 100 << 20 | 10485760    |
| filestore queue committing max ops   | 数据盘能够commit的操作数                                     | 500       | 5000        |
| filestore queue committing max bytes | 数据盘能够commit的最大字节数(bytes)                          | 100 << 20 | 10485760000 |
| filestore op threads                 | 并发文件系统操作数                                           | 2         | 32          |
|                                      |                                                              |           |             |

------

- 调整omap的原因主要是EXT4文件系统默认仅有4K
- filestore queue相关的参数对于性能影响很小，参数调整不会对性能优化有本质上提升

#### filestore merge/split配置参数

```shell
filestore_merge_threshold = 40       默认值 10
filestore_split_multiple = 16000      默认值 2
```

这两个参数是管理filestore的目录分裂/合并的，filestore的每个目录允许的最大文件数为：
`filestore_split_multiple * abs(filestore_merge_threshold) * 16`

在RGW的小文件应用场景，会很容易达到默认配置的文件数（320），若在写的过程中触发了filestore的分裂，则会非常影响filestore的性能；

每次filestore的目录分裂，会依据如下规则分裂为多层目录，最底层16个子目录：
例如PG 31.4C0, hash结尾是4C0，若该目录分裂，会分裂为 `DIR_0/DIR_C/DIR_4/{DIR_0, DIR_F}`；

原始目录下的object会根据规则放到不同的子目录里，object的名称格式为: `*__head_xxxxX4C0_*`，分裂时候X是几，就放进子目录DIR_X里。比如object：`*__head_xxxxA4C0_*`, 就放进子目录 `DIR_0/DIR_C/DIR_4/DIR_A` 里；

**解决办法：**

1. 增大merge/split配置参数的值，使单个目录容纳更多的文件；
2. `filestore_merge_threshold`配置为负数；这样会提前触发目录的预分裂，避免目录在某一时间段的集中分裂，详细机制没有调研；
3. 创建pool时指定`expected-num-objects`；这样会依据目录分裂规则，在创建pool的时候就创建分裂的子目录，避免了目录分裂对filestore性能的影响；

#### filestore fd cache配置参数

```
filestore_fd_cache_shards =  32     默认值 16     // FD number of shards
filestore_fd_cache_size = 204800     默认值 128  // FD lru size

```

filestore的fd cache是加速访问filestore里的file的，在非一次性写入的应用场景，增大配置可以很明显的提升filestore的性能；

#### filestore finisher threads配置参数

```
filestore_ondisk_finisher_threads = 2   默认值 1
filestore_apply_finisher_threads = 2   默认值 1
```

这两个参数定义filestore commit/apply的finisher处理线程数，默认都为1，任何IO commit/apply完成后，都需要经过对应的ondisk/apply finisher thread处理；

在使用普通HDD时，磁盘性能是瓶颈，单个finisher thread就能处理好；
但在使用高速磁盘的时候，IO完成比较快，单个finisher thread不能处理这么多的IO commit/apply reply，它会成为瓶颈；所以在jewel版本里引入了finisher thread pool的配置，这里一般配置为2即可；



### [osd] - journal

| 参数名                    | 描述                                     | 默认值   | 建议值      |
| ------------------------- | ---------------------------------------- | -------- | ----------- |
| osd journal size          | OSD日志大小(MB)                          | 5120     | 20000       |
| journal max write bytes   | journal一次性写入的最大字节数(bytes)     | 10 << 20 | 1073714824  |
| journal max write entries | journal一次性写入的最大记录数            | 100      | 10000       |
| journal queue max ops     | journal一次性最大在队列中的操作数        | 500      | 50000       |
| journal queue max bytes   | journal一次性最大在队列中的字节数(bytes) | 10 << 20 | 10485760000 |

------

- Ceph OSD Daemon stops writes and synchronizes the journal with the filesystem, allowing Ceph OSD Daemons to trim operations from the journal and reuse the space.
- 上面这段话的意思就是，Ceph OSD进程在往数据盘上刷数据的过程中，是停止写操作的。



### [osd] - osd config tuning

| 参数名                      | 描述                                     | 默认值             | 建议值                                       |
| --------------------------- | ---------------------------------------- | :----------------- | -------------------------------------------- |
| osd max write size          | OSD一次可写入的最大值(MB)                | 90                 | 512                                          |
| osd client message size cap | 客户端允许在内存中的最大数据(bytes)      | 524288000          | 2147483648                                   |
| osd deep scrub stride       | 在Deep Scrub时候允许读取的字节数(bytes)  | 524288             | 131072                                       |
| osd op threads              | OSD进程操作的线程数                      | 2                  | 8                                            |
| osd disk threads            | OSD密集型操作例如恢复和Scrubbing时的线程 | 1                  | 4                                            |
| osd map cache size          | 保留OSD Map的缓存(MB)                    | 500                | 1024                                         |
| osd map cache bl size       | OSD进程在内存中的OSD Map缓存(MB)         | 50                 | 128                                          |
| osd mount options xfs       | Ceph OSD xfs Mount选项                   | rw,noatime,inode64 | rw,noexec,nodev,noatime,nodiratime,nobarrier |
| osd_op_num_shards           |                                          | 5                  | 32                                           |
|                             |                                          |                    |                                              |

------

- 增加osd op threads和disk threads会带来额外的CPU开销

`osd_op_threads`：对应的work queue有`peering_wq`（osd peering请求），`recovery_gen_wq`（PG recovery请求）；
`osd_disk_threads`：对应的work queue为 `remove_wq`（PG remove请求）；
`osd_op_num_shards`和`osd_op_num_threads_per_shard`：对应的thread pool为`osd_op_tp`，work queue为`op_shardedwq`；

处理的请求包括：

1. `OpRequestRef`
2. `PGSnapTrim`
3. `PGScrub`

调大`osd_op_num_shards`可以增大osd ops的处理线程数，增大并发性，提升OSD性能；



### [osd] - recovery tuning

| 参数名                   | 描述                                         | 默认值 | 建议值 |
| ------------------------ | -------------------------------------------- | ------ | ------ |
| osd recovery op priority | 恢复操作优先级，取值1-63，值越高占用资源越高 | 10     | 4      |
| osd recovery max active  | 同一时间内活跃的恢复请求数                   | 15     | 10     |
| osd max backfills        | 一个OSD允许的最大backfills数                 | 10     | 4      |



## OS端配置

### Kernel pid max (线上机器无法修改)

```bash
echo 4194303 > /proc/sys/kernel/pid_max
```

### 设置MTU (没有修改)

交换机端需要支持该功能，系统网卡设置才有效果

配置文件追加MTU=9000

### read_ahead(挂载客户端机器可修改)

通过数据预读并且记载到随机访问内存方式提高磁盘读操作

```bash
echo "8192" > /sys/block/sda/queue/read_ahead_kb
```

### swappiness(不用修改，OS已默认)

主要控制系统对swap的使用

```bash
echo "vm.swappiness = 0" > /etc/sysctl.conf ;  
sysctl –p
```

### I/O Scheduler(不用修改，OS已默认)

SSD要用noop，SATA/SAS使用deadline

```bash
echo "deadline" >/sys/block/sd[x]/queue/scheduler
echo "noop" >/sys/block/sd[x]/queue/scheduler
```