## Storage > Block Storage > Console Guide 

## Create Block Storage

Create block storage to be attached to an instance. The size ranges from 10GB, up to 1,000GB. 

Block storage can be created with empty storage containing no data, or by snapshots of existing block storage.  

To create an empty block storage, select **Empty block storage, with no source** for **Block Storage Source**. Empty block storage must be attached to instance, with partitions divided, and formatted, before use.  Refer to [Block Storage Overview > Use Empty Block Storage](/Storage/Block%20Storage/en/overview/#_1) on how to use block storage. The availability zone where empty block storage is to be located must have an instance to which block storage is to be attached. For volume type, choose either **HDD** or **SSD**, based on the required I/O performance.

You can also create block storage from snapshots. In this case, the size of block storage must be same or larger than that of a snapshot. To set a larger size, customer must adjust partitions of existing block storage or add more partitions so as to make use of increased space. 

To create block storage with snapshots, the availability zone of block storage shall be fixed in the zone where the snapshot is saved:  cannot create block storage in any different availability zone.   

## Delete Block Storage 

Check the following before block storage is deleted: 

* Cannot delete block storage attached to an instance: detach first. 
* Cannot delete block storage which has snapshots: delete all snapshots of block storage. 

Block storage, once deleted, cannot be restored. 

## Manage Attachment 

### Attach Block Storage

Attach block storage to an instance: it is available even when an instance is running. Block storage can be attached to those instances that belong to the same availability zone only. Make sure the block storage is created within a same availability zone where the instance to be attached with belongs to. 

To use empty block storage, its partition must be divided at the instance and formatted: the formatted block storage needs to be mounted. For block storage created by using snapshots, customer must mount it within instance before use. 

> [Note]
> Depending on the operating system, mounting may be automatically applied, requiring no further mounting process. 

### Detach Block Storage

Detach unnecessary block storage from an instance. Note, however, that default disk cannot be detached.

Detaching block storage is available even when an instance is running. To detach without damaging block storage data, block storage must be unmounted within the instance. 

**Linux Instance**

	# umount <Mount-points>

**Windows Instance**

Create an **offline** disk in **Manage Disks** and detach.  

## Create Snapshots 

Create a read-only copy of block storage. It is possible to create block storage snapshots even while the storage is attached to an instance: however, to ensure data integrity and stability, it is recommended to detach from instance first, and create block storage snapshots.  
