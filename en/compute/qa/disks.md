# Disks, snapshots, and images

#### How much disk space can I use for a VM? {#disk-size}

For disk limitations, see [{#T}](../concepts/limits.md).

#### How do I change the disk size {#disk-resize}

If you need to increase the disk size, see [{#T}](../operations/disk-control/update.md#change-disk-size) within the set [limits](../concepts/limits.md#limits-disks). The data on the disk is kept. Make sure to wait until the operation is complete.

Reducing the disk size is impossible: this is an architectural feature of {{ yandex-cloud }} technologies. It's also impossible to create from the snapshot a disk smaller than the parent disk.

If the partition on the Linux boot disk doesn't expand automatically after increasing the disk size, use the following commands:

```bash
sudo growpart /dev/vda 2
```

```bash
sudo resize2fs /dev/vda2
```

For non-bootable disks, the partition size doesn't automatically go up. Use standard OS tools to expand it (for example, `parted`).

#### How do I upload my image {#load-image}

See: [{#T}](../operations/image-create/upload.md).

For the virtual machine created from your image to function properly, you must follow all the recommendations in the instructions.

If you're confident that you followed all the recommendations, but the image doesn't work, or if you have any additional questions, contact support.

#### Should I use swap {#swap-use}

We strongly recommend not using swap in cloud systems because the disk subsystem can become a "bottleneck" for the entire guest system. [Network disk limits](../concepts/limits.md#compute-limits-disk) are low enough to use the disk as a RAM "extension".

Instead of swap, you can use the `zram-config` utility. It allows you to organize a kind of swap inside the RAM itself thanks to compression, which is ten times faster than IO with a disk. Keep in mind that with a high load on IO and/or vCPUs, the use of `zram-config` may negatively affect the `iowait` and, consequently, the operation of the network, disks, and vCPUs.

The best solution for increasing the available memory is to expand the vRAM on the VM.

#### The snapshot size is larger than there was data on the disk {#snapshot-size-larger}

This happens because occupied sectors remain on the disk after deleting the files.

The solution is to write a file consisting of zeros to the entire unoccupied disk space, then flush the cache to the disk and delete the record about this file.
* For Windows: stop disk operations and use the `SDelete` utility. You can find out how it works and download it in [the Microsoft documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sdelete).
* For Linux: stop disk operations and enter the following commands one by one:

  ```bash
  dd if=/dev/zero | pv > full.disk
  ```

  ```bash
  sync
  ```

  ```bash
  rm full.disk
  ```

The "empty" space on the disk then becomes truly empty and you can create a disk snapshot. Its size is similar to the current occupied disk space.

#### How are snapshot quotas counted {#how-shapshot-qoutas-taken}

Snapshots are counted in the quotas based on the size of the parent disks. For example, if a snapshot was created from a 250 GB disk, the quota for the size of snapshots allows for 250 GB, even if the actual size of the snapshot is 20 GB. This is done so that both developers and customers can realistically understand possible loads on the disk snapshot service.

This approach of counting the quota doesn't affect the pricing: snapshots are charged according to their real size.

#### How do I move a VM to another folder/cloud {#move-vm-folder-cloud}

1. Grant rights in your cloud to a user from another cloud:
   * Role for the cloud: `resource-manager.clouds.member`.
   * Role for the folder: `viewer` or `compute.images.user`.

   See [{#T}](../../iam/operations/roles/grant.md).
1. Create an image from your snapshot under **Disk snapshots** or from the disk itself under **Disks**.

A user in another cloud must:
1. Run the command in the [CLI](../../cli/):

   ```bash
   yc compute image create --source-image-id=<your_image_id>
   ```

1. When creating a VM, specify this image as a boot disk.

#### How do I attach a new disk to a VM {#attach-disk-to-vm}

After creating and connecting a new disk to the VM, you need to mount it or assign it a letter, depending on the operating system. See: [{#T}](../operations/vm-control/vm-attach-disk.md#mount-disk-and-fix-uuid).

#### How do I set up automatic snapshots {#set-up-automatic-snapshot-creation}

{{ compute-name }} does not currently offer a ready-made solution for automatic backups. However, you can use {{ sf-full-name }} to create scheduled snapshots on a [timer](../../functions/concepts/trigger/timer.md).

#### What happens to my data when a VM is deleted? {#delete-vm}

When you select a disk to attach to a VM, you can specify whether the disk should be deleted along with the VM. You can choose this option when creating a VM, updating it, or attaching a new disk to it.

If previously created disks are attached to the VM, they will be detached when the VM is deleted. The disk data is preserved and this disk can be attached to another VM in the future.

If you want a disk to be deleted with a VM, specify this during one of the following operations: when creating the VM, updating it, or attaching the disk to it. Such disks will be deleted when you delete the VM.

#### Do I need to stop a VM to create disk snapshots? Do I have to wait until disk snapshots are created before I start a VM? {#create-snapshot}

You don't have to stop the VM. However, keep in mind that a snapshot contains only the data written to disk when creating the snapshot. You need to take care of the data integrity yourself. For information about how to create disk snapshots, see [{#T}](../operations/disk-control/create-snapshot.md).

A snapshot is created asynchronously. You can resume writing data to disk immediately after running the create snapshot command, without waiting for the snapshot creation to be completed.