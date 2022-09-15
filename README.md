# EBAZ4205

## Description

This repository contains the Vivado and the instructions required to get started with the Zynq EBAZ4205 Board.


## Requirement

### Hardware

* Zynq EBAZ4205 Board (cost-reduced version)
  * **No** 25MHz crystal (Y3) is required. The Ethernet transceiver (U24) clock is supplied by the ZYNQ (U31). However, it also works on a board on which a crystal is mounted
  * microSD card slot (U7) required
  * SD card boot support is required. Short the resistor (R2577)
  * Short the diode (D24) to supply power from the power connector (J4) (Optional)
  * Mount the tactile switch (S3), the capacitor (C2410) and the resistor (R2641A). The resistor (R2641A) can be shorted instead of mounting a 0 ohm resistor. I used 4.7uF for the capacitor (C2410) (Optional)


### Software tools

* Ubuntu 20.04.1 (64 bit) running on VirtualBox 6.1  
* Xilinx Vivado 2021.2
* Xilinx PetaLinux 2021.2


## How to Build 

* Create the block design, the bitstream and the hardware definition with Vivado
* [Use the hardware definition to build an bootable SD Card with petalinux and test it on EBAZ4205](./docs/how-to-build.md)



## References

### EBAZ4205

* [KeitetsuWorks EBAZ4205](https://github.com/KeitetsuWorks/EBAZ4205)

### Xilinx

* [UG585 - Zynq-7000 SoC Technical Reference Manual (v1.12.2)](https://www.xilinx.com/support/documentation/user_guides/ug585-Zynq-7000-TRM.pdf)
* [XC7Z010CLG400 ピン配置ファイル](https://japan.xilinx.com/support/packagefiles/z7packages/xc7z010clg400pkg.txt)
* [UG1144 - PetaLinux Tools Documentation Reference Guide (v2021.2)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2021_2/ug1144-petalinux-tools-reference-guide.pdf)


## License

* MIT
