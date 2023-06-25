# Rework TP-Link wr340g v4:
The wr340g v4 router has the same Wi-Fi chip (AR9331) and very similar main board as the wr740n and supports the 802.11n wireless network standard, so we can try to increase the Wi-Fi speed from 54Mbps to 150Mbps.
We will replace RAM (HY5DU121622CTP-D43), spi flash (25Q64FVSIG) and install openwrt firmware.

# Short instruction

## Hardware preparation 
1. Solder new RAM (I used HY5DU121622CTP-D43) and a 22 ohm resistor at position R59.
![plot](./pictures/DSC_1287.JPG)
2. Solder UART pins and TX wire as shown at photo:
![plot](./pictures/DSC_1286.JPG)
3. Program bootloader from pepe2k https://github.com/pepe2k/u-boot_mod to spi flash (I used 25Q64FVSIG) and solder it.

## Software preparation
1. Make full memory dump from an old spi flash.
2. Using a hex editor, cut the last 64 KB (1F0000 - 200000) from the full dump and save it in the art.bin file. 
3. Build openwrt firmware
    1. Prepare your build system https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem
    2. Copy patch **wr340g-v4.patch** to openwrt folder. 
    3. Apply patch:
        ```bash
        git apply wr340g-v4.patch
        ```
    4. Copy ***diffconfig*** file to openwrt root folder and rename it to ***.config***
    5. Expand diffconfig to full config:
        ```bash
        make defconfig
        ```
    6. Run build 
        ```bash
        make -j$(nproc)
        ```
4. Connect lan port of the router to the PC, configure the static ip 192.168.1.2 on the PC with the mask 255.255.255.0
5. Connect to router via UART with 115200 baudrate and turn it on. 
6. Set mac address. In UART console write command
    ```bash
    setmac 90:f6:52:7a:4e:56 
    ```
7. Optional action. Run a memory test to make sure the new RAM is working properly.
    ```bash
    mtest
    ``` 
8. Start web server to upload ART and firmware partitions to the router.
    ```bash
    httpd
    ```
9. To Upload ART partition open http://192.168.1.1/art.html from your PC in browser and select ART file from step 2.
10. To Upload Firmware partition open http://192.168.1.1/ from your PC in browser and select openwrt sysupgrade file from step 3.
11. Reboot router and wait for it to load.
