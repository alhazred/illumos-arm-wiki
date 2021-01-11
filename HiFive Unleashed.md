HiFive Unleashed is the RISC-V development board.  
Currently, GbE, uSD and UART are supported on OpenSolaris.

<!-- toc -->

- [DIP Switch](#dip-switch)
- [How to make a bootable uSD](#how-to-make-a-bootable-usd)
  * [Getting the source](#getting-the-source)
  * [Building and flashing](#building-and-flashing)
- [Booting from network](#booting-from-network)
- [Booting from uSD](#booting-from-usd)

<!-- tocstop -->

## DIP Switch
on: 5  
off: 1, 2, 3, 4, 6

## How to make a bootable uSD

### Getting the source

```
git clone -b osport/v2019.10 https://github.com/n-hys/u-boot.git
git clone https://github.com/n-hys/opensbi.git
git clone https://github.com/n-hys/freedom-u540-c000-bootloader.git
```

### Building and flashing
You should build OpenSolaris in advance.  
Refer to the following for building firmware and flashing.

```
#! /bin/bash

DISK=/dev/sdc

base_dir=$(cd $(dirname $0); pwd)

illumos_dir=${base_dir}/illumos-gate
opensbi_dir=${base_dir}/opensbi
uboot_dir=${base_dir}/u-boot
fsbl_dir=${base_dir}/freedom-u540-c000-bootloader
platform_dir=/data/proto/root_riscv64/platform/SUNW,sifive

# fsbl
CROSSCOMPILE=${illumos_dir}/usr/src/cross/bin/riscv64-solaris2.11- \
	make -C ${fsbl_dir} || exit 1

# u-boot
CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/riscv64-solaris2.11- \
	ARCH=riscv \
	make -C ${uboot_dir} sifive_fu540_defconfig || exit 1

CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/riscv64-solaris2.11- \
	ARCH=riscv \
	make -C ${uboot_dir} -j || exit 1

# oepnsbi
CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/riscv64-solaris2.11- \
	PLATFORM_RISCV_XLEN=64 \
	make -C ${opensbi_dir} \
		PLATFORM=sifive/fu540 \
		FW_PAYLOAD_PATH=${uboot_dir}/u-boot.bin \
		FW_PAYLOAD_FDT_PATH=${illumos_dir}/usr/src/psm/stand/boot/riscv64/sifive/hifive-unleashed-a00.dtb || exit 1

# flash
FSBL=5B193300-FC78-40CD-8002-E86C45580B47
UBOOT=2E54B353-1271-4842-806F-E436D6AF6985
VFAT=EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
SOLARIS=6A85CF4D-1DD2-11B2-99A6-080020736631

FSBL_START=2048
FSBL_END=4095
fsbl_img=${fsbl_dir}/fsbl.bin

VFAT_START=4096
VFAT_END=20479
VFAT_SIZE=$(expr ${VFAT_END} - ${VFAT_START} + 1)
vfat_img=${base_dir}/vfat.img

UBOOT_START=20480
UBOOT_END=28671
uboot_img=${opensbi_dir}/build/platform/sifive/fu540/firmware/fw_payload.bin

SOLARIS_START=28672

sgdisk -g -e --clear                                                               \
	    --new=1:${SOLARIS_START}:            --change-name=1:solaris    --typecode=1:${SOLARIS} \
	    --new=2:${FSBL_START}:${FSBL_END}    --change-name=2:fsbl       --typecode=2:${FSBL}    \
	    --new=3:${VFAT_START}:${VFAT_END}    --change-name=3:vfat       --typecode=3:${VFAT}    \
	    --new=4:${UBOOT_START}:${UBOOT_END}  --change-name=4:uboot      --typecode=4:${UBOOT}   \
	    ${DISK}

dd if=/dev/zero of=${vfat_img} bs=512 count=${VFAT_SIZE}
/sbin/mkfs.vfat ${vfat_img}
MTOOLS_SKIP_CHECK=1 mcopy -i ${vfat_img} ${platform_dir}/hifive-unleashed-a00.dtb ::hifive-unleashed-a00.dtb
MTOOLS_SKIP_CHECK=1 mcopy -i ${vfat_img} ${platform_dir}/inetboot ::inetboot

dd conv=notrunc if=${fsbl_img} of=${DISK} bs=512 seek=${FSBL_START}
dd conv=notrunc if=${vfat_img} of=${DISK} bs=512 seek=${VFAT_START}
dd conv=notrunc if=${uboot_img} of=${DISK} bs=512 seek=${UBOOT_START}
sync
```

-------

## Booting from network

In U-Boot, run `run enet_boot` command.  
When booting from network, some errors occurr but you can ignore errors.  
You can login as root without using password.

```
SiFive FSBL:       2020-01-16-f3fc96a
Using FSBL DTB
HiFive-U serial #: 00000095
Loading boot payload....


OpenSBI v0.5
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : SiFive Freedom U540
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 5
Current Hart           : 1
Firmware Base          : 0x80000000
Firmware Size          : 104 KB
Runtime SBI Version    : 0.2

PMP0: 0x0000000080000000-0x000000008001ffff (A)
PMP1: 0x0000000000000000-0x0000007fffffffff (A,R,W,X)


U-Boot 2019.10-g4b4f5e4eee-dirty (Feb 06 2020 - 21:25:38 +0900)

CPU:   rv64imafdc
Model: SiFive HiFive Unleashed A00
DRAM:  8 GiB
MMC:   spi@10050000:mmc@0: 0
In:    serial@10010000
Out:   serial@10010000
Err:   serial@10010000
Net:   eth0: ethernet@10090000
Hit any key to stop autoboot:  0 
=> run enet_boot
ethernet@10090000: PHY present at 0
ethernet@10090000: Starting autonegotiation...
ethernet@10090000: Autonegotiation complete
ethernet@10090000: link up, 1000Mbps full-duplex (lpa: 0x3c00)
BOOTP broadcast 1
DHCP client bound to address 192.168.5.55 (2 ms)
Using ethernet@10090000 device
TFTP from server 192.168.5.34; our IP address is 192.168.5.55
Filename '/root_riscv64/platform/SUNW,sifive/inetboot'.
Load address: 0x80400000
Loading: #################################################################
	 #############################################
	 6.6 MiB/s
done
Bytes transferred = 562232 (89438 hex)
ethernet@10090000: PHY present at 0
ethernet@10090000: Starting autonegotiation...
ethernet@10090000: Autonegotiation complete
ethernet@10090000: link up, 1000Mbps full-duplex (lpa: 0x3c00)
Using ethernet@10090000 device
TFTP from server 192.168.5.34; our IP address is 192.168.5.55
Filename '/root_riscv64/platform/SUNW,sifive/hifive-unleashed-a00.dtb'.
Load address: 0x80500000
Loading: ##
	 2.1 MiB/s
done
Bytes transferred = 6755 (1a63 hex)
## Booting kernel from Legacy Image at 80400000 ...
   Image Name:   OpenSolaris
   Image Type:   RISC-V Linux Kernel Image (uncompressed)
   Data Size:    562168 Bytes = 549 KiB
   Load Address: 80400000
   Entry Point:  80400000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 80500000
   Booting using the fdt blob at 0x80500000
   Loading Kernel Image
   Using Device Tree in place at 0000000080500000, end 0000000080504a62

Starting kernel ...

phys memory add 0000000080000000 - 000000027fffffff
memory resv 0000000080000000 - 00000000803fffff
add io 10090000 2000 for sifive,fu540-c000-gem
add io 100a0000 1000 for sifive,fu540-c000-gem
add io 10040000 1000 for sifive,spi0
add io 20000000 10000000 for sifive,spi0
add io 10041000 1000 for sifive,spi0
add io 30000000 10000000 for sifive,spi0
add io 10050000 1000 for sifive,spi0
add io 10030000 1000 for sifive,i2c0
add io 10010000 1000 for sifive,uart0
add io 10011000 1000 for sifive,uart0
add io 10020000 1000 for sifive,pwm0
add io 10021000 1000 for sifive,pwm0
add io 10000000 1000 for sifive,fu540-c000-prci
add io c000000 4000000 for sifive,plic-1.0.0
bootargs=-D /soc/ethernet@10090000
bootpath=/soc/ethernet@10090000
mfg_name=SUNW,sifive
bpath=/soc/ethernet@10090000
vdev_probe error
mfg_name=SUNW,sifive

bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
intr_init:1202 riscv,max-priority=1
intr_init:1203 riscv,ndev=53
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
rootnex_map_regspec: 10090000 -> ffffffc010090000
WARNING: Cannot mount /system/boot
rootnex_map_regspec: 10010000 -> ffffffc010010000
cpu0: RISC-V 64bit hart=1
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
cpu1: RISC-V 64bit hart=2
cpu1: sifive,fu540-c000
cpu1 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
cpu2: RISC-V 64bit hart=3
cpu2: sifive,fu540-c000
cpu2 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
cpu3: RISC-V 64bit hart=4
cpu3: sifive,fu540-c000
cpu3 initialization complete - online
ERROR: svc:/system/filesystem/usr:default failed to mount remount  (see 'svcs -x' for details)
Nov 10 07:03:16 svc.startd[100003]: svc:/system/filesystem/usr:default: Method "/lib/svc/method/fs-usr" failed with exit status 95.
Nov 10 07:03:16 svc.startd[100003]: system/filesystem/usr:default failed fatally: transitioned to maintenance (see 'svcs -xv' for details)
Hostname: sifive
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

Nov 10 07:03:27 su: 'su root' succeeded for root on /dev/console
The Illumos Project	SunOS 5.11	SunOS Development	Feb. 06, 2020
SunOS Internal Development: non-nightly build
root@sifive:~# 
```

-------

## Booting from uSD

In U-Boot, run `run mmc_boot` command.

```
SiFive FSBL:       2020-01-16-f3fc96a
Using FSBL DTB
HiFive-U serial #: 00000095
Loading boot payload....


OpenSBI v0.5
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : SiFive Freedom U540
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 5
Current Hart           : 1
Firmware Base          : 0x80000000
Firmware Size          : 104 KB
Runtime SBI Version    : 0.2

PMP0: 0x0000000080000000-0x000000008001ffff (A)
PMP1: 0x0000000000000000-0x0000007fffffffff (A,R,W,X)


U-Boot 2019.10-g4b4f5e4eee-dirty (Feb 06 2020 - 21:25:38 +0900)

CPU:   rv64imafdc
Model: SiFive HiFive Unleashed A00
DRAM:  8 GiB
MMC:   spi@10050000:mmc@0: 0
In:    serial@10010000
Out:   serial@10010000
Err:   serial@10010000
Net:   eth0: ethernet@10090000
Hit any key to stop autoboot:  0 
=> run mmc_boot
562232 bytes read in 291 ms (1.8 MiB/s)
6755 bytes read in 9 ms (732.4 KiB/s)
## Booting kernel from Legacy Image at 80400000 ...
   Image Name:   OpenSolaris
   Image Type:   RISC-V Linux Kernel Image (uncompressed)
   Data Size:    562168 Bytes = 549 KiB
   Load Address: 80400000
   Entry Point:  80400000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 80500000
   Booting using the fdt blob at 0x80500000
   Loading Kernel Image
   Using Device Tree in place at 0000000080500000, end 0000000080504a62

Starting kernel ...

phys memory add 0000000080000000 - 000000027fffffff
memory resv 0000000080000000 - 00000000803fffff
add io 10090000 2000 for sifive,fu540-c000-gem
add io 100a0000 1000 for sifive,fu540-c000-gem
add io 10040000 1000 for sifive,spi0
add io 20000000 10000000 for sifive,spi0
add io 10041000 1000 for sifive,spi0
add io 30000000 10000000 for sifive,spi0
add io 10050000 1000 for sifive,spi0
add io 10030000 1000 for sifive,i2c0
add io 10010000 1000 for sifive,uart0
add io 10011000 1000 for sifive,uart0
add io 10020000 1000 for sifive,pwm0
add io 10021000 1000 for sifive,pwm0
add io 10000000 1000 for sifive,fu540-c000-prci
add io c000000 4000000 for sifive,plic-1.0.0
bootargs=-D /soc/spi@10050000
bootpath=/soc/spi@10050000
bpath=/soc/spi@10050000
zfs_lookup error /platform//boot_archive
zfs_lookup error /platform/SUNW,sifive/boot_archive
zfs_lookup error /platform/sifive,hifive-unleashed-a00/boot_archive
zfs_lookup error /platform/sifive,fu540-c000/boot_archive
vdev_probe error
mfg_name=SUNW,sifive

bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
intr_init:1202 riscv,max-priority=1
intr_init:1203 riscv,ndev=53
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
rootnex_map_regspec: 10050000 -> ffffffc010050000
WARNING: Cannot mount /system/boot
rootnex_map_regspec: 10010000 -> ffffffc010010000
cpu0: RISC-V 64bit hart=1
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
cpu1: RISC-V 64bit hart=2
cpu1: sifive,fu540-c000
cpu1 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
cpu2: RISC-V 64bit hart=3
cpu2: sifive,fu540-c000
cpu2 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: index 1 is not used. hart1
plic_map: plic index 2 for hart1
plic_map: index 3 is not used. hart2
plic_map: plic index 4 for hart2
plic_map: index 5 is not used. hart3
plic_map: plic index 6 for hart3
plic_map: index 7 is not used. hart4
plic_map: plic index 8 for hart4
cpu3: RISC-V 64bit hart=4
cpu3: sifive,fu540-c000
cpu3 initialization complete - online
Hostname: unknown

unknown console login: rootnex_map_regspec: 10011000 -> ffffffc010011000

unknown console login: root

Nov 10 07:02:20 unknown login: Solaris_audit getaddrinfo(unknown) failed[node name or service name not known]: Error 0
Nov 10 07:02:20 unknown login: Solaris_audit adt_get_local_address failed, no Audit IP address available, faking loopback and error: Network is down
Nov 10 07:02:20 unknown login: pam_unix_cred: cannot load ttyname: Network is down, continuing.
Nov 10 07:02:21 unknown login: ROOT LOGIN /dev/console
Last login: Tue Nov 10 07:02:23 on console
The Illumos Project     SunOS 5.11      SunOS Development       Feb. 06, 2020
SunOS Internal Development: non-nightly build
root@unknown:~# 
```

