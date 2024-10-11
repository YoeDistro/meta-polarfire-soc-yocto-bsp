# Microchip PolarFire SoC Yocto BSP

Collection of OpenEmbedded/Yocto layers for PolarFire SoC.

- **meta-polarfire-soc-bsp**: layer containing PolarFire SoC's evaluation boards' metadata such as machine configuration files and core recipes (Linux, U-Boot, etc.).

- **meta-polarfire-soc-community**: layer containing Microchip's partners' evaluation kits' machine configuration files and associated configuration fragments.

- **meta-polarfire-soc-extras**: layer containing recipes that extend and supplement the meta-polarfire-soc-bsp layer. These include additional applications and demos to demonstrate PolarFire SoC features.

The complete User Guides for each development platform, containing board and boot instructions, are available for the following supported platforms:

- [ICICLE-KIT-ES](https://mi-v-ecosystem.github.io/redirects/icicle-kit-sw-developer-guide_icicle-kit-sw-developer-guide) (Icicle Kit Engineering Sample)
- [MPFS-VIDEO-KIT](https://mi-v-ecosystem.github.io/redirects/boards-mpfs-sev-kit-sev-kit-user-guide) (PolarFire SoC Video Kit)
- [MPFS-DISCO-KIT](https://mi-v-ecosystem.github.io/redirects/boards-mpfs-discovery-kit-user-guide) (PolarFire SoC Discovery Kit)
- [M100PFSEVP](https://www.aries-embedded.com/evaluation-kit/fpga/polarfire-microchip-soc-fpga-m100pfsevp-riscv-hsmc-pmod) (ARIES Embedded M100PFSEVP PolarFire SoC-FPGA Evaluation Platform)

## Build Instructions

Before continuing, ensure that the prerequisite packages are present on your system. Please see the [Host PC setup for Yocto section](#Dependencies) for further details.

### Create the Workspace

This needs to be done every time you want a clean setup based on the latest BSP.

```bash
mkdir yocto-dev && cd yocto-dev
repo init -u https://github.com/polarfire-soc/polarfire-soc-yocto-manifests.git -b 2024.09 -m default.xml
```

### Update the repo workspace

```bash
repo sync
repo rebase
```

### Setup Bitbake environment

```bash
. ./meta-polarfire-soc-yocto-bsp/polarfire-soc_yocto_setup.sh
```

### Building board Disk Image

#### Building a Linux Image with a root file system (RootFS)

Using Yocto bitbake command and setting the MACHINE and image required.

```bash
MACHINE=icicle-kit-es bitbake mpfs-dev-cli
```

For instructions on how to copy the image to the eMMC or SD card refer to the [Copy the created Disk Image to a flash device](#Copy-the-created-Disk-Image-to-flash-device) section.

#### Building a RAM-based Root Filesystem (Initramfs)

Using Yocto bitbake command and setting the initramfs configuration file (conf/initramfs.conf) and the mpfs-initramfs-image

```bash
MACHINE=icicle-kit-es bitbake -R conf/initramfs.conf mpfs-initramfs-image
```

The image generated from the command above can be used to boot Linux with a RAM-based root filesystem from the eMMC or SD card.

For instructions on how to copy the image to the eMMC or SD card refer to the [Copy the created Disk Image to a flash device](#Copy-the-created-Disk-Image-to-flash-device) section.

#### Building a Linux Image for an external QSPI flash memory

The `icicle-kit-es-nand` and `icicle-kit-es-nor` target machines provide support for building images suitable for programming to the oficially supported QSPI flash memories. The `core-image-minimal-mtdutils` generates a Linux image with either a `.nand.mtdimg` or `.nor.mtdimg` file extensions in the `build/tmp-glibc/deploy/images/<MACHINE>/` directory.

For more information on how to enable QSPI support on PolarFire SoC, please refer to the [booting from QSPI](https://mi-v-ecosystem.github.io/redirects/booting-from-qspi_booting-from-qspi) documentation.

##### Building a Linux image suitable for a Winbond W25N01GV NAND flash memory

To generate an image for the Winbond W25N01GV NAND flash memory, use the following Yocto bitbake command, which will build the `core-image-minimal-mtdutils` image:

```bash
MACHINE=icicle-kit-es-nand bitbake core-image-minimal-mtdutils
```

The created image is a '.nand.mtdimg', and is located in `yocto-dev/build/tmp-glibc/deploy/images/icicle-kit-es-nand/` directory.

Note: The `.nand.mtd` image generated by the BSP triggers a "free space fixup" procedure in the kernel the very first time the
file system is mounted. Therefore, the first mount might take additional time to complete.

For instructions on how to transfer the image to the external QSPI flash memory refer to the [Copy the created Disk Image to an external QSPI flash memory](#Copy-the-created-Disk-Image-to-an-external-QSPI-flash-memory) section.

##### Building a Linux image suitable for a Micron MT25QL256 NOR flash memory

To generate an image for the Micron MT25QL256 NOR flash memory, use the following Yocto bitbake command, which will build the `core-image-minimal-mtdutils` image:

```bash
MACHINE=icicle-kit-es-nor bitbake core-image-minimal-mtdutils
```

The created image is a '.nor.mtdimg', and is located in `yocto-dev/build/tmp-glibc/deploy/images/icicle-kit-es-nor/` directory.

For instructions on how to transfer the image to the external QSPI flash memory refer to the [Copy the created Disk Image to an external QSPI flash memory](#Copy-the-created-Disk-Image-to-an-external-QSPI-flash-memory) section.

<a name="Copy-the-created-Disk-Image-to-flash-device"></a>

### Copy the created Disk Image to flash device (USB mmc flash/SD/uSD)

We recommend using the `bmaptool` utility to program the storage device. `bmaptool` is a generic tool for creating the block map (bmap) for a file and copying files using this block map. Raw system image files can be flashed a lot faster with bmaptool than with traditional tools, like "dd" or "cp".

The created disk image is a 'wic' file, and is located in `yocto-dev/build/tmp-glibc/deploy/images/<MACHINE>/` directory,

e.g., for mpfs-dev-cli image and machine icicle-kit-es, it will be located in

`yocto-dev/build/tmp-glibc/deploy/images/icicle-kit-es/mpfs-dev-cli-icicle-kit-es.wic`.

Navigate to the build directory and flash the image using `bmaptool` command:

```bash
cd yocto-dev/build
sudo bmaptool copy tmp-glibc/deploy/images/icicle-kit-es/mpfs-dev-cli-icicle-kit-es.wic /dev/sdX
```

> Replace sdX with your drive identifier. Be very careful while picking /dev/sdX device! Look at dmesg, lsblk, GNOME Disks, etc. before and after plugging in your usb flash device/uSD/SD to find a proper device. Double check it to avoid overwriting any of system disks/partitions!
>

<a name="Copy-the-created-Disk-Image-to-an-external-QSPI-flash-memory"></a>

### Copy the created Disk Image to an external QSPI flash memory

Before proceeding with the steps shown below, make sure you have followed either the "Using a Winbond W25N01GV NAND flash memory" or
"Using a Micron MT25QL256 NOR flash memory" sections in the [booting from QSPI](https://mi-v-ecosystem.github.io/redirects/booting-from-qspi_booting-from-qspi) documentation.

Connect to UART0 (J11), and power on the board. Settings are 115200 baud, 8 data bits, 1 stop bit, no parity, and no flow control.

Press a key to stop automatic boot.  In the HSS console, type `qspi` to select the QSPI interface and then type `usbdmsc` to expose the QSPI flash memory device as a block device.

Connect the board to your host PC using J16, located beside the SD card slot.

Once this is complete, on the host PC, use `dmesg` to check what the drive identifier for the QSPI flash memory device is.

```bash
dmesg | egrep "sd"
```

The output should contain a line similar to one of the following lines:

```bash
[114353.477108] sd 11:0:0:0: [sdX] 65536 2048-byte logical blocks: (134 MB/128 MiB)
[114353.477111] sd 11:0:0:0: [sdX] Write Protect is off
[114353.477471] sd 11:0:0:0: [sdX] Mode Sense: 00 00 00 00
```

`sdX` is the drive identifier that should be used in the following commands, where `X` should be replaced with the specific character from the output of the previous command.

> Be very careful while picking /dev/sdX device! Look at dmesg, lsblk, GNOME Disks, etc. before and after plugging in your usb flash device/uSD/SD to find a proper device. Double check it to avoid overwriting any of system disks/partitions!
>

Once sure of the drive identifier, use the following command to copy your Linux image to the external QSPI flash memory device, replacing the X as appropriate:

For flashing a Linux image suitable for a **Winbond W25N01GV NAND** flash memory:

```bash
sudo dd if=tmp-glibc/deploy/images/icicle-kit-es-nand/core-image-minimal-mtdutils-icicle-kit-es-nand.nand.mtdimg of=/dev/sdX
```

For flashing a Linux image suitable for a **Micron MT25QL256 NOR** flash memory:

```bash
sudo dd if=tmp-glibc/deploy/images/icicle-kit-es-nor/core-image-minimal-mtdutils-icicle-kit-es-nor.nor.mtdimg of=/dev/sdX
```

When the transfer has completed, press CTRL+C in the HSS serial console to return to the HSS console.

Wait for the image transfer to complete. A progress bar will be shown in the HSS serial console.

To boot into Linux, type boot in the HSS console. U-Boot and Linux will use UART1. When Linux boots, log in with the username root. There is no password required.

### Supported Machine Targets

The `MACHINE` (board) option can be used to set the target board for which linux is built, and if left blank it will default to `MACHINE=icicle-kit-es`.
The following table details the available targets:

| `MACHINE`                   | Board Name                                                                                |
| --------------------------- | ----------------------------------------------------------------------------------------- |
| `MACHINE=icicle-kit-es`     | ICICLE-KIT-ES, Icicle Kit engineering samples                                             |
| `MACHINE=icicle-kit-es-amp` | ICICLE-KIT-ES, Icicle Kit engineering samples in AMP mode                                 |
| `MACHINE=icicle-kit-es-auth`| ICICLE-KIT-ES, Icicle Kit engineering samples with authenticated boot                     |
| `MACHINE=icicle-kit-es-nand`| ICICLE-KIT-ES, Icicle Kit engineering samples with Winbond W25N01GV NAND flash memory boot|
| `MACHINE=icicle-kit-es-nor` | ICICLE-KIT-ES, Icicle Kit engineering samples with Micron MT25QL256 NOR flash memory boot |
| `MACHINE=mpfs-disco-kit`    | MPFS-DISCO-KIT, PolarFire SoC Discovery Kit                                               |
| `MACHINE=mpfs-video-kit`    | MPFS250-VIDEO-KIT, PolarFire SoC Video Kit                                                |
| `MACHINE=m100pfsevp`        | M100PFSEVP, Aries M100PFSEVP PolarFire SoC-FPGA Evaluation Platform                       |
| `MACHINE=beaglev-fire`      | BeagleV-Fire, BeagleBoard.org BeagleV-Fire single-board computer (SBC)                    |

The `icicle-kit-es-amp` machine can be used to build the Icicle Kit engineering sample with AMP support. Please see the [Asymmetric Multiprocessing (AMP)](https://mi-v-ecosystem.github.io/redirects/asymmetric-multiprocessing_amp) documentation for further details.

The `icicle-kit-es-auth` machine can be used to build an image that demonstrates a simple approach
for booting an authenticated Linux kernel. Please see the [Linux Boot Authentication](https://mi-v-ecosystem.github.io/redirects/linux-boot-authentication) documentation for further details on
how to build an authentication scheme implementing a chain of trust.

When building for different 'Machines' or want a 'clean' build, we recommend deleting the 'build' directory when switching.
This will delete all cache / configurations and downloads.

```bash
cd yocto-dev
rm -rf build
```

### Linux Images

The table below summarizes the most common Linux images that can be built using this BSP.

| bitbake `<image>` argument    | Description                                                                  |
| ------------------------------| -----------------------------------------------------------------------------|
| `core-image-minimal-dev`      | A small console image with some development tools.                           |
| `core-image-minimal-mtdutils` | A small image with minimal MTD utilities to interact with QSPI flash devices |
| `mpfs-dev-cli`                | A console image with development tools.                                      |
| `mpfs-initramfs-image`        | A small RAM-based Root Filesystem (initramfs) image                          |

For more information on available images refer to [Yocto reference manual](https://docs.yoctoproject.org/ref-manual/images.html)

#### Target machine Linux login

Login with `root` account, there is no password set.

## Bitbake commands

With the bitbake environment setup, execute the bitbake command in the following format to build the disk images.

```bash
MACHINE=<machine> bitbake <image>
```

Example building the icicle-kit-es machine and the mpfs-dev-cli Linux image

```bash
MACHINE=icicle-kit-es bitbake mpfs-dev-cli
```

To work with individual recipes:

```bash
MACHINE=<MACHINE> bitbake <recipe> -c <command>
```

View/Edit the Kernel menuconfig:

```bash
MACHINE=<MACHINE> bitbake mpfs-linux -c menuconfig
```

Run the diffconfig command to prepare a configuration fragment.
The resulting file fragment.cfg should be copied to meta-polarfire-soc-yocto-bsp/recipes-kernel/linux/files directory:
Afterwards the mpfs-linux.bb src_uri should be updated to include the <fragment>.cfg,

```bash
MACHINE=<MACHINE> bitbake mpfs-linux -c diffconfig
```

**Available BSP recipes:**

- hss (Microchip Hart Software Services) payload generator
- u-boot-mchp (Microchip U-Boot)
- mpfs-linux (Linux Kernel for PolarFire SoC)

**Available commands:**

- clean
- configure
- compile
- install

### Yocto Image and Binaries directory

```bash
build/tmp-glibc/deploy/images/{MACHINE}
```

For Example the following is the path for the Icicle-kit-es

```bash
build/tmp-glibc/deploy/images/icicle-kit-es
```

<a name="Dependencies"></a>
## Host PC setup for Yocto

### Yocto Dependencies

This document assumes you are running on a modern Linux system. The process documented here was tested using Ubuntu 18.04 LTS.
It should also work with other Linux distributions if the equivalent prerequisite packages are installed.

The BSP uses the Yocto RISCV Architecture Layer, and the Yocto release Kirkstone (Revision 4.0.17) (Released March 2024).

**Make sure to install the [repo utility](https://gerrit.googlesource.com/git-repo/+/HEAD/README.md) first.**

Detailed instructions for various distributions can be found in the ["Required Packages for the Build Host"](https://docs.yoctoproject.org/4.0.17/ref-manual/system-requirements.html#required-packages-for-the-build-host) section in the Yocto Project Reference Manual.

```bash
**Note: Some extra packages are requried to support the Yocto 4.0.17 Release (codename “kirkstone”) compared to the prior release.**
```

<a name="OtherDeps"></a>
### Other Dependencies

For Ubuntu 18.04 (or newer) install python3-distutils:

```bash
sudo apt install python3-distutils
```

You can install the bmap-tools package using the following command:

```bash
sudo apt-get install bmap-tools
```

## Additional Reading

[Yocto Overview Manual](https://docs.yoctoproject.org/overview-manual/index.html)

[Yocto Development Task Manual](https://docs.yoctoproject.org/dev-manual/index.html)

[Yocto Bitbake User Manual](https://docs.yoctoproject.org/bitbake/index.html)

[Yocto Application Development and Extensible Software Development Kit (sSDK)](https://docs.yoctoproject.org/sdk-manual/index.html)

[Linux4Microchip Buildroot External for PolarFire SoC](https://github.com/linux4microchip/buildroot-external-microchip)

[U-Boot Documentation](https://www.denx.de/wiki/U-Boot/Documentation)

[Kernel Documentation for v5.12](https://www.kernel.org/doc/html/v5.12/)

[Yocto Flashing images using bmaptool](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#flashing-images-using-bmaptool)

## Licensing

This project is licensed under the terms of the MIT license (please see LICENSE file in this directory for further details).
By using the PolarFire SoC Yocto BSP layer in this repository, the user agrees to the terms and conditions from the licenses of the packages that are installed into the final image and that are covered by a commercial license.
The user also acknowledges that it's their responsibility to make sure they hold the right to use code protected by commercial agreements, whether the commercially protected packages are selected by Microchips' PolarFire SoC BSPs or by them.
Finally, the user acknowledges that it's their responsibility to make sure they hold the right to copy, use, modify, and re-distribute the intellectual property offered by this collection of meta-layers.

## Known issues

### Issue 001: Required binaries not available before creating the disk image

We sometimes get dependencies not building correctly.
During the process do_wic_install payload may not be present.

For example after requesting a complete build:

```bash
MACHINE=icicle-kit-es bitbake mpfs-dev-cli
```

If u-boot or boot.src.uimg or payload.bin is missing,
execute the following:

```bash
MACHINE=icicle-kit-es bitbake u-boot -c clean
MACHINE=icicle-kit-es bitbake u-boot -c install
```

And finally a complete build:

```bash
MACHINE=icicle-kit-es bitbake mpfs-dev-cli
```

### Issue 002 fs.inotify.max_user_watches

Listen uses inotify by default on Linux to monitor directories for changes. It's not uncommon to encounter a system limit on the number of files you can monitor. For example, Ubuntu Lucid's (64bit) inotify limit is set to 8192.

You can get your current inotify file watch limit by executing:

```bash
cat /proc/sys/fs/inotify/max_user_watches
```

When this limit is not enough to monitor all files inside a directory, the limit must be increased for Listen to work properly.

Run the following command:

```bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

[Details on max_user_watches](https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers)

See [Other Dependencies](#OtherDeps) for installation instructions.
