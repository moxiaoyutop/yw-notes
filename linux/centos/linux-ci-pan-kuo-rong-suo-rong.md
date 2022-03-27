# Linux-磁盘扩容缩容



## 1. 磁盘扩容

### 1.1. 扩容前查看

```bash
[root@k3s-work ~]# df -hT
Filesystem                        Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos_k3s--work-root xfs        36G  1.7G   34G   5% /
devtmpfs                          devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                             tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                             tmpfs     3.9G  9.0M  3.9G   1% /run
tmpfs                             tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                         xfs      1014M  142M  873M  14% /boot
/dev/mapper/centos_k3s--work-home xfs        18G   33M   18G   1% /home         # 增加此容量
tmpfs                             tmpfs     783M     0  783M   0% /run/user/0


[root@k3s-work ~]# fdisk -l

Disk /dev/sda: 64.4 GB, 64424509440 bytes, 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00050669

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200   125829119    61864960   8e  Linux LVM

Disk /dev/mapper/centos_k3s--work-root: 38.2 GB, 38235275264 bytes, 74678272 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos_k3s--work-swap: 6442 MB, 6442450944 bytes, 12582912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos_k3s--work-home: 18.7 GB, 18668847104 bytes, 36462592 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 5368 MB, 5368709120 bytes, 10485760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes






[root@k3s-work ~]# pvs
  PV         VG              Fmt  Attr PSize   PFree
  /dev/sda2  centos_k3s-work lvm2 a--  <59.00g    0 

[root@k3s-work ~]# vgs
  VG              #PV #LV #SN Attr   VSize   VFree
  centos_k3s-work   1   3   0 wz--n- <59.00g    0 

[root@k3s-work ~]# lvs
  LV   VG              Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home centos_k3s-work -wi-ao---- <17.39g                                                    
  root centos_k3s-work -wi-ao---- <35.61g                                                    
  swap centos_k3s-work -wi-ao----   6.00g   
```

### 1.2. 格式化磁盘

```bash
# 磁盘进行格式化 | 或者 分区进行格式化
[root@k3s-work ~]# mkfs.xfs /dev/sdb
meta-data=/dev/sdb               isize=512    agcount=4, agsize=327680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=1310720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@k3s-work ~]# 


# 整个磁盘创建成一个物理卷 | 或者分区创建成一个物理卷(分区需要是 8e lvm)
[root@k3s-work ~]# pvcreate /dev/sdb
WARNING: xfs signature detected on /dev/sdb at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sdb.
  Physical volume "/dev/sdb" successfully created.

[root@k3s-work ~]# pvscan 
  PV /dev/sda2   VG centos_k3s-work   lvm2 [<59.00 GiB / 0    free]
  PV /dev/sdb                         lvm2 [5.00 GiB]
  Total: 2 [<64.00 GiB] / in use: 1 [<59.00 GiB] / in no VG: 1 [5.00 GiB]


# 查看卷组
[root@k3s-work ~]# vgscan
  Reading volume groups from cache.
  Found volume group "centos_k3s-work" using metadata type lvm2
```

### 1.3. 扩容卷组

```bash
# vgextend 卷组名 新物理卷
[root@k3s-work ~]# vgextend centos_k3s-work /dev/sdb
  Volume group "centos_k3s-work" successfully extended

[root@k3s-work ~]# vgdisplay 
  --- Volume group ---
  VG Name               centos_k3s-work
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               63.99 GiB   #（可以看到卷组容量变大5G）
  PE Size               4.00 MiB
  Total PE              16382
  Alloc PE / Size       15103 / <59.00 GiB
  Free  PE / Size       1279 / <5.00 GiB   # 显示vg的空余空间
  VG UUID               ae8Fdu-fqFt-mSO7-V8ZS-sKzz-GdtI-P6KC32
   
[root@k3s-work ~]# pvs
  PV         VG              Fmt  Attr PSize   PFree 
  /dev/sda2  centos_k3s-work lvm2 a--  <59.00g     0 
  /dev/sdb   centos_k3s-work lvm2 a--   <5.00g <5.00g
```

### 1.4. 扩容卷组下某个LV

```bash
[root@k3s-work ~]# lvscan
  ACTIVE            '/dev/centos_k3s-work/swap' [6.00 GiB] inherit
  ACTIVE            '/dev/centos_k3s-work/home' [<17.39 GiB] inherit
  ACTIVE            '/dev/centos_k3s-work/root' [<35.61 GiB] inherit


# 加 1G 的空间， 如果使用 VG 全部剩余空间，使用 lvresize -r -L +100%FREE /dev/centos_k3s-work/home 
# 注意：-r 或--resizefs参数表示自动调用在线扩容程序，ext调用 resize2fs，xfs调用xfs_growfs，这里忘记加 -r 了
[root@k3s-work ~]# lvresize -L +1G /dev/centos_k3s-work/home 
  Size of logical volume centos_k3s-work/home changed from <17.39 GiB (4451 extents) to <18.39 GiB (4707 extents).
  Logical volume centos_k3s-work/home successfully resized.


[root@k3s-work ~]# xfs_growfs /dev/centos_k3s-work/home 
meta-data=/dev/mapper/centos_k3s--work-home isize=512    agcount=4, agsize=1139456 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4557824, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4557824 to 4819968
```

```bash
[root@k3s-work ~]# df -hT  
Filesystem                        Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos_k3s--work-root xfs        36G  1.7G   34G   5% /
devtmpfs                          devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                             tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                             tmpfs     3.9G  9.0M  3.9G   1% /run
tmpfs                             tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                         xfs      1014M  142M  873M  14% /boot
/dev/mapper/centos_k3s--work-home xfs        19G   33M   19G   1% /home          # 已经加上了1G空间
tmpfs                             tmpfs     783M     0  783M   0% /run/user/0
```

***

### (加餐)扩容固定空间

```bash
lvresize -r -L +1G /dev/centos_k3s-work/home
```

### (加餐)扩容全部空间

```bash
lvextend -r -l +100%FREE  /dev/centos_k3s-work/home
```

### (加餐)在线扩容文件系统

（扩容 LV 时，如果带-r或--resizefs且文件系统支持fsadm，则该步可跳过)

```bash
# ext文件系统： 
resize2fs /dev/mapper/centos-root

# xfs文件系统： 
xfs_growfs /dev/mapper/centos-root
```

## 2. 磁盘缩容

### 2.1 xfs 缩容

* xfs不支持缩容只能扩容
* xfs要缩小容量，只能先删除然后再建立lv

#### 2.1.1 备份

```bash
tar zcvf  home.tar.gz  /home
或
xfsdump -f home.dump /home
```

#### 2.1.2 卸载

* 卸载并查看是否有应用仍在使用对应的目录分区

```bash
[root@k3s-work ~]# umount /home 
[root@k3s-work ~]# df -Th
Filesystem                        Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos_k3s--work-root xfs        36G  1.7G   34G   5% /
devtmpfs                          devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                             tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                             tmpfs     3.9G  9.0M  3.9G   1% /run
tmpfs                             tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                         xfs      1014M  142M  873M  14% /boot
tmpfs                             tmpfs     783M     0  783M   0% /run/user/0
```

#### 2.1.3 删除对应lv

```bash
[root@k3s-work ~]# lvremove /dev/centos_k3s-work/home
Do you really want to remove active logical volume centos_k3s-work/home? [y/n]: y
  Logical volume "home" successfully removed
```

#### 2.1.4 建立新的lv

```bash
[root@k3s-work ~]# lvcreate -L 18GB -n home centos_k3s-work
WARNING: xfs signature detected on /dev/centos_k3s-work/home at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/centos_k3s-work/home.
  Logical volume "home" created.
```

#### 2.1.5 格式化

```bash
[root@k3s-work ~]# mkfs.xfs /dev/mapper/centos_k3s--work-home 
meta-data=/dev/mapper/centos_k3s--work-home isize=512    agcount=4, agsize=1179648 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=4718592, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

#### 2.1.6 挂载

```bash
[root@k3s-work ~]# mount /dev/mapper/centos_k3s--work-home /home       


[root@k3s-work ~]# df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/centos_k3s--work-root   36G  1.7G   34G   5% /
devtmpfs                           3.9G     0  3.9G   0% /dev
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              3.9G  9.0M  3.9G   1% /run
tmpfs                              3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                         1014M  142M  873M  14% /boot
tmpfs                              783M     0  783M   0% /run/user/0
/dev/mapper/centos_k3s--work-home   18G   33M   18G   1% /home

[root@k3s-work home]# more /etc/fstab 
/dev/mapper/centos_k3s--work-home /home                   xfs     defaults        0 0
```

#### 2.1.7 还原

```bash
tar zxvf home.tar.gz
```

### 2.2 ext 缩容

```bash
lvresize -r -L 20G /dev/RHEL/Data
```

## 3. LVM 概念及使用

一个是PV：就是物理空间的意思，比如/dev/sdb1 也可以是一个盘/dev/sdb。只有将一个物理空间添加到一个VG(可以理解这个是多个PV组成的Pool)。 一个是VG：就是一个Pool，有多个PV组成，可以动态向VG中添加PV，使整个VG空间增大，也可以缩小这个VG。 一个是LV：就是linux用来建立一个文件系统的空间，这个空间来源于VG，大小随意，可以扩展。比如/dev/mapper/rhel-root这个目录其实是一个文件系统挂载点，这个点就是承载在一个LV上，这个文件系统的大小就是这个LV的大小。

* 磁盘分区使用fdisk 如 fdisk /dev/sdx

### 一、 创建lvm 的流程

1、fdisk 磁盘分区或者整个磁盘 2、创建物理卷 如 pvcreate /dev/sda5 /dev/sda6 3、创建卷组 vgcreate vg\_linux /dev/sda5 /dev/sda6 4、创建逻辑卷 lvcreate -n centos6 -L 50G vg\_linux 5、格式化逻辑卷 mkfs.xfs /dev/mapper/vg\_linux-centos6

### 二、删除逻辑卷命令

1、卸载相关逻辑卷 umount /lvm1 (逻辑卷名称) 2、删除逻辑卷 如 lvremove /dev/mapper/vg\_linux-centos6 3、删除卷组 如 vgremove vg\_linux 4、删除物理卷 如 pvremove /dev/sdb /dev/sdc

### 三 、其他相关命令整理如下

1. pvcreate:将物理分区建立成独立的pv;
2. pvscan:查找目前系统里面任何具有pv的磁盘；
3. pvdisplay:显示出目前系统上面的pv状态；
4. pvremove：将pv属性删除，让该分区不具有pv属性。

卷组(Volume Group,VG)

所谓的LVM就是将许多的pv整合成了这个VG，所以VG就是LVM组合起来的大磁盘。

相关命令如下：

```bash
1) vgcreate：主要建立VG的命令，主要参数如下：
   -l：卷组上允许创建的最大逻辑卷数；
   -p：卷组中允许添加的最大物理卷数；
   -s：卷组上的物理卷的PE大小。

2) vgscan:查找系统上面是否有VG存在；
3) vgdisplay:显示系统上面的VG状态；
4) vgextend：在VG内增加额外的PV；
5) vgreduce:在VG内删除PV;
6) vgchange：设置VG是否启动；
7) vgremove:删除一个vg。

8) lvcreat:建立LV；
9) lvscan：查询系统上面的LV;
10) lvdisplay:显示系统上面的LV状态；
11) lvextend:在LV里面增加容量；
12) lvreduce:在LV里面减少容量；
13) lvremove:删除一个LV；
14) lvresize:对LV进行容量大小的调整。
```

5，查看的相关命令如下：

```bash
pes、pedisplay 查看pe的大小(pes==pescan)
pvs、pvdisplay 查看物理卷
vgs、vgdisplay、 查看卷组
lvs、lvdisplay、 查看逻辑卷
```

6，创建的相关命令如下：

```bash
pvcreate 设备路径 创建物理卷
vgcreate 名字 pv路径 创建卷组
lvcreate -n 名字 -L 大小 vg名 创建逻辑卷
格式化：mkfs.ext4 lv完整路径               格式化逻辑卷(mkfs.文件系统格式或-t 文件系统格式)
挂载：mount  lv完整路径  挂载点          挂载使用(可以使用/etc/fstab或autofs)
```

7，逻辑卷删除

```bash
1)卸载：umount
2)删lv：lvremove lv完整路径
3)删vg：vgremove vg名称
4)删PV：pvremove 设备完整路径 去硬盘
```

8，逻辑卷扩展：

```bash
1)扩展pv：相当于创建pv
2)扩展vg： vgextend vg名 新增pv路径
3)扩展lv： lvextend -L +扩展量 lv完整名
4)刷新文件系统：resize2fs lv完整路径 注意：灵活运用，看实际情况，注意顺序 (支持在线操作)
```

9，逻辑卷的缩小

```bash
1)首先进行卸载 umount 检查文件系统：e2fsck -f lv完整路径
2)减少文件系统：resize2fs lv完整路径 减少到的大小
3)减少lv卷大小：lvreduce -L -减少量的大小 lv的完整路径
4)挂载使用

减小需谨慎，文件系统的减小后大小一定要和lv卷最终大小相等，并且只有EXT文件系统支持缩小，XFS文件系
```
