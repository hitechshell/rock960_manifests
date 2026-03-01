Build instruction for Android 10 for Rock960

# Install Deps
```
$ sudo apt install openjdk-8-jdk python3 git gnupg flex bison gperf build-essential zip curl liblz4-tool zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5 x11proto-core-dev libx11-dev lib32z-dev ccachelibgl1-mesa-dev libxml2-utils xsltproc unzip mtools u-boot-tools htop iotop sysstat iftop pigz bc device-tree-compiler lunzip dosfstools vim-common parted udev lzopi

$ ln -s /usr/bin/python2 ~/.local/bin/python
$ pip2 install pycrypto
```

## Download Source Code
```bash
$ repo init -u https://github.com/hitechshell/rock960_manifests -b rock960-android-10 -m rockchip-q-release.xml
$ repo sync -d --no-tags -j4
```

## Build Steps
### Build U-Boot
```bash
$ cd u-boot
$ ./make.sh rk3399
$ cd ..
```

### Build Kernel
```bash
$ cd kernel
$ make ARCH=arm64 rk3399_rock960_android_defconfig
$ make ARCH=arm64 rk3399-rock960-ab.img -j8
$ cd ..
```

### Build AOSP
```bash
$ source build/envsetup.sh
$ lunch rk3399-userdebug
$ make -j8
```

### Build Image
```bash
$ ln -s RKTools/linux/Linux_Pack_Firmware/rockdev/ rockdev
$ ./mkimage.sh
$ cd rockdev
$ ln -s Image-rk3399 Image
$ ./android-gpt.sh
```
