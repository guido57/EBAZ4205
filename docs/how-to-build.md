# How to Build 

## Overview
 
#### Only Once: described here!
1. Basic Vivado Project. Create a basic Vivado project and get the Bitstream (PL configuration) and exported hardware from it
1. Petalinux Project. Create a SD card with Linux, based on that exported hardware

#### As many times as necessary: not described here!
1. Add/Modify IPs and Ports to the Vivado project and get the Bitstream from it
1. Load the new Bitstream from Linux



## Basic Vivado Project

1. Launch Vivado 2021.2, click Tools/Run Tcl scripts and select ebaz4205.tcl

1. Generate Bitstream from Flow Navigator

1. In Vivado, select File -> Export -> Export Hardware

1. In the Export Hardware Platform dialog box, select the Include bitstream check box and export XSA file


## PetaLinux Project

1. Only the first time, install libtinfo5

   ```console
   $ # install libtinfo5 package
   $ sudo apt-get install libtinfo5
   ```

1. Create and config the PetaLinux project

    ```console
    $ # Change to the petalinux project directories container. In my case /home/guido/Xilinx/Petalinux
    $ cd /home/guido/Xilinx/Petalinux
    
    $ # source the Petalinux settings for this directory so that you can run the Petalinux tools (petalinux-config ...) from here
    $ source 2021.2/settings.sh 
    # source 2022.1/tool/settings.sh
    
    $ # create the project. In my case the project name is ebaz4205 
    $ petalinux-create --type project --template zynq --name ebaz4205 
    $ # note that a new folder ebaz4205 has been created
    
    $ # Change to the petalinux project directory
    $ cd ebaz4205
    
    $ # Specify the directory where you exported the XSA file
    $ # Exit the configuration without any changes
    $ petalinux-config --get-hw-description=../../vivado/ebaz4205
    ```

1. Include the package libstdc that is mandatory to debug your program with Vitis. Otherwise, while you'll be tryig to debug with Vitis you will obtain:

    ```
    error while loading shared libraries: libstdc++.so.6
    ```

    To include the package libstdc, run:

    ```console
    petalinux-config -c rootfs
    ```
    then select: Filesystem Packages -> misc -> gcc-runtime -> libstdc

    Finally: Save and Exit


1. Modify the system-user.dtsi to specify the root partition

    I read that this step is optional but in my environment is mandatory because, without it, when booting from the sd card, I got:

    ```
    No filesystem could mount root, tried:
     ext3
     ext2
     ext4
     vfat
     msdos

    Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(1,0)
    ```

    Your project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi should look like this:

    ```
        /include/ "system-conf.dtsi"
        / {
            chosen {
                bootargs = "earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait cma=256M ";
            };
        };
    ```
1. Build Linux and create the sd image

    ```console
    $ # Change to the petalinux project directory
    $ cd ./linux/ebaz4205
    
    $ # Build
    $ petalinux-build
    
    $ # Make BOOT.BIN
    $ petalinux-package --boot --force --fsbl ./images/linux/zynq_fsbl.elf --fpga ./project-spec/hw-description/ebaz4205_wrapper.bit --u-boot
    ```

    I strongly "suggest" to create a .wic sd card image file to flash it directly to the sd card with BalenaEtcher or similar

    ```console
    $ # generate a .wic file 
    $ petalinux-package --wic
    ```
    
    Otherwise, [manually prepare the SD card](https://github.com/guido57/EBAZ4205/blob/master/docs/prepare%20SD.md)
    
1. Generate the SDK folder and files

   This is mandatory if you want to develop, cross compile and debug linux apps with Vitis. It will generate a folder ./images/linux/sdk with all the files of the target root filesystem:
   * include and library files necessary to cross compile and build are here (e.g. axidma.h) 
   * specific compiler and debugger
   * the "environment" to source for cross compiling
   * others

   Steps to generate the SDK folder and files:
   
   * create the directory:
      <petalinux project folder>/project-spec/meta-user/recipes-qt/qt5

   * place these two files in the just created directory: 
   
   qt3d_%.bbappend
   ```
   SRC_URI = "git://code.qt.io/qt/qt3d.git;name=qt3d;branch=5.15;protocol=git"

   SRCREV = "92853c6e1aa95dfb7d605959ff44ccc124fbd62c"
           
   ```

   qtserialbus_%.bbappend 
   ```
   SRC_URI = "git://code.qt.io/qt/qtserialbus.git;name=qt3d;branch=5.15;protocol=git"

   SRCREV = "d3394c81f10e5d5c40663e88e185335549e4bc12"
           
   ```

   * in a shell:

   ```console
   $ # Build the sdk folder and files
   $ petalinux-build --sdk
   $ cd ./images/linux/ebaz4205/images/linux
   ./sdk.sh -d ./sdk
           
   ``` 

1. Test the MicroSD just created

- Connect an USB port of your PC to the EBAZ405 board serial port using a USB to serial converter
- Run a serial terminal on your PC (e.g. putty) 
- Insert the MicroSD into the ebaz4205 sd card slot and turn on the EBAZ4205 board
- You shold see something like this:

```
U-Boot 2021.01 (Oct 12 2021 - 09:28:42 +0000)

CPU:   Zynq 7z010
Silicon: v3.1
DRAM:  ECC disabled 256 MiB
Flash: 0 Bytes
NAND:  128 MiB
MMC:   mmc@e0100000: 0
Loading Environment from FAT... *** Warning - bad CRC, using default environment

In:    serial@e0001000
Out:   serial@e0001000
Err:   serial@e0001000
Net:
ZYNQ GEM: e000b000, mdio bus e000b000, phyaddr -1, interface gmii
eth0: ethernet@e000b000
Hit any key to stop autoboot:  0
```

then 

```
Starting kernel ...
```

and finally a prompt like this:

```
root@ebaz4205:~#
```

