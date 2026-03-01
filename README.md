Build instruction for Android 10 for Rock960

# Install Deps
```
# apt install autoconf autotools-dev bc bison build-essential ccache curl device-tree-compiler dosfstools fastboot flex gcc-multilib git g++-multilib gnupg gperf htop iftop iotop lib32ncurses5-dev lib32z-dev libarchive-tools libc6-dev-i386 libgl1-mesa-dev libglib2.0-dev liblz4-tool libncurses5 libtinfo5 libusb-1.0-0 libusb-1.0-0-dev libx11-dev libxml2-utils lunzip lzop mtools openjdk-8-jdk parted pigz python2 python2-dev python3 python3-pycryptodome python-pip simg2img sysstat u-boot-tools udev unzip vim-common x11proto-core-dev xsltproc zip zlib1g-dev

$ ln -s /usr/bin/python2 ~/.local/bin/python
$ pip2 install pycrypto
```

## Download Source Code
```bash
$ repo init -u https://github.com/hitechshell/rock960_manifests -b rock960-android-10 -m rockchip-q-release.xml
$ repo sync -d -c
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

output file is `rockdev/Image/gpt.img`
