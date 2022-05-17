## Storage > Block Storage > Console Guide

## Create Block Storage

Create block storage to be attached to an instance.

Block storage can be created with empty storage containing no data, or by snapshots of existing block storage.

To create empty block storage, select **Empty block storage, with no source** for **Block Storage Source**. Empty block storage must be attached to instance, with partitions divided, and formatted, before use.  Refer to [Block Storage Overview > Use Empty Block Storage](/Storage/Block%20Storage/en/overview/#use-empty-block-storage) on how to use block storage. The availability zone where empty block storage is to be located must have an instance to which block storage is to be attached. For volume type, choose either **HDD** or **SSD**, based on the required I/O performance.

You can also create block storage from snapshots. In this case, the size of block storage must be the same as or larger than that of a snapshot. To set a larger size, the customer must manually adjust partitions of existing block storage or add more partitions so as to make use of increased space.

To create block storage with snapshots, the availability zone of block storage will be fixed to the zone where the snapshot is stored. A block storage cannot be created in any different availability zone.

## Delete Block Storage

Check the following before deleting block storage:

* You cannot delete block storage attached to an instance. Detach it from the instance first.
* You cannot delete block storage which has snapshots. Delete all snapshots of the block storage.

Once deleted, block storage cannot be restored.

## Manage Attachment

### Attach Block Storage

Attach block storage to an instance. You can attach block storage while the instance is running. Block storage can only be attached to an instance in the same availability zone. When creating block storage, make sure that you create the block storage in the same availability zone as the instance to attach to.

If you attach empty block storage, it must be partitioned and formatted in the instance before use. A formatted block storage must be mounted before use. For block storage created with snapshots, you must mount it manually within the instance to use it.

> [Note]
> Depending on the operating system, mounting may be automatically applied, requiring no further mounting process.

### Detach Block Storage

Detach unnecessary block storage from an instance. Note, however, that default disk cannot be detached.

You can detach block storage even while the instance is running. However, you must first unmount the block storage from the instance and detach the block storage in the console. Detaching while the block storage is mounted causes the following issues:

1. Block storage data may be corrupted, resulting in data loss.
2. When you add new block storage, the block storage name on the console and the block storage name on the instance appear different. You will need to restart the instance for the same block storage name to appear on the console and on the instance.

**Linux Instance**

	# umount <mount point>

**Windows Instance**

Make the disk **Offline** in **Disk Management** and then detach it.

## Create Snapshots

Create a read-only copy of the block storage. Although block storage snapshots can be created while the block storage is attached to an instance, it is recommended to detach it from the instance and create block storage snapshots to ensure data consistency and reliability.

## Replicate Block Storage

You can use block storage by replicating it to another region. Although block storage can be replicated while being attached to an instance, it is recommended that you stop the instance or detach the block storage from the instance and proceed with replication to ensure data consistency and reliability.

After requesting replication, you can check the replication status and whether the replication is successful or not in **Replication Status**.

> [Note]
> The replication function is a one-time operation, and changes to the original block storage after the replication are not reflected.

### Region

Select a region to replicate to other than the region you are currently using.

### Block Storage Type

Select the type of block storage to use in the region to which to replicate. You can select a type that is different from the block storage type being used in the current region.

### Availability Zone

Select the availability zone to use in the region to which to replicate. You can select an availability zone that is different from the availability zone being used in the current region.
