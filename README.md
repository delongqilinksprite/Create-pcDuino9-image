# 制作pcDuino9系统镜像

## 准备工作

* 1.下载pcDuino9 kernel点击[这里]()

* 2.下载pcDuino9 uboot点击[这里]()

* 3.下载pcDuino9 rootfs点击[这里](https://pan.baidu.com/s/1eSE1tfW#list/path=%2F)

* 3.宿主机安装必要的环境


在linux主机上创建一个目录名为rk-linux,然后将下载的uboot,kernel和根文件系统放到此文件下面。


### 安装必要的环境

备注：GCC版本请使用5.x版本

```
sudo apt-get install git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler gcc-aarch64-linux-gnu

sudo apt-get install gcc-arm-linux-gnueabihf libssl-dev gcc-aarch64-linux-gnu
```

### 编译内核

```
cd rk-linux/kernel/
make clean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- rockchip_linux_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4
```

### 编译uboot

```
cd /rk-linux/u-boot/
make distclean
CROSS_COMPILE=arm-linux-gnueabihf- make fennec-rk3288_defconfig all
```

### 制作boot.img

* 制作uboot-spl.img

```
cd rk-linux/
u-boot/tools/mkimage -n rk3288 -T rksd -d u-boot/spl/u-boot-spl-dtb.bin uboot-spl.img
cat u-boot/u-boot-dtb.bin >> uboot-spl.img
```

* 制作extlinux.conf

```
vim extlinux.comf

label kernel-4.4
    kernel /zImage
    fdt /rk3288-fennec.dtb
    initrd /initrd.img
    append  earlyprintk console=ttyS2,115200n8 rw root=/dev/ram0 rootfstype=ext4 init=/sbin/init ramdisk_size=49152
```



* 制作boot.img

```
cd rk-linux/
sudo dd if=/dev/zero of=boot.img bs=1M count=128
sudo mkfs.fat boot.img
mkdir boot
sudo mount boot.img boot
sudo cp kernel/arch/arm/boot/zImage boot
sudo cp kernel/arch/arm/boot/dts/rk3288-fennec.dtb boot
sudo mkdir boot/extlinux
sudo cp extlinux.conf boot/extlinux
sudo umount boot
```



### 下载根文件系统

下载根文件系统并将其重新命名

```
mv linaro-rootfs.img rootfs.img
```



### 制作卡更新系统

 * 下载[ramdisk source](https://github.com/wzyy2/rk-initrd-build)
 
```
sh ./mk-initrd.sh
```

* 格式化SD卡

```
chen@chen-HP-ProDesk-680-G1-TWR:~/work/linaro-alip/ramdisk/update$ sudo gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.8
Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present



Found valid GPT with protective MBR; using GPT.



Command (? for help): o

This option deletes all partitions and creates a new protective MBR.

Proceed? (Y/N): y



Command (? for help): n

Partition number (1-128, default 1): 1

First sector (34-126613470, default = 2048) or {+-}size{KMGTP}: 8192

Last sector (8192-126613470, default = 126613470) or {+-}size{KMGTP}: 

Current type is 'Linux filesystem'

Hex code or GUID (L to show codes, Enter = 8300): 

Changed type of partition to 'Linux filesystem'



Command (? for help): w



Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING

PARTITIONS!!



Do you want to proceed? (Y/N): y

OK; writing new GUID partition table (GPT) to /dev/sdc.

Warning: The kernel is still using the old partition table.

The new table will be used at the next reboot.

The operation has completed successfully.

```
```
sudo umount /dev/sdb1

sudo mkfs.fat /dev/sdb1
```
```
sudo dd if=uboot-spl.img of=/dev/sdb seek=64
```

* 复制 zimage, dts and ramdisk 到 /dev/sdb1

```
cd rk-linux/

cp kernel/arch/arm/boot/zImage /media/chen/9F35-9565/

cp kernel/arch/arm/boot/dts/rk3288-fennec.dtb /media/ls/9F35-9565/rk3288-fennec.dtb

cp ../rk-initrd-build/initrd.img /media/ls/9F35-9565/
```

* 在、dev/sdb1/目录添加extlinux/extlinux.conf

```
label kernel-4.4
    kernel /zImage
    fdt /rk3288-fennec.dtb
    initrd /initrd.img
    append  earlyprintk console=ttyS2,115200n8 rw root=/dev/ram0 rootfstype=ext4 init=/sbin/init ramdisk_size=49152
```

* 复制 u-boot-dtb.img uboot-spl.img boot.img rootfs.img 和 update.sh 到 /dev/sdb1

```
mkdir /media/chen/9F35-9565/update

cp u-boot-dtb.img /media/chen/9F35-9565/update

cp uboot-spl.img /media/chen/9F35-9565/update

cp boot.img /media/chen/9F35-9565/update

cp rootfs.img /media/chen/9F35-9565/update

cp update.sh /media/chen/9F35-9565/update

```

### 更新pcDuino9系统通过SD卡

将刚制作好的SD插入pcDuino9开关拨到卡启动模式，当内核起来以后，将开关拨到eMMC启动，当系统起来后，你便可以看到刚刚制作的系统镜像正在被烧入到eMMC当中。







