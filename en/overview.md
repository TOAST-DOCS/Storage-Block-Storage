## Storage > Block Storage > Overview

Block Storage is a virtual disk that can be attached in addition to the default disk of the instance.

- Deleting an attached instance does not affect the block storage.
- Block storage cannot be attached to multiple instances at the same time.
- Detached block storage can be newly attached to another instance.
- Block storage can be attached only to the instances that exist within the same availability zone.

Block storage can be used in many situations:

- When the storage space of the default disk is insufficient, you can increase the storage space of the instance by attaching additional block storage.
- To keep the data on the instance's default disk permanently before deleting the instance, you can attach block storage and copy data to the block storage.

To use block storage, you need to do the following:

1. [Create block storage](/Storage/Block%20Storage/en/console-guide/#create-block-storage).
2. [Attach the created block storage to the target instance](/Storage/Block%20Storage/en/console-guide/#attach-block-storage).
3. [Partition, format, and mount the block storage](#use-empty-block-storage) and use it.

Block storage can be attached while the instance is running. Attached block storage is an empty disk, so you will need to partition, format, and mount it manually depending on the operating system of the instance before using it.


## Use Empty Block Storage
### Linux

Connect to the instance and take the following steps:

> [Note] All commands in the following examples must be executed with the `root` privilege.

#### Create a Partition
When block storage is attached to an instance, it is registered as an empty disk device. Check the list of registered disks with the Linux `lsblk` command.
```
# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   2G  0 part [SWAP]
└─vda2 253:2    0  18G  0 part /
vdb    253:16   0  10G  0 disk
```
The example above that the default disk `vda` and the additional disk `vdb` are attached. If you look at the output of `lsblk`, you can see that partitions have been created in `vda`, but `vdb` is empty.

> [Note] Disk devices are named alphabetically in the order in which block storage is attached to the instance, such as `vda`, `vdb`, `vdc`, and so on. In the example above, the `vdb` disk indicates the second attached disk. The device names can be found on the console's Storage List page.

First, create a partition on the empty disk device `vdb`. As shown below, use the `fdisk` utility to create a single partition for the entire disk. A disk can also be partitioned into multiple partitions as needed.
```
# fdisk /dev/vdb
Command (m for help): n

Partition number (1-4): 1
First cylinder (1-20805, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-20805, default 20805): 20805
Command (m for help): w
```
If you type `fdisk /dev/{device name}` in the shell, you will be prompted for the device's partition management commands. Type `m` for a list of commands available at this prompt. In this example, we will create a new partition, so type `n` which means 'New Partition'. Then, you will be asked to enter the type of partition to be created, as shown below. In this example, type `p` which means 'Primary'. For more information about partitions, see [Master Boot Record](https://en.wikipedia.org/wiki/Master_boot_record).
```
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
```
Once the type of partition is decided, you need to enter the number of partitions to create. In this example, we will create one partition, so enter `1`.
```
Partition number (1-4, default 1): 1
```
Now, determine the size of the partition. The available range varies depending on the size of created block storage. In this example, we will create a partition that uses the entire disk, so select the default value.
```
First sector (2048-20971519, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):
Using default value 20971519
Partition 1 of type Linux and of size 10 GiB is set
```
The basic partition setup is now complete. Type `w` to reflect the settings entered so far on the disk.
```
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
This completes the process to create a partition.

#### Format the Partition
To use the created partition, you need to format it. Find the partition to format with the `lsblk` command.
```
[root@host-192-168-0-67 ~]# lsblk /dev/vdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part /mnt/vdb
```
In the example above, you can see that a partition named `vdb1` has been created on the `vdb` disk. In Linux, partition names are usually in the form of 'device name + number'.

Now, format the `vdb1` partition. On Linux, use the `mkfs` command. When formatting a partition, you must specify the file system to use. In this example, we will format the partition using `xfs`, one of the popular Linux file systems. For file systems available on Linux, see [File system](https://en.wikipedia.org/wiki/File_system).
```
# mkfs -t xfs /dev/vdb1
```

#### Mount the Disk
A disk where a file system has been created can only be accessed after the mount process. You can mount the disk with the simple `mount` command, but it will be unmounted when the instance reboots. This example explains how to mount a disk automatically during the instance boot process by adding a disk to be mounted to the `/etc/fstab` file.

The following is the output of the `/etc/fstab` file.
```
# /etc/fstab
# Created by anaconda on Tue Nov 17 18:37:50 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=3d9cc015-610e-4482-9071-fbf998d68121 /                       xfs     defaults,nodev,noatime        1 1
```
You can see that one disk is already registered. The registered disk is the instance's default disk (root disk).

To register a disk, you need the disk's device unique ID. The device unique ID can be checked with the `blkid` command as shown below.
```
# blkid /dev/vdb1
/dev/vdb1: UUID="5a4004f4-3ba6-4484-9459-7c2b321b727f" TYPE="xfs"
```
The item corresponding to `UUID` in the output is the device unique ID.

Create a mount target directory. The mount target directory can be any directory. In this example, we will use `/mnt/vdb`.
```
mkdir -p /mnt/vdb
```
When the mount target directory is ready, register the disk as follows.
```
# echo "UUID=5a4004f4-3ba6-4484-9459-7c2b321b727f /mnt/vdb xfs defaults,nodev,noatime,nofail 1 2" >> /etc/fstab
```

> [Note] In the example above, the `nofail` option has been added so that booting can be performed even if the volume mount fails.

Finally, you need to reflect the contents of `/etc/fstab`. Use the `mount -a` command to mount all disks registered in `/etc/fstab`.
```
# mount -a
```
Use the `df` command to check whether the partition has been mounted normally.
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
You can see that the new partition is mounted.

You can find a detailed description of each command with the Linux `man` command.

> [Note] To handle the process above at once, refer to the following script.
> The following script has been tested on CentOS 7.

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
   mkfs -t $FS_TYPE $PART_NAME > /dev/null

   UUID=`blkid $PART_NAME -o export | grep "^UUID=" | cut -d'=' -f 2`
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

   mount -a
done
```


> [Note] The default file system for CentOS6, Debian, and Ubuntu is ext4.
> So you have to use the script below.

```
#!/bin/bash

DEVICES=(`lsblk -s -d -o name,type | grep disk | awk '{print $1}'`)

for DEVICE_NAME in ${DEVICES[@]}
do
   MOUNT_DIR=/mnt/$DEVICE_NAME
   FS_TYPE=ext4

   mkdir -p $MOUNT_DIR

   echo -e "n\np\n1\n\n\nw" | fdisk /dev/$DEVICE_NAME
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE $PART_NAME > /dev/null

   UUID=`blkid $PART_NAME -o export | grep "^UUID=" | cut -d'=' -f 2`
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

   mount -a
done
```

### Windows

There are two main ways to add a volume in Windows. The first is to use the GUI-based **Server Manager**, and the second is to use the CLI-based **PowerShell**. This article briefly introduces each method.

#### Add a Volume Using **Server Manager**
Block storage attached to a Windows instance appears as an offline disk. To use this disk, you must bring it online and create a volume. The process of bring the disk online is as follows:

1. After connecting to the instance, click **Server Manager** on the start screen.
2. Go to **Server Manager > Dashboard**, and select **File and Storage Services**.
3. On the Servers page of File and Storage Services, select **Volumes > Disks**.
4. Right-click the new disk that is offline and select **Bring Online**.
5. When prompted to bring online, click **Yes**.
6. Refresh the disk list to verify that the new disk is online.

When the disk becomes online, you can create a new volume. The process of creating a new volume is as follows:

1. Right-click the new disk in the disk list and select **New Volume**.
2. In the **New Volume Wizard** dialog box, select the disk on which to create the volume.
3. Specify the size of the volume to be created.
4. Select a drive letter.
5. Select a file system for the volume.
6. Finally, check the set items and select **Create**.

Perform setting to make the created volume available in the **Disk Management** program.

1. Right-click on the **Start** button and select **Disk Management**.
2. In the disk list in **Disk Management**, right-click the disk on which you created the volume and select **Change Drive Letter and Path**.
3. Add the drive letter and path for the new volume.

Now you can see that the disk has been added in Windows Explorer. For more details on disk management, see [Initialize new disks | Microsoft Docs](https://docs.microsoft.com/en-us/windows-server/storage/disk-management/initialize-new-disks).

> [Note] Windows provides two disk formats.
> * Master Boot Record (MBR): This is a disk format that has been used for a long time and is used for disks of 2TB or less.
> * GUID Partition Table (GPT): A new disk format for disks larger than 2TB.
> By default, disks configured in **Server Manager** are in GPT format.

#### Add a Volume Using PowerShell
You can also add volumes with PowerShell provided by Windows. Click **Start**, and then click on **Windows PowerShell**.

Type the `Get-Disk` command to print a list of disks currently attached to the instance. The disk marked `RAW` in the result below is the disk to be newly added.
```
PS C:\Users\Administrator> Get-Disk
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
0      Red Hat VirtIO SCSI Disk Device          Online                                    50 GB MBR
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

Initialize the disk with the `Initialize-Disk` command. A description of each option is as follows.

* Number: Specifies the number of the disk to be initialized.
* PartitionStyle: Specifies the disk format. In this example, we format the disk as GPT as we do in the **Server Manager** example.

```
PS C:\Users\Administrator> Initialize-Disk -Number 1 -PartitionStyle GPT
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

Create a partition with the `New-Partition` command. A description of each option is as follows.

* DiskNumber: Selects the number of the disk to create a partition on.
* AssignDriveLetter: Sets to automatically assign a drive letter to the created partition.
* UseMaximumSize: Selects all available disk capacity as the size of the partition.

```
PS C:\Users\Administrator> New-Partition -DiskNumber 1 -AssignDriveLetter -UseMaximumSize
   Disk Number: 1
PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
2                D           34603008                                   9.97 GB Basic
```

Format the partition with the `Format-Volume` command. A description of each option is as follows.

* FileSystem: Specifies the file system format to use. In this example, specify NTFS.
* Confirm: Specifies whether to output a prompt for user confirmation. In this example, set it to false to prevent the prompt output.

```
PS C:\Users\Administrator> Format-Volume -DriveLetter D -FileSystem NTFS -Confirm:$false

DriveLetter       FileSystemLabel  FileSystem       DriveType        HealthStatus        SizeRemaining             Size
-----------       ---------------  ----------       ---------        ------------        -------------             ----
D                                  NTFS             Fixed            Healthy                   9.92 GB          9.97 GB
```

Now you can see the disk has been added in Windows Explorer. For a more detailed explanation of PowerShell, see [PowerShell Module Browser | Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/?view=win10-ps).

> [Note] To handle the process above at once, refer to the following script. The following script was tested on Windows 2012 and Windows 2016 which support PowerShell 3.0 and above.
```
Get-Disk |
 Where PartitionStyle -eq 'RAW' |
 Initialize-Disk -PartitionStyle GPT -PassThru |
 New-Partition -AssignDriveLetter -UseMaximumSize |
 Format-Volume -FileSystem NTFS -Confirm:$false
```

> [Note] In Windows 2008 that supports up to PowerShell 2.0, you can use the script below to add a disk.
```
$newdisk = gwmi -Query "SELECT * FROM Win32_diskdrive where partitions=0"
$index = $newdisk.index
$Scriptblock=@"
select disk=$index
clean
convert GPT
Create Partition Primary
Format fs=ntfs quick
assign
"@
$Scriptblock | diskpart
```

## Block Storage Snapshot

The block storage snapshot feature allows users to back up block storage faster than directly copying data from the block storage. Although a block storage snapshot can be created while block storage is attached to the instance, it is recommended to detach it from the instance and create a block storage snapshot to ensure data consistency and reliability. To enhance reliability, unmount block storage before detaching it from the instance.

A snapshot of block storage is read-only, so it cannot be used by attaching it directly to the instance. To use a snapshot, create block storage from the snapshot and attach the created block storage to the instance.

> [Caution]
Block storage with block storage snapshots cannot be deleted. To delete block storage, delete all snapshots of the block storage.

### Billing

Block storage is charged according to `tbe size of the block storage` set when it is created. Snapshot is charged according to the size of the original block storage.