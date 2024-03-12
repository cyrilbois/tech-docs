---
title:  如何在CentOS 7上扩展磁盘空间
...

在UI (如vsphere center)控制面板中扩容磁盘后，还需要在操作系统层面扩容。

#### 以 root 身份打开终端并键入以下命令：
```
echo 1 > /sys/block/sda/device/rescan
```
该命令将值1写入重新扫描文件，指示系统对sda设备执行重新扫描，从而让系统知晓添加的存储设备。 

#### 使用 fdisk 运行以下命令进行磁盘操作, 进入交互提示
```
fdisk /dev/sda
```

按如下输入
```shell
Command (m for help): d
Partition number (1,2, default 2): 2
#上面输入删除2号分区 /dev/sda2


Command (m for help): n
Select (default p): p
Partition number (2-4, default 2): 2
# 选择创建新的主分区 分区号选择2

First sector (XXX-YYY, default ZZZ):
Last sector, +sectors or +size{K,M,G} (XXX-YYY, default ZZZ):
# 两次回车接收默认值

Command (m for help): t
Partition number (1-2): 2
Hex code (type L to list all codes): 8e
# 选择修改分区 2 的ID为8e （之前删除的一般ID就是8e）

Command (m for help): w
# 保存更改
```

#### 后续操作

- 将改变通知内核更改 `partx -u /dev/sda2`
- 更改物理卷大小 `pvresize /dev/sda2`
- 需要获取逻辑卷的名称：`lvdisplay` 查找以 XXXXXXX/root 结尾的卷的名称（在下面的示例中为 /dev/centos/root）
- 扩展逻辑卷： `lvextend -l +100%FREE /dev/XXXXXXX/root`;   这里实际是 `lvextend -l +100%FREE /dev/centos/root`
- 命令不支持的话，采用`xfs_growfs /dev/XXXXXXX/root`
- 运行命令查看 `df -h`;  如果没有加上，尝试使用`resize2fs /dev/mapper/centos-root`