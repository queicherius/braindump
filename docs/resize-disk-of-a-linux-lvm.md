# Resize disk of a Linux LVM

This is a step by step guide of increasing a Linux LVM, e.g. after resizing a harddrive of a virtual server managed by ESXi. Mainly a summary of [this guide](https://www.rootusers.com/how-to-increase-the-size-of-a-linux-lvm-by-adding-a-new-disk/).

**1. Partition the disk**

First, check your existing partitions so you can later note which one is the new one:

```bash
fdisk -l
```

Next, use the `cfdisk` utility to partition the disk:

```bash
cfdisk
```

- Select either the free space or the new harddrive
- [New] -> Enter Partition Size -> [Type] -> Linux LVM -> [Write]

```bash
fdisk -l
```

Now, you should see your new partition (called `Linux LVM`). You will need the device for the next steps (e.g. `/dev/sda3`). We will need the new partition to be available everywhere, so we'll have to `reboot` the server.

**2. Extend the volume group**

```bash
# Create the physical volume for later use
pvcreate /dev/sda3

# Show the volume groups
# Remember "VG Name" (e.g. "raw-ubuntu-vg")
vgdisplay

# Extend the volume group
vgextend raw-ubuntu-vg /dev/sda3

# Scan all disks, should confirm partition
pvscan
```

**3. Extend the logical volume**

```bash
# Show the logical volumes
# Remember the "LV Path" (e.g. "/dev/raw-ubuntu-vg/root")
lvdisplay

# Extend the logical volume
lvextend /dev/raw-ubuntu-vg/root /dev/sda3
```

**4. Resize the file system**

```bash
# Resize the file system
resize2fs /dev/raw-ubuntu-vg/root

# Show that the file system is now bigger!
df -h
```