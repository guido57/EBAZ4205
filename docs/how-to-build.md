# How to Build 

## Vivado Project (Optional)

1. Change directory and launch Vivado GUI

    ```console
    $ cd ./vivado
    $ vivado
    ```

1. Run the following command in the Tcl console to create a Vivado project and generate a block design

    ```
    source ./ebaz4205.tcl
    ```

1. Generate Bitstream from Flow Navigator

1. In Vivado, select File -> Export -> Export Hardware

1. In the Export Hardware Platform dialog box, select the Include bitstream check box and export XSA file


## PetaLinux Project

1. Update Hardware Description (Optional)

    ```console
    $ # Change to the petalinux project directory
    $ cd ./linux/ebaz4205
    $ # Specify the directory where you exported the XSA file
    $ # Exit the configuration without any changes
    $ petalinux-config --get-hw-description=../../vivado/ebaz4205
    ```

1. Build Linux

    ```console
    $ # Change to the petalinux project directory
    $ cd ./linux/ebaz4205
    $ # Build
    $ petalinux-build
    $ # Make BOOT.BIN
    $ ./make_BOOT.BIN.sh
    ```


## microSD card

you need:
- 12 GB (or more) sd card
- fdisk
- mkfs

```
sudo fdisk /dev/sdb
Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

To delete eventual previous partitions in the SD card, do "d" as many time as many previous partitions were present
```
Command (m for help): d
```

Create two new partitions with "n" command

```
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

Steps and log for formatting:

```
$ sudo mkfs.vfat /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
```

```
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

SD EXT ROOTFS BOOT:
Mount the fat partition and copy BOOT.BIN, boot.scr, Image, and system.dtb files on it.
Mount the EXT partition and untar rootfs.tar.gz to it.
Finally unmount the SD card and use it for booting.




1. Copy the files to the first partition and write rootfs to the second partition

    ```console
    $ # Mount the first partition and copy the files
    $ sudo mount /dev/sdX1 /path/to/mountpoint
    $ cp BOOT.BIN boot.scr image.ub /path/to/mountpoint
    $ sudo umount /path/to/mountpoint
    $ # Write rootfs. If the second partition is mounted, unmount it before dd command
    $ sudo umount /dev/sdX2
    $ sudo dd if=rootfs.ext4 of=/dev/sdX2
    $ sudo resize2fs /dev/sdX2
    ```
