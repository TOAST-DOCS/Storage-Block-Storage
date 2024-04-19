## Storage > Block Storage > Console Guide

## Create Block Storage

Create block storage to be attached to an instance.

Block storage can be created with empty storage containing no data, or by snapshots of existing block storage.

To create empty block storage, select **Empty block storage, with no source** for **Block Storage Source**. Empty block storage must be attached to instance, with partitions divided, and formatted, before use.  Refer to [Block Storage Overview > Use Empty Block Storage](/Storage/Block%20Storage/en/overview/#use-empty-block-storage) on how to use block storage. The availability zone where empty block storage is to be located must have an instance to which block storage is to be attached. For block storage type, choose either **HDD** or **SSD**, based on the required I/O performance.

You can also create block storage from snapshots. In this case, the size of block storage must be the same as or larger than that of a snapshot. To set a larger size, the customer must manually adjust partitions of existing block storage or add more partitions so as to make use of increased space.

To create block storage with snapshots, the availability zone of block storage will be fixed to the zone where the snapshot is stored. A block storage cannot be created in any different availability zone.

### Encrypted Block Storage

You can create encrypted block storage by selecting **Encrypted HDD** or **Encrypted SSD** from the block storage type. Encrypted block storage is encrypted using a symmetric key managed by NHN Cloud's Secure Key Manager service. Therefore, to create encrypted block storage, you must create a symmetric key in the Secure Key Manager service in advance.

The policies for encrypted block storage are as follows.

* Due to encryption and decryption, I/O performance may be reduced compared to general block storage types (**HDD**, **SSD**).
* You cannot change the symmetric key ID that is registered when creating encrypted block storage. To change the symmetric key, you must use the key rotation feature of Secure Key Manager.
* After selecting encrypted block storage, the **Rotate Key** button allows you to re-encrypt block storage encrypted with an older version of the key with a newer version of the key.

> [Note]
Backup products allow you to prepare for data loss due to key deletion, and create a replica for safe keeping.

<!-- For newline -->

> [Caution]
If the Secure Key Manager service deletes the symmetric key that you set for encrypted block storage and then detaches that block storage from the instance, it can't be decrypted again. You must manage symmetric keys carefully to avoid accidentally deleting them.

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

Detach unnecessary block storage from an instance. Note, however, root block storage cannot be detached.

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

<!-- 개행을 위한 주석이므로 필수로 포함되어야 합니다. -->

> [Caution]
> To proceed with replication, at least 100KB of free space in block storage is required.

### Region

Select a region to replicate to other than the region you are currently using.

### Block Storage Type

Select the type of block storage to use in the region to which to replicate. You can select a type that is different from the block storage type being used in the current region.

### Availability Zone

Select the availability zone to use in the region to which to replicate. You can select an availability zone that is different from the availability zone being used in the current region.

## Troubleshooting Guide

### An issue where an instance boots from unintended block storage

The instance might boot with block storage additionally attached to the instance mounted on `/`. This usually happens when you attach block storage created with the instance's OS image to another instance additionally.

Linux determines which block storage to mount on `/` using `etc/fstab` at boot time. For the OS images used by NHN Cloud, the block storage to mount is determined based on the file system UUID. If block storage with the same file system UUID value is attached, unintended block storage may be mounted on `/`.

```console
# cat /etc/fstab
...
UUID=6cd50e51-cfc6-40b9-9ec5-f32fa2e4ff02 /                       xfs     defaults        0 0
```

You can check the file system UUID of block storage with the `blkid` command.

```console
# blkid
/dev/vda1: UUID="6cd50e51-cfc6-40b9-9ec5-f32fa2e4ff02" TYPE="xfs"
/dev/vdb1: UUID="6cd50e51-cfc6-40b9-9ec5-f32fa2e4ff02" TYPE="xfs"
```

As shown above, if the file system UUID of the additionally attached block storage is the same, the additionally attached block storage might be mounted on `/` depending on how the Linux distribution works.

Use the following steps to solve the problem by making the file system UUIDs of the two block storage different.

1. After stopping the instance, [detach the block storage](console-guide/#detach-block-storage) that is causing the problem (that is, the one that was mounted on `/` unexpectedly).

2. Start the instance.

3. When booting is complete, [attach the block storage](console-guide/#attach-block-storage) that is causing the problem.

4. Use the command below to change the file system UUID of the block storage that is causing the problem. Execute the command below according to the type of block storage causing the problem. The type of block storage can be found with the `blkid` command.

	If the file system of the block storage causing the problem is ext4:

	<pre><code class="language-console"># tune2fs -U random /dev/vdb1
	tune2fs 1.42.9 (28-Dec-2013)
	Setting the UUID on this filesystem could take some time.
	Proceed anyway (or wait 5 seconds to proceed) ? (y,N) y
	</code></pre>

	If the file system of the block storage causing the problem is xfs:

	<pre><code class="language-console"># xfs_admin -U generate /dev/vdb1
	Clearing log and setting UUID
	writing all SBs
	new UUID = 0037c590-0545-4736-bcdc-d052681eb5f5
	</code></pre>

5. Verify that the file system UUID has been changed.

	<pre><code class="language-console"># blkid
	/dev/vda1: UUID="6cd50e51-cfc6-40b9-9ec5-f32fa2e4ff02" TYPE="xfs"
	/dev/vdb1: UUID="0037c590-0545-4736-bcdc-d052681eb5f5" TYPE="xfs"
	</code></pre>

### An issue where an instance does not operate because the block storage mount fails

If you set `/etc/fstab` incorrectly when adding block storage, the volume mount may fail during boot and the instance may enter emergency mode.

To prevent this situation, it is recommended to use the `nofail` option according to the [Block Storage Mounting Guide](/Storage/Block%20Storage/en/overview/#_4) when adding additional block storage in `/etc/fstab`.

If you have modified `/etc/fstab` incorrectly and your instance is not booting properly, please contact the Customer Center.