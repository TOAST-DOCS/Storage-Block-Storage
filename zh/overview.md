## Storage > Block Storage > 概要

块存储是独立的虚拟磁盘，除实例（instance）的系统盘之外，可以挂载到实例上使用的数据盘。

- 即使删除已挂载的实例，块存储也不会被删除。 
- 块存储无法在多个实例中同时挂载使用。
- 已卸载的块存储，可挂载到其他实例上使用。 
- 块存储只能连挂载到位于相同可用区内的实例。 

块存储可以在多种情况下使用。

- 系统盘的可用空间不足时，可以添加块存储设备，来扩展实例的存储空间。
- 为了永久保存在实例上系统盘的数据，可添加块存储设备，并复制数据。

要使用块存储，请参考以下操作。

1. [创建块存储](/Storage/Block%20Storage/zh/console-guide/#_1)。 
2. 将所创建的块存储[挂载到对象实例(instance)上](/Storage/Block%20Storage/zh/console-guide/#_4)。
3. 块存储 [分区、格式化、挂载（mount）](#_1)之后进行使用。

块存储，亦可在实例运行中进行挂载。挂载的块存储因为是空盘，所以在使用之前，应按照实例操作系统，进行分区、格式化、挂载操作。


## 使用空块存储
### Linux
挂载到实例之后，请参考以下操作。

> [参考] 此示例中所使用的所有命令，必须以`root`权限执行。

#### 创建分区
当块存储挂载到实例时，为未分区状态的空盘。已挂载的磁盘列表可通过Linux操作系统的`lsblk`命令进行确认。
```
# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   2G  0 part [SWAP]
└─vda2 253:2    0  18G  0 part /
vdb    253:16   0  10G  0 disk
```
上述示例显示，系统盘 `vda`与数据盘 `vdb`。通过`lsblk`的输出结果，可查看到： `vda`已做了分区，`vdb`是未分区的状态。

> [参考] 磁盘设备名称，将按照在实例上挂载块存储的顺序，以字母的升序方式排列， `vda`, `vdb`, `vdc`...。上述示例中，`vdb` 是指第二个挂载的磁盘。设备的名称可在控制台的存储列表页面中进行确认。

如下述内容所示，首先在空盘`vdb`上创建分区，利用`fdisk`命令，将磁盘做成一个分区。根据实际需要，也可以将一个磁盘划分多个分区。
```
# fdisk /dev/vdb
Command (m for help): n

Partition number (1-4): 1
First cylinder (1-20805, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-20805, default 20805): 20805
Command (m for help): w
```
在shell输入`fdisk /dev/{磁盘名称}`，系统会进入`Command`模式，提示您请输入相关的命令。在该模式中可输入`m`，来查看可以使用的命令列表。在此示例中，我们将创建一个新的分区，因此输入`n`表示'New Partition'。然后，系统将询问您要创建的分区类型。如下所示，输入`p`表示'Primary'。关于分区的更为详细的信息，请参考 [主引导记录](https://zh.wikipedia.org/wiki/%E4%B8%BB%E5%BC%95%E5%AF%BC%E8%AE%B0%E5%BD%95)。
```
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
```
选定分区类型之后，将再次要求选择要创建的分区个数。在此示例中，将创建1个分区，因此输入`1`。
```
Partition number (1-4, default 1): 1
```
现在可以选择分区大小。可输入的范围取决于所创建的块存储大小。在此示例中，将创建1个使用全部磁盘的分区，因此将使用默认值。
```
First sector (2048-20971519, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):
Using default value 20971519
Partition 1 of type Linux and of size 10 GiB is set
```
分区的操作已完成。输入`w`，保存并反映当前的设置。
```
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
以上，分区创建完成。

#### 分区格式化
若想使用已创建的分区，应进行格式化操作。使用`lsblk`命令来查看要进行格式化的分区。
```
[root@host-192-168-0-67 ~]# lsblk /dev/vdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part /mnt/vdb
```
在上述示例中，可确认：在`vdb`磁盘上已创建了名为`vdb1`的分区。一般情况下，在 Linux上，分区的名称为 '磁盘名称+数字'的形态。

现在要对 `vdb1` 分区进行格式化。在 Linux系统上使用`mkfs`命令。格式化分区时，必须指定要使用的文件系统。在此示例中，将使用Linux常用的文件系统之一 `xfs`。对于可在Linux中可用的文件系统，请参考 [文件系统](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)。
```
# mkfs -t xfs /dev/vdb1
```

#### 磁盘挂载（mount）
必须要先在操作系统中挂载已创建文件系统的磁盘，才能进行访问。您可以使用`mount`命令来挂载磁盘，但如果实例被重启，则磁盘将会被卸载(unmount)。此示例中，将说明如何在`/etc/fstab`文件添加要挂载的磁盘，并且在实例开机（重启）过程中，自动的进行磁盘挂载的方法。

下面是`/etc/fstab`文件中的内容。
```
# /etc/fstab
# Created by anaconda on Tue Nov 17 18:37:50 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=3d9cc015-610e-4482-9071-fbf998d68121 /                       xfs     defaults,nodev,noatime        1 1
```
可确认到已有了一个磁盘的信息。已写入的磁盘信息为实例的系统盘 (root disk)。

要想自动挂载磁盘，则需要磁盘设备的ID。可以使用`blkid`命令来查看，如下内容所示。
```
# blkid /dev/vdb1
/dev/vdb1: UUID="5a4004f4-3ba6-4484-9459-7c2b321b727f" TYPE="xfs"
```
输出内容中，`UUID`的值，即为磁盘设备的ID。

创建挂载的目录文件夹。文件夹无特殊要求。在此示例中，将选择`/mnt/vdb`目录。
```
mkdir -p /mnt/vdb
```
如果安目录文件夹已准备好，可以开始挂载磁盘，如下。
```
# echo "UUID=5a4004f4-3ba6-4484-9459-7c2b321b727f xfs defaults,nodev,noatime 1 2" >> /etc/fstab
```
想要是`/etc/fstab`中新追加的内容即时生效，请使用`mount -a` 命令，新的磁盘将会被挂载。
```
# mount -a
```
是否已挂载成功，可以使用`df`命令，进行确认。
```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        18G  1.3G   17G   7% /
devtmpfs        912M  4.0K  912M   1% /dev
tmpfs           921M     0  921M   0% /dev/shm
tmpfs           921M   89M  832M  10% /run
tmpfs           921M     0  921M   0% /sys/fs/cgroup
/dev/vdb1        10G   33M   10G   1% /mnt/vdb
```
可以确认到新的分区已被挂载。

对各命令的详细说明，可通过 Linux的 `man` 命令进行确认。

> [参考] 要想将上述过程一次性地进行处理，则请参考以下脚本。
> 以下脚本在Cent OS中已进行过测试。

```
#!/bin/bash

DEVICES=(`lsblk -s -d -o name,type | grep disk | awk '{print $1}'`)

for DEVICE_NAME in ${DEVICES[@]}
do
    MOUNT_DIR=/mnt/$DEVICE_NAME
    FS_TYPE=xfs

   mkdir -p $MOUNT_DIR

    echo -e "n\np\n1\n\n\nw" | fdisk /dev/$DEVICE_NAME
    PART_NAME="/dev/${DEVICE_NAME}1"
    mkfs -t $FS_TYPE -f $PART_NAME > /dev/null

    UUID=`blkid $PART_NAME -o export | grep UUID | cut -d'=' -f 2`
    echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime 1 2" >> /etc/fstab

    mount -a
done
```


### Windows
在Windows中添加卷，主要有两种方法。第一种，使用基于GUI**服务器管理器**；第二种，使用基于CLI**PowerShell**。本文件中将对这两种方法进行简要的介绍。

#### 使用**服务器管理器**，添加卷
在控制台中，已挂载到实例上的块存储，在Windows系统中初始状态为脱机。要想使用该磁盘，则必须联机该磁盘，并创建卷。操作过程如下：

1. 登录实例后，在开始页面处点击**服务器管理器**。
2. **服务器管理器 > 仪表板** 页面中，选择 **文件和存储服务**。
3. 在文件及保存位置服务的服务器页面中，选择 **卷 > 磁盘**。
4. 将脱机状态的新磁盘，鼠标右键点击之后，选择 **联机**。
5. 如果系统提示您联机时，则点击 **是**。
6. 是在空白处单击鼠标右键，点击刷新，确认磁盘已为**联机**状态。

磁盘联机后，您可以创建新卷。操作如下：

1. 在磁盘列表中，新磁盘上单击鼠标右键，选择 **新建卷**。
2. 在**新建卷向导** 对话框中，选择要创建卷的磁盘。
3. 指定要创建卷的大小。
4. 选择驱动器号。
5. 选择卷的文件系统。
6. 确认以上的设置项，再选择**创建**。

也可以在**磁盘管理**中，创建卷。

1. 在开始页面处点击鼠标右键，再选择**磁盘管理**。
2. 在**磁盘管理**的磁盘列表中，新磁盘上单击鼠标右键，再选择**新建简单卷**进行创建。
3. 指定要创建卷的大小，选择驱动器号等操作。

您现在可以在Windows资源管理器中查看到新添加的磁盘。关于磁盘管理的更为详细的说明，请参考 [新磁盘初始化 | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows-server/storage/disk-management/initialize-new-disks)。

> [参考] Windows的磁盘分区形式有两种。
> * MBR: 目前仍是最常用的分区形式，但是，MBR只支持不大于2TB的磁盘。
> * GPT: 是一种新的分区形式，GPT支持2TB以上的磁盘。
>默认情况下，在**服务器管理器**中设置的磁盘，采用GPT分区形式。

#### 使用PowerShel挂载磁盘
使用Windows所提供的PowerShell，可以实现挂载磁盘。点击Windows的**开始**，再点击**Windows PowerShell**。

使用`Get-Disk`命令，查看目前实例上的磁盘列表。在以下结果中，标示为`RAW`的磁盘为要新添加的磁盘。
```
PS C:\Users\Administrator> Get-Disk
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
0      Red Hat VirtIO SCSI Disk Device          Online                                    50 GB MBR
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

使用`Initialize-Disk`命令初始化磁盘。各选项说明如下：
* -Number:选择要初始化的磁盘编号。
* -PartitionStyle:选择分区形式。在此示例中，与在**服务器管理器**中的情况一样，将选择GPT分区形式。 

```
PS C:\Users\Administrator> Initialize-Disk -Number 1 -PartitionStyle GPT
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

通过`New-Partition`命令创建分区。各选项说明如下：
* -DiskNumber:选择要创建分区的磁盘编号。
* -AssignDriveLetter:设置自动为创建的分区分配驱动器号。
* -UseMaximumSize:选择整个磁盘容量作为分区大小。

```
PS C:\Users\Administrator> New-Partition -DiskNumber 1 -AssignDriveLetter -UseMaximumSize
   Disk Number: 1
PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
2                D           34603008                                   9.97 GB Basic
```

使用`Format-Volume`命令格式化分区。各选项说明如下：
* -FileSystem: 选定要使用的文件系统类型。在此示例中将选择NTFS。
* -Confirm: 选定以使用者确认为目的的提示输出与否。在该例题中，为了防止出现提示等选择框，而设定为false。

```
PS C:\Users\Administrator> Format-Volume -DriveLetter D -FileSystem NTFS -Confirm:$false

DriveLetter       FileSystemLabel  FileSystem       DriveType        HealthStatus        SizeRemaining             Size
-----------       ---------------  ----------       ---------        ------------        -------------             ----
D                                  NTFS             Fixed            Healthy                   9.92 GB          9.97 GB
```

您现在可以在Windows资源管理器中查看到新添加的磁盘。关于 PowerShell的更为详细的说明，请参考 [PowerShell Module Browser | Microsoft Docs](https://docs.microsoft.com/zh-cn/powershell/module/?view=win10-ps)。

> [参考] 如果想把上述过程一次性地处理完，请参考以下脚本。
```
PS C:\Users\Administrator> Get-Disk |
   Where PartitionStyle -eq 'RAW' |
   Initialize-Disk -PartitionStyle GPT -PassThru |
   New-Partition -AssignDriveLetter -UseMaximumSize |
   Format-Volume -FileSystem NTFS -Confirm:$false
```

## 块存储快照（snapshot）

块存储快照功能，可以让您更快速的复制块存储上的数据，和备份块存储。在挂载到实例的状态下，也可创建块存储快照，但为了保障数据的可用性与安全性，建议您：①在实例的操作系统中进行卸载操作(unmount)；②在Toast块存储控制台页面中，卸载实例的块存储；③创建块存储快照。

块存储的快照是只读的，不能直接挂载到实例上使用。要想使用快照，请先从快照创建块存储，并将所创建的块存储挂载到实例，之后可以正常使用。

> [注意事项]
无法删除具有快照的块存储。如果要删除块存储，请先删除其块存储的所有快照。

### 费用

块存储，根据从形成的瞬间开始所使用的存储空间征收费用，同时，将根据最终生成为空的块存储或快拍的块存储大小征收费用。
