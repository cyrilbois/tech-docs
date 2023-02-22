---
title:  共有云 - 杂记
...

##  挂载云盘
挂载数据盘，参考官方文档[挂载云盘](https://help.aliyun.com/document_detail/25446.html)。 

通过上面的链接很容易完成线上的挂载，但是挂载后任然需要以下的Linux操作：
1. 给云盘分区 `fdisk /dev/vdb` 
2. 格式化分区创建文件系统  `mkfs.ext4 /dev/vdb1`; (通过`fdisk -l`查看分区信息)
3. 挂载文件系统 `mount /dev/vdb1 /data`; (需要提前创建好文件夹`/data`)
4. 查看结果`df -h`
5. 写入挂载磁盘信息`echo '/dev/vdb1 /data ext4   defaults  0 0' >> /etc/fstab`  **(如果不写入信息的话， 每次实例重启都需要重新mount)**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  1.6G   17G   9% /
devtmpfs        486M     0  486M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  428K  496M   1% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
tmpfs           100M     0  100M   0% /run/user/0
/dev/vdb1        20G   45M   19G   1% /data
```

> 参考链接：https://help.aliyun.com/document_detail/25426.html  过程中使用命令`fdisk -l`查看磁盘情况


## 在线扩容数据盘
在线扩容很简单，参考 [在线扩容云盘](https://help.aliyun.com/document_detail/113316.html)，在线扩容无需重启服务，离线扩容需要重启服务，通过命令`fdisk -l`即可查看到磁盘信息如下：
```

Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b2d99

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    41943039    20970496   83  Linux

Disk /dev/vdb: 26.8 GB, 26843545600 bytes, 52428800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x2522c6cb

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    41943039    20970496   83  Linux
```
原数据盘`/dev/vdb`是20G的，扩容到了25G，分区`/dev/vdb1`依然是20G，相当于还有5G没有分区。

扩容后续操作参考[Linux格式化数据盘](https://help.aliyun.com/document_detail/25452.html)。 具体分以下几步：(示例是针对'扩展已有MBR分区'的场景)
1. umout 分区 `umount /dev/vdb1` (注意确保分区目前没有进程使用，不能再/data 目录此目录，查看数据盘的挂载路径，根据返回的文件路径卸载分区，直至完全卸载已挂载的分区。`mount | grep "/dev/vdb"`)
2. 使用fdisk工具删除旧分区,  `fdisk /dev/vdb` 选d删除分区
3. 创建分区`fdisk /dev/vdb` （操作同挂载类似, 运行`lsblk /dev/vdb`确保分区表已经增加。)
4. 通知内核更新分区表：`partprobe /dev/vdb`
5. 扩容文件系统；使用 resize2fs 指令扩大文件系统大小`resize2fs /dev/vdb1`
6. 再次挂载 `mount /dev/vdb1 /data`


## 腾讯云 发邮件25端口问题
此端口默认封的，解封需要单独在安全管控这里申请。
https://cloud.tencent.com/document/product/213/40436
