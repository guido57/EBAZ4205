
## If you prefer, manually prepare the microSD card and load files on it.

You need:
- 32 GB sd card (I used this "big" size but you can also use a smaller one. e.g. 8 GB)
- fdisk
- mkfs

From now on, I suppose that the USB card is mapped on /dev/sdb 
If not, properly change /dev/sdb1 and /dev/sdb2 in the followings 

```console
$ # unmount the sd card if it were auto mounted (eventually) 
$sudo umount /dev/sdb1
$sudo umount /dev/sdb2

$sudo fdisk /dev/sdb
Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

To delete eventual previous partitions in the SD card, do "d" as many time as many previous partitions were present

```console
Command (m for help): d
```

Create two new partitions with "n" command

```console
Command (m for help): n
Partition type
p primary (0 primary, 0 extended, 4 free)
e extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-62333951, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-62333951, default 62333951): 21111220
Creates a new partition 1 of type 'Linux' and of size 10.1 GB. Partition #1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: y
The signature will be removed by a write command.


Command (m for help): n
Partition type
p primary (1 primary, 0 extended, 3 free)
e extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2):
First sector (21111221-62333951, default 21112832):
Last sector, +sectors or +size{K,M,G,T,P} (21112832-62333951, default 62333951):
Created a new partition 2 of type 'Linux' and of size 19.7 GB.
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Steps and log for formatting the two partitions:

```console
$ sudo mkfs.vfat /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
```

```console
$ sudo mkfs.ext4 /dev/sdb2
mke2fs 1.44.1 (24-Mar-2018)
Creating file system with 5152640 4k blocks and 1289280 inodes
File system UUID: ad549f34-ee6e-4efc-ab03-fba390e98ede
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 4096000
Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and file system accounting information: done
```

Now the SD card should be ready to be loaded with files. Using fdisk it should appear in this way:

```console
$ sudo fdisk /dev/sdb

Command (m for help): p
Disk /dev/sdb: 28.87 GiB, 30979129344 bytes, 60506112 sectors
Disk model: SD/MMC/MS PRO   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x26e786d8

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sdb1           2048 21111220 21109173 10.1G 83 Linux
/dev/sdb2       21112832 60506111 39393280 18.8G 83 Linux
```

## load files onto the MicroSD card

Mount the fat partition and copy BOOT.BIN, boot.scr, Image, and system.dtb files on it.

```console
$ sudo mkdir /mnt/sdb1 
$ sudo mount /dev/sdb1 /mnt/sdb1
```

Copy files (BOOT.BIN boot.scr image.ub) to the boot partition (sdb1)

```console
$ sudo cp BOOT.BIN boot.scr image.ub /mnt/sdb1
$ sudo umount /dev/sdb1
```

Copy files to the EXT partition by untar rootfs.tar.gz to it.

```console
$ # Write rootfs. If the second partition is mounted, unmount it before dd command
$ sudo umount /dev/sdb2
$ # dd can take some time. In my Ubuntu Virtualbox on i7 10750 took 290 seconds to copy a 24MBytes rootfs.ext (which became 67 MBytes expanded on disk) 
$ sudo dd if=rootfs.ext4 of=/dev/sdb2 status=progress
$ sudo resize2fs /dev/sdb2
```
