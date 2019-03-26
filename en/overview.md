## Storage > Block Storage > Overview 

Block Storage is a virtual disk which can be attached further to a default instance disk. 

- Deleting an attached instance does not affect block storage. 
- Block storage cannot be attached to many instances at the same time. 
- Detached block storage can be newly attached to another instance. 
- Block storage can be attached only to those instances that exist within a same availability zone. 

Block storage is useful in many situations: 

- When a default disk is short of space, attach block storage to expand space. 
- To permanently save data on a default disk before deleting instance, install block storage and copy data. 

Must follow the procedure as below, to use block storage: 

1. [Create block storage](/Storage/Block%20Storage/en/console-guide/#_1).  
2. [Attach it to the instance ](/Storage/Block%20Storage/en/console-guide/#_4).
3. [Divide partitions, format, and mount ](#_1) to use block storage.

Block storage can be attached while an instance is running. Since block storage is attached as empty disk, partitioning, formatting, and mounting may be required depending on the instance operating system. 


## Use Empty Block Storage 
### Linux
Access the instance and follow the procedure: 

> [Note] All commands in the following example must be executed by the `root` authority.

#### Create Partitions 
When block storage is attached to an instance, it is registered as an empty disk device. Check the list of registered disks with `lsblk` , a command of Linux. 
```
# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  20G  0 disk
├─vda1 253:1    0   2G  0 part [SWAP]
└─vda2 253:2    0  18G  0 part /
vdb    253:16   0  10G  0 disk
```
The above example shows `vda`, which is the default disk, is attached to `vdb`, the additional disk. The output of `lsblk` describes partitions have been created for `vda` , but not for `vdb`.

> [Note] Disk devices are named in the alphabetical order of block storage attachment, such as `vda`, `vdb`, and`vdc`. For instance, `vdb` is the secondly attached disk. Check the name of device on the page of storage list in the console.   

First, create a partition at `vdb`, which is an empty disk device. Make the whole disk as one partition, by using `fdisk`, like below. Or, a disk may be divided into many partitions, as required. 
```
# fdisk /dev/vdb
Command (m for help): n

Partition number (1-4): 1
First cylinder (1-20805, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-20805, default 20805): 20805
Command (m for help): w
```
Enter `fdisk /dev/{device name}`in the shell and a prompt will show to enter commands for partition management of a device. To view the list of available commands in the prompt, enter `m`. In this example, enter `n` which means 'New Partition', instead, as a new partition is to be created. Then, you'll be required to enter the type of partition to create. Here, enter `p`  which refers to 'Primary'. For more details on partitioning, refer to [Master Boot Records](https://en.wikipedia.org/wiki/Master_boot_record).
```
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
```
Once the type of partition is decided, it is required to enter the number of partitions to make. As one is to be created in the example, enter `1`.
```
Partition number (1-4, default 1): 1
```
Now it's time to determine the size. The range varies depending on the size of created block storage. As this example regards to creating a partition which uses the whole disk, the default value shall be applied.  
```
First sector (2048-20971519, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):
Using default value 20971519
Partition 1 of type Linux and of size 10 GiB is set
```
We're done with basic setting for partitioning. Enter `w` to apply such setting to the disk.   
```
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
This ends the process on how to create partitions.  

#### Format Partitions 
To use partitions that are created, formatting is required. Type `lsblk` to search for partitions to format. 
```
[root@host-192-168-0-67 ~]# lsblk /dev/vdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  10G  0 disk
└─vdb1 253:17   0  10G  0 part /mnt/vdb
```
The example shows how a partition called `vdb1` has been created under the `vdb` disk. Generally in Linux, a partition is named, 'device name + number'.   

Now, let's format the `vdb1` partition. In Linux,  `mkfs`  is used as the command word. To format partitions, it is required  to specify a corresponding file system. In this example, let's use `xfs`, which is one of the widely used Linux file systems. Refer to [File Systems](https://en.wikipedia.org/wiki/File_system) for available Linux file systems. 
```
# mkfs -t xfs /dev/vdb1
```

#### Mount Disks 
Even after file system is created, mounting is required to access disk. A simple command with `mount` is available, but it is unmounted when an instance is rebooted. This example describes how to automatically mount in the process of instance booting, by adding a disk to mount to the `/etc/fstab` file. 

Below shows the output of the `/etc/fstab` file.  
```
# /etc/fstab
# Created by anaconda on Tue Nov 17 18:37:50 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=3d9cc015-610e-4482-9071-fbf998d68121 /                       xfs     defaults,nodev,noatime        1 1
```
You can find a disk already registered, and it is the root disk of an instance. 

Registering a disk requires an original ID of a disk device, which can be found with `blkid`.  
```
# blkid /dev/vdb1
/dev/vdb1: UUID="5a4004f4-3ba6-4484-9459-7c2b321b727f" TYPE="xfs"
```
`UUID` in the above output refers to the original device ID. 

It is time to create a directory for mounting, and there's no specific requirement. Let's choose `/mnt/vdb` for this example. 
```
mkdir -p /mnt/vdb
```
When the directory is ready, register disks as follows: 
```
# echo "UUID=5a4004f4-3ba6-4484-9459-7c2b321b727f xfs defaults,nodev,noatime 1 2" >> /etc/fstab
```
Lastly, `/etc/fstab` must be reflected: command with `mount -a` to mount all disks registered in `/etc/fstab`. 
```
# mount -a
```
Type `df` to check all are properly mounted. 
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
You can see new partitions are mounted. 

For more details on each command word, command with `man`. 

> [Note] The script below describes how to process the procedure in the above all at once: it has been tested on CentOS.   

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
Adding volumes in Windows is available in two ways, by and large. One is to use GUI-based **Server Manager**, and the other is to use CLI-based **PowerShell**. Each method is briefly described as below: 

#### Add Volumes via Server Manager 
Block storage attached to an Window instance is displayed as an offline disk. To use the disk, the status must be turned to online with volumes created. Here is how to convert the status to online:   

1. Access the instance and click **Server Manager** on the start screen. 
2. Go to **Server Manager> Dashboard** and select **File and Storage Service**. 
3. On its server page, select **Volume>Disk**. 
4. Right-click the offline new disk and select **Convert to Online**. 
5. Click **Yes** to the message asking whether to convert the status to online. 
6. Refresh the list of disks to see if the new disk has changed its status to online. 

Once a disk is converted to online, new volumes can be created. Following describes how to make a new volume: 

1. Right-click the new disk on the list and select **New Volume**. 
2. Select a disk to create a volume for, in the **New Volume Wizard **dialogue box.  
3. Specify the size of a volume to create. 
4. Select a drive letter. 
5. Select a file system for the volume. 
6. Lastly, check the setting and select **Create**. 

Configure the volume available at **Disk Management**. 

1. Right-click the **Start** button and choose **Disk Management**. 
2. Right-click the disk with volume created on the list of disks at **Disk Management** and select **Change Drive Letter and Paths**. 
3. Add a drive letter and paths to a new volume. 

 You can find disks added via Windows browser. For more details on disk management, refer to [Initialize new disks | Microsoft Docs](https://docs.microsoft.com/en-us/windows-server/storage/disk-management/initialize-new-disks).

> [Note] Windows provides two types of disks: 
> * Master Boot Record (MBR): Original disk type, which is applied to less than 2TB-disks. 
> * GUID Partition Table (GPT): A new type of disks which are larger than 2TB.  
> The default disk type configured at **Server Manager** is GPT. 

#### Add Volumes via PowerShell
Adding volumes is also available by using Windows PowerShell. Click **Start** and **Windows PowerShell**. 

Type `Get-Disk` to show the list of disks currently attached to an instance. `RAW` refers to a disk which is to be newly added. 
```
PS C:\Users\Administrator> Get-Disk
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
0      Red Hat VirtIO SCSI Disk Device          Online                                    50 GB MBR
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

Initialize a disk with the `Initialize-Disk` command. Each option is described as follows: 
- Number: Specify the disk number to initialize. 
- PartitionStyle: Specify the type of disk. In this example, GPT is selected, like in **Server Manager**. 

```
PS C:\Users\Administrator> Initialize-Disk -Number 1 -PartitionStyle GPT
Number Friendly Name                            OperationalStatus                    Total Size Partition Style
------ -------------                            -----------------                    ---------- ---------------
1      Red Hat VirtIO SCSI Disk Device          Offline                                   10 GB RAW
```

Create partitions, with `New-Partition`. Each option is described as follows: 
- DiskNumber: Select the disk number to create partitions for. 
- AssignDriveLetter: Set for automatic assignment of drive letters for newly-created partitions.
- UseMaximumSize: Select the entire available disk volumes with the partition size. 

```
PS C:\Users\Administrator> New-Partition -DiskNumber 1 -AssignDriveLetter -UseMaximumSize
   Disk Number: 1
PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
2                D           34603008                                   9.97 GB Basic
```

Type`Format-Volume` to format partitions. Each option is described as follows: 
* -FileSystem: Specify a file system format to be applied. NTFS is used in this example. 
* -Confirm: Specify whether to show prompt to confirm users. It is set False so as not to ask. 

```
PS C:\Users\Administrator> Format-Volume -DriveLetter D -FileSystem NTFS -Confirm:$false

DriveLetter       FileSystemLabel  FileSystem       DriveType        HealthStatus        SizeRemaining             Size
-----------       ---------------  ----------       ---------        ------------        -------------             ----
D                                  NTFS             Fixed            Healthy                   9.92 GB          9.97 GB
```

 You can find disks added via Windows browser. For more details on PowerShell, refer to [PowerShell Module Browser | Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/?view=win10-ps).

> [Note] Take reference of the script as below to process the procedure in the above all at once.  
```
PS C:\Users\Administrator> Get-Disk |
   Where PartitionStyle -eq 'RAW' |
   Initialize-Disk -PartitionStyle GPT -PassThru |
   New-Partition -AssignDriveLetter -UseMaximumSize |
   Format-Volume -FileSystem NTFS -Confirm:$false
```

## Block Storage Snapshots 

By using block storage snapshots, block storage backup can be faster than copying block storage data. It is possible to create block storage snapshots even while the storage is attached to an instance: however, to ensure data integrity and stability, it is recommended to detach at an instance first, and create block storage snapshots.  For enhanced stability, block storage must be unmounted before it is detached from instance. 

Block storage snapshots are read-only, and cannot be directly attached to an instance. Therefore, to use snapshots, create block storage from snapshots and attach it to an instance.  

> [Caution]
Cannot delete block storage if it has block storage snapshots: to delete block storage, delete all snapshots of it. 

### Charges 

Block storage shall be charged from the moment it is created, as much as it is used and by the size of block storage created in the end using empty storage or snapshots.  
