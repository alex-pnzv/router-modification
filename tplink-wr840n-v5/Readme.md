# Rework TP-Link wr840n v5:
Rework include increasing memory from 4MB to 16MB (25Q128JV) and installing openWRT instead of stock firmware

# Why was this rework needed?
My router has a broken WAN port, but wifi and all LAN ports work fine, the stock firmware doesn't have the ability to configure the LAN port as WAN. 
With stock flash memory, openwrt can't work properly, so I decided to replace the flash memory and build a latest version of openwrt firmware for this router.

# Short instruction
### 1. Back up boot and factory partitions

If openwrt is already installed on your wr840n v5:
https://openwrt.org/docs/guide-user/installation/generic.backup
```bash
root@OpenWrt:~# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00020000 00010000 "boot"
mtd1: 003d0000 00010000 "firmware"
mtd2: 0016e89d 00010000 "kernel"
mtd3: 00261760 00010000 "rootfs"
mtd4: 00050000 00010000 "rootfs_data"
mtd5: 00010000 00010000 "factory"
```

```bash
dd if=/dev/mtd0 of=/tmp/boot.backup
dd if=/dev/mtd5 of=/tmp/factory.backup
```
#### Using usb programmer:
Create fulldump of router flash.
Partition addresses on stock 4 mb flash.

```bash
[    0.334929] Creating 3 MTD partitions on "spi0.0":
[    0.339813] 0x000000000000-0x000000020000 : "boot"
[    0.345597] 0x000000020000-0x0000003f0000 : "firmware"
[    0.353651] 2 tplink-fw partitions found on MTD device firmware
[    0.359730] Creating 2 MTD partitions on "firmware":
[    0.364777] 0x000000000000-0x00000016e89d : "kernel"
[    0.370740] 0x00000016e8a0-0x0000003d0000 : "rootfs"
[    0.376545] mtd: device 3 (rootfs) set to be root filesystem
[    0.383789] 1 squashfs-split partitions found on MTD device rootfs
[    0.390140] 0x000000380000-0x0000003d0000 : "rootfs_data"
[    0.396501] 0x0000003f0000-0x000000400000 : "factory"
```
Using any hex editor cut first 128kb and save it to boot.backup file.
Using any hex editor cut last 64kb and save it to factory.backup file.


### Build openwrt firmware
https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem

1. Copy patch for 16MB flash (***901-tplink-add-16MB-flash.patch***) to folder ***openwrt/tools/firmware-utils/patches/***

2. Copy path for wr840n target (***901-add_support_for_wr840n_16MB.patch***) to openwrt root folder and apply it: 
```bash
git apply 901-add_support_for_wr840n_16MB.patch
``` 
3. Copy ***diffconfig*** file to openwrt root folder and rename it to ***.config***
4. Expand diffconfig to full config:
```bash
make defconfig
```
5 Run build 
```bash
make -j$(nproc)
```
### Prepare new spi flash
1. Open any hex editor and create new file with bootloader + firmware + factory data
```bash
0x0      - 0x1FFFF -  boot
0x20000  - 0xFEFFFF - openwrt firmware
0xFF0000 - 0xFFFFFF - factory
``` 
https://openipc.org/tools/firmware-partitions-calculation

2. Flash prepared file to new 16 mb spi flash and solder it to router.
If router does not boot after this action, need to connect to router UART with 2 serial to USB adaptors, and open 2 terminal windows for each to send data in 8N1 and receive it in 7N1 with 115200 baudrate.
To stop router booting need sent 't' in uart console.

### Flash sysupgrade file via u boot
```bash
setenv serverip 192.168.0.66
tftpboot 0x81000000 wr840n-v5-initramfs-kernel.bin
tftpboot 0x81000000 tl-wr840n-v5-squashfs-sysupgrade.bin
erase tplink 0x20000 0x59033a
cp.b 0x81000000 0x20000 0x59033a
reset
```
Note: 0x59033a is a number recived after firmware downloading.
