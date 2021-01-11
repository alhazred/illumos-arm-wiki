QEMU is a open source machine emulator.

<!-- toc -->

- [Installing QEMU](#installing-qemu)
  * [Getting the source](#getting-the-source)
  * [Building and Installing](#building-and-installing)
  * [Creating a bridge interface](#creating-a-bridge-interface)
- [How to make a bootable image](#how-to-make-a-bootable-image)
  * [Creating a bootable image](#creating-a-bootable-image)
  * [Creating a disk image](#creating-a-disk-image)
- [Booting from network](#booting-from-network)
- [Booting from disk image](#booting-from-disk-image)

<!-- tocstop -->

## Installing QEMU

### Getting the source

```
git clone https://github.com/n-hys/qemu.git
```

### Building and Installing

```
cd qemu
./configure --target-list=aarch64-softmmu
make
sudo make install
```

### Creating a bridge interface

Create a bridge interface for using network from the QEMU virtual machine.

## How to make a bootable image

### Creating a bootable image

You should build OpenSolaris in advance.  
Get the source code of ARM Trusted Firmware.

```
git clone -b osport/v2020.02 https://github.com/n-hys/arm-trusted-firmware.git
```

Refer to the following for building firmware.

```
CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/aarch64-solaris2.11- \
	make -C ${atf_dir} -j DEBUG=1 PLAT=qemu ARM_LINUX_KERNEL_AS_BL33=1 PRELOADED_BL33_BASE=0x40100000 all fip
```

### Creating a disk image

```
qemu-img create -f raw disk.img 16G
```

## Booting from network

```
qemu_cmd=/usr/local/bin/qemu-system-aarch64
bios_dir=${base_dir}/arm-trusted-firmware/build/qemu/debug
illumos_dir=${base_dir}/illumos-gate
disk_img=${base_dir}/disk.img

sudo ${qemu_cmd} -nographic -machine virt,secure=on -m 1G -smp 4 \
	-cpu cortex-a53 \
	-bios ${bios_dir}/bl1.bin \
	-device loader,file=${bios_dir}/fip.bin,addr=0x4000000 \
	-device loader,file=${illumos_dir}/usr/src/psm/stand/boot/aarch64/virt/inetboot.bin,addr=0x40100000 \
	-append "-D /virtio_mmio@a003e00" \
	-netdev bridge,id=net0,br=br1 \
	-device virtio-net-device,netdev=net0,mac=52:54:00:70:0a:e4 \
	-device virtio-blk-device,drive=hd0 \
	-drive file=${disk_img},format=raw,id=hd0,if=none \
	-semihosting-config enable,target=native \
	-gdb tcp::1234
```

When booting from network, some errors occurr but you can ignore errors.  
You can login as root without using password.

```
NOTICE:  Booting Trusted Firmware
NOTICE:  BL1: v2.2(debug):572628c3
NOTICE:  BL1: Built : 19:24:16, Feb 29 2020
INFO:    BL1: RAM 0xe04e000 - 0xe056000
WARNING: BL1: cortex_a53: CPU workaround for 835769 was missing!
WARNING: BL1: cortex_a53: CPU workaround for 843419 was missing!
WARNING: BL1: cortex_a53: CPU workaround for 855873 was missing!
INFO:    BL1: Loading BL2
INFO:    Loading image id=1 at address 0xe01b000
INFO:    Image id=1 loaded: 0xe01b000 - 0xe0231e1
NOTICE:  BL1: Booting BL2
INFO:    Entry point address = 0xe01b000
INFO:    SPSR = 0x3c5
NOTICE:  BL2: v2.2(debug):572628c3
NOTICE:  BL2: Built : 19:24:16, Feb 29 2020
INFO:    BL2: Doing platform setup
INFO:    BL2: Loading image id 3
INFO:    Loading image id=3 at address 0xe040000
INFO:    Image id=3 loaded: 0xe040000 - 0xe04a084
INFO:    BL2: Skip loading image id 5
NOTICE:  BL1: Booting BL31
INFO:    Entry point address = 0xe040000
INFO:    SPSR = 0x3cd
NOTICE:  BL31: v2.2(debug):572628c3
NOTICE:  BL31: Built : 19:24:16, Feb 29 2020
INFO:    ARM GICv2 driver initialized
INFO:    BL31: Initializing runtime services
WARNING: BL31: cortex_a53: CPU workaround for 835769 was missing!
WARNING: BL31: cortex_a53: CPU workaround for 843419 was missing!
WARNING: BL31: cortex_a53: CPU workaround for 855873 was missing!
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x40100000
INFO:    SPSR = 0x3c5
phys memory add 0000000040000000 - 000000007fffffff
add io 9040000 1000 for arm,pl011
add io 9000000 1000 for arm,pl011
add io 9010000 1000 for arm,pl031
add io 8000000 10000 for arm,cortex-a15-gic
add io 8010000 10000 for arm,cortex-a15-gic
add io a000000 1000 for virtio,mmio
add io a001000 1000 for virtio,mmio
add io a002000 1000 for virtio,mmio
add io a003000 1000 for virtio,mmio
bootargs=-D /virtio_mmio@a003e00
bootpath=/virtio_mmio@a003e00
bpath=/virtio_mmio@a003e00
vdev_probe error
rootnex_map_regspec: a003e00 -> ffff00000a003e00
WARNING: Cannot mount /system/boot
rootnex_map_regspec: 9000000 -> ffff000009000000
cpu0: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu0: QEMU VIRT CPU
cpu1: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu1: QEMU VIRT CPU
cpu1 initialization complete - online
cpu2: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu2: QEMU VIRT CPU
cpu2 initialization complete - online
cpu3: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu3: QEMU VIRT CPU
cpu3 initialization complete - online
ERROR: svc:/system/filesystem/usr:default failed to mount remount  (see 'svcs -x' for details)
Feb 29 04:41:49 svc.startd[100003]: svc:/system/filesystem/usr:default: Method "/lib/svc/method/fs-usr" failed with exit status 95.
Feb 29 04:41:49 svc.startd[100003]: system/filesystem/usr:default failed fatally: transitioned to maintenance (see 'svcs -xv' for details)
Hostname: virt-a64
Requesting System Maintenance Mode
(See /lib/svc/share/README for more information.)
Console login service(s) cannot run

Enter user name for system maintenance (control-d to bypass): root
Enter root password (control-d to bypass): 
single-user privilege assigned to root on /dev/console.
Entering System Maintenance Mode

Feb 29 04:42:12 su: 'su root' succeeded for root on /dev/console
The Illumos Project	SunOS 5.11	SunOS Development	Feb. 29, 2020
SunOS Internal Development: non-nightly build
root@virt-a64:~# 
```

## Booting from disk image

The difference with the network boot is `-append "-D /virtio_mmio@a003c00"`.

```
qemu_cmd=/usr/local/bin/qemu-system-aarch64
bios_dir=${base_dir}/arm-trusted-firmware/build/qemu/debug
illumos_dir=${base_dir}/illumos-gate
disk_img=${base_dir}/disk.img

sudo ${qemu_cmd} -nographic -machine virt,secure=on -m 1G -smp 4 \
	-cpu cortex-a53 \
	-bios ${bios_dir}/bl1.bin \
	-device loader,file=${bios_dir}/fip.bin,addr=0x4000000 \
	-device loader,file=${illumos_dir}/usr/src/psm/stand/boot/aarch64/virt/inetboot.bin,addr=0x40100000 \
	-append "-D /virtio_mmio@a003c00" \
	-netdev bridge,id=net0,br=br1 \
	-device virtio-net-device,netdev=net0,mac=52:54:00:70:0a:e4 \
	-device virtio-blk-device,drive=hd0 \
	-drive file=${disk_img},format=raw,id=hd0,if=none \
	-semihosting-config enable,target=native \
	-gdb tcp::1234
```

```
NOTICE:  Booting Trusted Firmware
NOTICE:  BL1: v2.2(debug):572628c3
NOTICE:  BL1: Built : 19:24:16, Feb 29 2020
INFO:    BL1: RAM 0xe04e000 - 0xe056000
WARNING: BL1: cortex_a53: CPU workaround for 835769 was missing!
WARNING: BL1: cortex_a53: CPU workaround for 843419 was missing!
WARNING: BL1: cortex_a53: CPU workaround for 855873 was missing!
INFO:    BL1: Loading BL2
INFO:    Loading image id=1 at address 0xe01b000
INFO:    Image id=1 loaded: 0xe01b000 - 0xe0231e1
NOTICE:  BL1: Booting BL2
INFO:    Entry point address = 0xe01b000
INFO:    SPSR = 0x3c5
NOTICE:  BL2: v2.2(debug):572628c3
NOTICE:  BL2: Built : 19:24:16, Feb 29 2020
INFO:    BL2: Doing platform setup
INFO:    BL2: Loading image id 3
INFO:    Loading image id=3 at address 0xe040000
INFO:    Image id=3 loaded: 0xe040000 - 0xe04a084
INFO:    BL2: Skip loading image id 5
NOTICE:  BL1: Booting BL31
INFO:    Entry point address = 0xe040000
INFO:    SPSR = 0x3cd
NOTICE:  BL31: v2.2(debug):572628c3
NOTICE:  BL31: Built : 19:24:16, Feb 29 2020
INFO:    ARM GICv2 driver initialized
INFO:    BL31: Initializing runtime services
WARNING: BL31: cortex_a53: CPU workaround for 835769 was missing!
WARNING: BL31: cortex_a53: CPU workaround for 843419 was missing!
WARNING: BL31: cortex_a53: CPU workaround for 855873 was missing!
INFO:    BL31: Preparing for EL3 exit to normal world
INFO:    Entry point address = 0x40100000
INFO:    SPSR = 0x3c5
phys memory add 0000000040000000 - 000000007fffffff
add io 9040000 1000 for arm,pl011
add io 9000000 1000 for arm,pl011
add io 9010000 1000 for arm,pl031
add io 8000000 10000 for arm,cortex-a15-gic
add io 8010000 10000 for arm,cortex-a15-gic
add io a000000 1000 for virtio,mmio
add io a001000 1000 for virtio,mmio
add io a002000 1000 for virtio,mmio
add io a003000 1000 for virtio,mmio
bootargs=-D /virtio_mmio@a003c00
bootpath=/virtio_mmio@a003c00
bpath=/virtio_mmio@a003c00
zfs_lookup error /platform//boot_archive
zfs_lookup error /platform/SUNW,virt/boot_archive
zfs_lookup error /platform/linux,dummy-virt/boot_archive
vdev_probe error
rootnex_map_regspec: a003c00 -> ffff00000a003c00
WARNING: Cannot mount /system/boot
rootnex_map_regspec: 9000000 -> ffff000009000000
cpu0: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu0: QEMU VIRT CPU
cpu1: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu1: QEMU VIRT CPU
cpu1 initialization complete - online
cpu2: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu2: QEMU VIRT CPU
cpu2 initialization complete - online
cpu3: ARM 64bit MIDR=410fd034 REVIDR=00000000
cpu3: QEMU VIRT CPU
cpu3 initialization complete - online
Hostname: unknown

unknown console login: Feb 29 02:27:24 unknown rootnex: rootnex_map_regspec: a003e00 -> ffff00000a003e00
root
Feb 29 02:27:53 unknown login: Solaris_audit getaddrinfo(unknown) failed[node name or service name not known]: Error 0
Feb 29 02:27:53 unknown login: Solaris_audit adt_get_local_address failed, no Audit IP address available, faking loopback and error: Network is down
Feb 29 02:27:53 unknown login: pam_unix_cred: cannot load ttyname: Network is down, continuing.
Feb 29 02:27:53 unknown login: ROOT LOGIN /dev/console
The Illumos Project     SunOS 5.11      SunOS Development       Feb. 29, 2020
SunOS Internal Development: non-nightly build
root@unknown:~#
```
