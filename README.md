Android 12 for Rock960 (AB version)
==========

Pre-requisites
--------------

- At least 200 GB of storage

The following packages might require building:

```
sudo apt-get update -y

sudo apt install -y asciidoc autotools-dev bash bc binutils bison \
build-essential bzip2 chrpath cpio curl cvs dblatex default-jre \
device-tree-compiler diffstat expect-dev fakeroot file flex g++ gawk gcc \
gcc-aarch64-linux-gnu gcc-arm-linux-gnueabihf g+conf genext2fs git git-core \
git-gui gitk graphviz gzip intltool lib32stdc++6 libdrm-dev libglade2-dev \
libglib2.0-dev libgtk2.0-dev liblz4-tool libncurses5 libparse-yapp-perl \
libsigsegv2 libssl-dev libudev-dev libusb-1.0-0-dev m4 make mercurial mtools \
openssh-client parted patch patchutils perl python3 python-is-python3 \
qemu-user-static rsync sed subversion swig tar texinfo u-boot-tools unzip w3m \
wget
```

Cloning the source
-------------------

```
repo init --no-tags --no-clone-bundle -u https://github.com/hitechshell/rock960_manifests -b rock960-android-12 -m rockchip-s-rock960.xml
```

Syncing the source
-------------------

After the configuration is complete, you can start compiling the firmware.

To download the code sources from the repositories to your local machine, use the following command.

```
repo sync -j$(nproc)
```

Source the Android 12 environment setup
------------------------------------------

```
source build/envsetup.sh
```

Lunch the device configuration
-------------------------------

```
lunch rk3399-userdebug
```

Building the Android 12 firmware
-----------------------------------

After the configuration is complete, you can start compiling the firmware.

```
./build.sh -UACKup
```

Note
-----

build.sh is the firmware build script that can be used to interactively build the firmware. It can accept command line arguments to customize the build process.

-U -> Build U-Boot

-A -> Build Android

-C -> Build kernel using clang compiler

-K -> Build kernel

-u -> Build rockchip update image

-p -> Pack the firmware

Result
---

`rockdev/Image-rk3399/gpt.img` is file that can be flashed by `rkdeveloptool wl 0 <filename>`

Mainline U-Boot
---

Also, we can boot android from (semi-) mainline uboot.

(One of advantage is ability boot from nvme; also in downstream uboot usb-ethernet is broken, while in mainline - not)

build u-boot (and ATF):
```
export CROSS_COMPILE=aarch64-linux-gnu-

git clone https://github.com/ARM-software/arm-trusted-firmware
cd arm-trusted-firmware
make PLAT=rk3399 bl31
cd ..
export export BL31=$(pwd)/arm-trusted-firmware/build/rk3399/release/bl31/bl31.elf
git clone https://github.com/hitechshell/rock960_u-boot -b rock960-android-12-mainline-u-boot
cd rock960_u-boot
make rock960-rk3399_defconfig
make -j$(nproc)
```
file with name u-boot-rockchip.bin can be flashed to mmc or sdcard (sdcard has priority over mmc).
```
dd if=u-boot-rockchip.bin of=/dev/mmcblk1 status=progress seek=64
```


repack boot.img and recovery.img
```
git clone https://github.com/anestisb/android-unpackbootimg
cd android-unpackbootimg
make
cd ..
mkdir work
cd work
cp <path_to_boot.img> .
cp <path_to_recovery.img> .

../android-unpackbootimg/unpackbootimg -i boot.img -o boot
../android-unpackbootimg/unpackbootimg -i recovery.img -o recovery

mkbootimg \
  --header_version 2 \
  --kernel boot/boot.img-zImage \
  --ramdisk boot/boot.img-ramdisk.gz \
  --dtb ./rk3399-rock960-ab.dtb \
  --cmdline "console=ttyFIQ0 firmware_class.path=/vendor/etc/firmware init=/init rootwait ro loop.max_part=7 androidboot.console=ttyFIQ0 androidboot.wificountrycode=CN androidboot.hardware=rk30board androidboot.boot_devices=f8000000.pcie,fe330000.sdhci androidboot.selinux=permissive earlycon=uart8250,mmio32,0xff1a0000 androidboot.mode=normal" \
  --pagesize 2048 \
  --base           0x04000000 \
  --kernel_offset  0x00080000 \
  --ramdisk_offset 0x04000000 \
  --dtb_offset     0x06000000 \
  --tags_offset    0x00000100 \
  --output new_boot.img

mkbootimg \
  --header_version 2 \
  --kernel recovery/recovery.img-zImage \
  --ramdisk recovery/recovery.img-ramdisk.gz \
  --dtb ./rk3399-rock960-ab.dtb \
  --cmdline "console=ttyFIQ0 firmware_class.path=/vendor/etc/firmware init=/init rootwait ro loop.max_part=7 androidboot.console=ttyFIQ0 androidboot.wificountrycode=CN androidboot.hardware=rk30board androidboot.boot_devices=f8000000.pcie,fe330000.sdhci androidboot.selinux=permissive earlycon=uart8250,mmio32,0xff1a0000 androidboot.mode=normal" \
  --pagesize 2048 \
  --base           0x20000000 \
  --tags_offset    0x00000100 \
  --kernel_offset  0x00008000 \
  --ramdisk_offset 0x04008000 \
  --dtb_offset     0x08008000 \
  --output new_recovery.img
```

and flash to partitions
