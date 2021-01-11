The [Hardkernel](https://www.hardkernel.com/) ODROID-C2 is a low cost development board.  
Currently, GbE, uSD, UART and RTC Shield are supported on OpenSolaris. __eMMC is not supported.__

<!-- toc -->

- [How to make a bootable uSD](#how-to-make-a-bootable-usd)
  * [Partitioning](#partitioning)
  * [Getting the source](#getting-the-source)
  * [Building and flashing](#building-and-flashing)
- [Booting from network](#booting-from-network)
- [Booting from uSD](#booting-from-usd)

<!-- tocstop -->

## How to make a bootable uSD

### Partitioning
Two partitions are required.  
Create FAT partition and Solaris in an MBR partition table.  
You need to mark the solaris partition as active.
*You should create slices with format command when booting from network for Solaris installation.*

```
Command (m for help): p
Disk /dev/sdc: 7.5 GiB, 8068792320 bytes, 15759360 sectors
Disk model:  Storage Device 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot  Start      End  Sectors  Size Id Type
/dev/sdc1         4096   135167   131072   64M  c W95 FAT32 (LBA)
/dev/sdc2  *    135168 15757311 15622144  7.5G bf Solaris
```

### Getting the source

```
git clone -b osport/v2020.02 https://github.com/n-hys/arm-trusted-firmware.git
git clone -b osport/v2019.10 https://github.com/n-hys/u-boot.git
git clone -b hardkernel/odroidc2-v2015.01 https://github.com/n-hys/u-boot.git u-boot-odroid
git clone https://github.com/n-hys/meson-tools.git
```

### Building and flashing
You should build OpenSolaris in advance.  
Refer to the following for building firmware and flashing.

```
#! /bin/bash

DISK=/dev/sdc

base_dir=$(cd $(dirname $0); pwd)
illumos_dir=${base_dir}/illumos-gate
uboot_dir=${base_dir}/u-boot
uboot_odroid_dir=${base_dir}/u-boot-odroid
atf_dir=${base_dir}/arm-trusted-firmware
fiptool_dir=${atf_dir}/tools/fiptool
mesontools_dir=${base_dir}/meson-tools

# u-boot
CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/aarch64-solaris2.11- \
	ARCH=arm \
	make -C ${uboot_dir} odroid-c2_defconfig || exit 1

CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/aarch64-solaris2.11- \
	ARCH=arm \
	make -C ${uboot_dir} -j || exit 1

# arm trusted firmware
CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/aarch64-solaris2.11- \
	make -C ${atf_dir} -j DEBUG=1 PLAT=gxbb bl31 || exit 1
make -C ${fiptool_dir} || exit 1

# meson-tools
make -C ${mesontools_dir} || exit 1

${fiptool_dir}/fiptool create --align 16384 \
	--scp-fw ${uboot_odroid_dir}/fip/gxb/bl30.bin \
	--amlogic-scp-task-fw ${uboot_odroid_dir}/fip/gxb/bl301.bin \
	--soc-fw  ${atf_dir}/build/gxbb/debug/bl31.bin \
	--nt-fw  ${uboot_dir}/u-boot.bin \
	${base_dir}/fip.bin

cat ${uboot_odroid_dir}/fip/gxb/bl2.package ${base_dir}/fip.bin > ${base_dir}/boot_new.bin
${mesontools_dir}/amlbootsig ${base_dir}/boot_new.bin ${base_dir}/u-boot.img
dd if=${base_dir}/u-boot.img of=${base_dir}/u-boot.gxbb bs=512 skip=96

BL1=${uboot_odroid_dir}/sd_fuse/bl1.bin.hardkernel
dd if=$BL1 of=$DISK conv=fsync bs=1 count=442
dd if=$BL1 of=$DISK conv=fsync bs=512 skip=1 seek=1
dd if=${base_dir}/u-boot.gxbb of=$DISK conv=fsync bs=512 seek=97
```

## Booting from network

In U-Boot, run `run enet_boot` command.  
When booting from network, some errors occurr but you can ignore errors.  
You can login as root without using password.

```
GXBB:BL1:08dafd:0a8993;FEAT:EDFC318C;POC:3;RCY:0;EMMC:800;NAND:81;SD:0;READ:0;CHK:0;
TE: 280926
no sdio debug board detected 

BL2 Built : 11:44:26, Nov 25 2015. 
gxb gfb13a3b-c2 - jcao@wonton

Board ID = 8
set vcck to 1100 mv
set vddee to 1050 mv
CPU clk: 1536MHz
DDR channel setting: DDR0 Rank0+1 same
DDR0: 2048MB(auto) @ 912MHz(2T)-13
DataBus test pass!
AddrBus test pass!
Load fip header from SD, src: 0x0000c200, des: 0x01400000, size: 0x000000b0
Load bl30 from SD, src: 0x00010200, des: 0x01000000, size: 0x00009ef0
Sending bl30........................................OK. 
Run bl30...
Load bl301 from SD, src: 0x0001c200, des: 0x01000000, size: 0x000018c0
Wait bl30...Done
Sending bl301.......OK. 
Run bl301...
31 from SD, src: 0x00020200, des: 0x10100000, size: 0x00009220


--- UART initialized after reboot ---
[Reset cause: unknown]
[Image: unknown, amlogic_Load bl33 from SD, src: 0x0002c200, des: 0x01000000, size: 0x0007cae0
v1.1.3046-00db630-dirty 2016-08-31 09:24:14 tao.zeng@droid04]
bl30: check_permit, count is 1
bl30: check_permit: ok!
chipid: ef be ad de d f0 ad ba ef be ad de not ES chip
[0.395644 Inits done]
secure task start!
high task start!
low task start!
NOTICE:  BL31: v2.2(debug):v2.2-593-gf11098fc7-dirty
NOTICE:  BL31: Built : 09:33:52, Feb  6 2020
INFO:    ARM GICv2 driver initialized
INFO:    BL31: Initializing runtime services
INFO:    BL31: cortex_a53: CPU workaround for 843419 was applied
INFO:    BL31: cortex_a53: CPU workaround for 855873 was applied
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x1000000
INFO:    SPSR = 0x3c9


U-Boot 2019.10-g5ed04291cb (Feb 06 2020 - 16:03:57 +0900) odroid-c2

Model: Hardkernel ODROID-C2
Soc:   Amlogic Meson GXBB (S905) Revision 1f:b (0:1)
DRAM:  2 GiB
MMC:   mmc@72000: 0, mmc@74000: 1
In:    serial
Out:   serial
Err:   serial
Net:   eth0: ethernet@c9410000
Hit any key to stop autoboot:  0 
=> run enet_boot
Speed: 1000, full duplex
BOOTP broadcast 1
DHCP client bound to address 192.168.5.53 (0 ms)
Using ethernet@c9410000 device
TFTP from server 192.168.5.34; our IP address is 192.168.5.53
Filename '/root_aarch64/platform/SUNW,meson/inetboot'.
Load address: 0x11000000
Loading: #################################################################
	 ##########################################################
	 6.9 MiB/s
done
Bytes transferred = 624984 (98958 hex)
Speed: 1000, full duplex
Using ethernet@c9410000 device
TFTP from server 192.168.5.34; our IP address is 192.168.5.53
Filename '/root_aarch64/platform/SUNW,meson/meson-gxbb-odroidc2.dtb'.
Load address: 0x11100000
Loading: ###
	 5.2 MiB/s
done
Bytes transferred = 10946 (2ac2 hex)
## Booting kernel from Legacy Image at 11000000 ...
   Image Name:   OpenSolaris
   Image Type:   AArch64 Linux Kernel Image (uncompressed)
   Data Size:    624920 Bytes = 610.3 KiB
   Load Address: 7c080000
   Entry Point:  7c080000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 11100000
   Booting using the fdt blob at 0x11100000
   Loading Kernel Image
   Loading Device Tree to 000000007df5e000, end 000000007df63ac1 ... OK

Starting kernel ...

phys memory add 0000000000000000 - 000000007fffffff
memory resv 0000000000000000 - 0000000000ffffff
memory resv 0000000010000000 - 00000000101fffff
bootargs=-D /soc/ethernet@c9410000
bootpath=/soc/ethernet@c9410000
vdev_probe error
rootnex_map_regspec: c9410000 -> ffff0000c9410000
rootnex_map_regspec: c8834540 -> ffff0000c8834540
WARNING: Cannot mount /system/boot
rootnex_map_regspec: c81004c0 -> ffff0000c81004c0
cpu0: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu0: Amlogic S905
cpu1: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu1: Amlogic S905
cpu1 initialization complete - online
cpu2: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu2: Amlogic S905
cpu2 initialization complete - online
cpu3: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu3: Amlogic S905
cpu3 initialization complete - online
ERROR: svc:/system/filesystem/usr:default failed to mount remount  (see 'svcs -x' for details)
Oct 25 07:18:20 svc.startd[100003]: svc:/system/filesystem/usr:default: Method "/lib/svc/method/fs-usr" failed with exit status 95.
Oct 25 07:18:20 svc.startd[100003]: system/filesystem/usr:default failed fatally: transitioned to maintenance (see 'svcs -xv' for details)
Hostname: meson
Requesting System Maintenance Mode
(See /lib/svc/share/README for more information.)
Console login service(s) cannot run
Requesting System Maintenance Mode
(See /lib/svc/share/README for more information.)
Console login service(s) cannot run

Enter user name for system maintenance (control-d to bypass): root
Enter root password (control-d to bypass): 
single-user privilege assigned to root on /dev/console.
Entering System Maintenance Mode

Oct 25 07:18:54 su: 'su root' succeeded for root on /dev/console
The Illumos Project	SunOS 5.11	SunOS Development	Feb. 05, 2020
SunOS Internal Development: non-nightly build
root@meson:~# 
```

## Booting from uSD

Copy inetboot and meson-gxbb-odroidc2.dtb to FAT Partition in uSD.  
After building OpenSolaris, these files are installed to the following.

```
/data/proto/root_aarch64/platform/SUNW,meson/inetboot
/data/proto/root_aarch64/platform/SUNW,meson/meson-gxbb-odroidc2.dtb
```

In U-Boot, run `run mmc_boot` command.

```
GXBB:BL1:08dafd:0a8993;FEAT:EDFC318C;POC:3;RCY:0;EMMC:800;NAND:81;SD:0;READ:0;CHK:0;
TE: 279009
no sdio debug board detected 

BL2 Built : 11:44:26, Nov 25 2015. 
gxb gfb13a3b-c2 - jcao@wonton

Board ID = 8
set vcck to 1100 mv
set vddee to 1050 mv
CPU clk: 1536MHz
DDR channel setting: DDR0 Rank0+1 same
DDR0: 2048MB(auto) @ 912MHz(2T)-13
DataBus test pass!
AddrBus test pass!
Load fip header from SD, src: 0x0000c200, des: 0x01400000, size: 0x000000b0
Load bl30 from SD, src: 0x00010200, des: 0x01000000, size: 0x00009ef0
Sending bl30........................................OK. 
Run bl30...
Load bl301 from SD, src: 0x0001c200, des: 0x01000000, size: 0x000018c0
Wait bl30...Done
Sending bl301.......OK. 
Run bl301...
l31 from SD, src: 0x00020200, des: 0x10100000, size: 0x00009220


--- UART initialized after reboot ---
[Reset cause: unknown]
[Image: unknown, amlogic_Load bl33 from SD, src: 0x0002c200, des: 0x01000000, size: 0x0007cae0
v1.1.3046-00db630-dirty 2016-08-31 09:24:14 tao.zeng@droid04]
bl30: check_permit, count is 1
bl30: check_permit: ok!
chipid: ef be ad de d f0 ad ba ef be ad de not ES chip
[0.393815 Inits done]
secure task start!
high task start!
low task start!
NOTICE:  BL31: v2.2(debug):v2.2-593-gf11098fc7-dirty
NOTICE:  BL31: Built : 09:33:52, Feb  6 2020
INFO:    ARM GICv2 driver initialized
INFO:    BL31: Initializing runtime services
INFO:    BL31: cortex_a53: CPU workaround for 843419 was applied
INFO:    BL31: cortex_a53: CPU workaround for 855873 was applied
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x1000000
INFO:    SPSR = 0x3c9


U-Boot 2019.10-g5ed04291cb (Feb 06 2020 - 16:03:57 +0900) odroid-c2

Model: Hardkernel ODROID-C2
Soc:   Amlogic Meson GXBB (S905) Revision 1f:b (0:1)
DRAM:  2 GiB
MMC:   mmc@72000: 0, mmc@74000: 1
In:    serial
Out:   serial
Err:   serial
Net:   eth0: ethernet@c9410000
Hit any key to stop autoboot:  0 
=> run mmc_boot
624984 bytes read in 29 ms (20.6 MiB/s)
10946 bytes read in 3 ms (3.5 MiB/s)
## Booting kernel from Legacy Image at 11000000 ...
   Image Name:   OpenSolaris
   Image Type:   AArch64 Linux Kernel Image (uncompressed)
   Data Size:    624920 Bytes = 610.3 KiB
   Load Address: 7c080000
   Entry Point:  7c080000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 11100000
   Booting using the fdt blob at 0x11100000
   Loading Kernel Image
   Loading Device Tree to 000000007df5e000, end 000000007df63ac1 ... OK

Starting kernel ...

phys memory add 0000000000000000 - 000000007fffffff
memory resv 0000000000000000 - 0000000000ffffff
memory resv 0000000010000000 - 00000000101fffff
bootargs=-D /soc/sd@d0072000/blkdev@0
bootpath=/soc/sd@d0072000/blkdev@0
High Speed supported
4bit width
zfs_lookup error /platform//boot_archive
zfs_lookup error /platform/SUNW,meson/boot_archive
zfs_lookup error /platform/hardkernel,odroid-c2/boot_archive
zfs_lookup error /platform/amlogic,meson-gxbb/boot_archive
vdev_probe error
rootnex_map_regspec: d0072000 -> ffff0000d0072000
WARNING: Cannot mount /system/boot
rootnex_map_regspec: c81004c0 -> ffff0000c81004c0
cpu0: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu0: Amlogic S905
cpu1: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu1: Amlogic S905
cpu1 initialization complete - online
cpu2: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu2: Amlogic S905
cpu2 initialization complete - online
cpu3: ARM 64bit MIDR=410fd034 REVIDR=00000080
cpu3: Amlogic S905
cpu3 initialization complete - online
Hostname: unknown

unknown console login: Oct 25 07:20:28 unknown rootnex: rootnex_map_regspec: c9410000 -> ffff0000c9410000
Oct 25 07:20:28 unknown rootnex: rootnex_map_regspec: c8834540 -> ffff0000c8834540

unknown console login: root
Oct 25 07:20:38 unknown login: Solaris_audit getaddrinfo(unknown) failed[node name or service name not known]: Error 0
Oct 25 07:20:38 unknown login: Solaris_audit adt_get_local_address failed, no Audit IP address available, faking loopback and error: Network is down
Oct 25 07:20:38 unknown login: pam_unix_cred: cannot load ttyname: Network is down, continuing.
Oct 25 07:20:38 unknown login: ROOT LOGIN /dev/console
Last login: Tue Oct 25 07:16:58 on console

The Illumos Project     SunOS 5.11      SunOS Development       Feb. 05, 2020
SunOS Internal Development: non-nightly build
root@unknown:~# 
```

