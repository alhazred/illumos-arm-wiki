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
./configure --target-list=riscv64-softmmu
make
sudo make install
```

### Creating a bridge interface

Create a bridge interface for using network from the QEMU virtual machine.

## How to make a bootable image

### Creating a bootable image

You should build OpenSolaris in advance.  
Get the source code of OpenSBI.

```
git clone https://github.com/n-hys/opensbi.git
```

Refer to the following for building firmware.

```
CROSS_COMPILE=${illumos_dir}/usr/src/cross/bin/riscv64-solaris2.11- \
	PLATFORM_RISCV_XLEN=64 \
	make -C ${opensbi_dir} \
		PLATFORM=qemu/virt \
		FW_PAYLOAD_PATH=${illumos_dir}/usr/src/psm/stand/boot/riscv64/virt/inetboot.bin
```

### Creating a disk image

```
qemu-img create -f raw disk.img 16G
```

## Booting from network

```
sudo /usr/local/bin/qemu-system-riscv64 -nographic -machine virt -m 1G -smp 4 \
	-kernel ${opensbi_dir}/build/platform/qemu/virt/firmware/fw_payload.elf \
	-append "-D /virtio_mmio@10008000" \
	-netdev bridge,id=net0,br=br1 \
	-device virtio-net-device,netdev=net0,mac=52:54:00:70:0a:e3 \
	-drive file=disk.img,format=raw,id=hd0 \
	-device virtio-blk-device,drive=hd0 \
	-gdb tcp::1234
```

When booting from network, some errors occurr but you can ignore errors.  
You can login as root without using password.

```
qemu-system-riscv64: warning: No -bios option specified. Not loading a firmware.
qemu-system-riscv64: warning: This default will change in a future QEMU release. Please use the -bios option to avoid breakages when this happens.
qemu-system-riscv64: warning: See QEMU's deprecation documentation for details.

OpenSBI v0.5-74-gd6fa7f9
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : QEMU Virt Machine
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 8
Current Hart           : 3
Firmware Base          : 0x80000000
Firmware Size          : 120 KB
Runtime SBI Version    : 0.2

PMP0: 0x0000000080000000-0x000000008001ffff (A)
PMP1: 0x0000000000000000-0xffffffffffffffff (A,R,W,X)
phys memory add 0000000080000000 - 00000000bfffffff
memory resv 0000000080000000 - 00000000801fffff
add io 10000000 1000 for ns16550a
add io 10008000 1000 for virtio,mmio
add io 10007000 1000 for virtio,mmio
add io 10006000 1000 for virtio,mmio
add io 10005000 1000 for virtio,mmio
add io 10004000 1000 for virtio,mmio
add io 10003000 1000 for virtio,mmio
add io 10002000 1000 for virtio,mmio
add io 10001000 1000 for virtio,mmio
add io c000000 4000000 for riscv,plic0
add io 2000000 10000 for riscv,clint0
bootargs=-D /virtio_mmio@10008000
bootpath=/virtio_mmio@10008000
mfg_name=SUNW,virt
bpath=/virtio_mmio@10008000
vdev_probe error
mfg_name=SUNW,virt

bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
intr_init:1202 riscv,max-priority=1
intr_init:1203 riscv,ndev=53
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
rootnex_map_regspec: 10008000 -> ffffffc010008000
WARNING: Cannot mount /system/boot
rootnex_map_regspec: 10000000 -> ffffffc010000000
cpu0: RISC-V 64bit hart=0
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
cpu1: RISC-V 64bit hart=1
cpu1: QEMU VIRT CPU
cpu1 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
cpu2: RISC-V 64bit hart=2
cpu2: QEMU VIRT CPU
cpu2 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
cpu3: RISC-V 64bit hart=3
cpu3: QEMU VIRT CPU
cpu3 initialization complete - online
ERROR: svc:/system/filesystem/usr:default failed to mount remount  (see 'svcs -x' for details)
Nov 10 07:03:02 svc.startd[100003]: svc:/system/filesystem/usr:default: Method "/lib/svc/method/fs-usr" failed with exit status 95.
Nov 10 07:03:02 svc.startd[100003]: system/filesystem/usr:default failed fatally: transitioned to maintenance (see 'svcs -xv' for details)
Hostname: virt
Requesting System Maintenance Mode
(See /lib/svc/share/README for more information.)
Console login service(s) cannot run

Enter user name for system maintenance (control-d to bypass): 
```

## Booting from disk image

The difference with the network boot is `-append "-D /virtio_mmio@10007000"`.

```
sudo /usr/local/bin/qemu-system-riscv64 -nographic -machine virt -m 1G -smp 4 \
	-kernel ${opensbi_dir}/build/platform/qemu/virt/firmware/fw_payload.elf \
	-append "-D /virtio_mmio@10007000" \
	-netdev bridge,id=net0,br=br1 \
	-device virtio-net-device,netdev=net0,mac=52:54:00:70:0a:e3 \
	-drive file=disk.img,format=raw,id=hd0 \
	-device virtio-blk-device,drive=hd0 \
	-gdb tcp::1234
```

```
qemu-system-riscv64: warning: No -bios option specified. Not loading a firmware.
qemu-system-riscv64: warning: This default will change in a future QEMU release. Please use the -bios option to avoid breakages when this happens.
qemu-system-riscv64: warning: See QEMU's deprecation documentation for details.

OpenSBI v0.5-74-gd6fa7f9
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : QEMU Virt Machine
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 8
Current Hart           : 0
Firmware Base          : 0x80000000
Firmware Size          : 120 KB
Runtime SBI Version    : 0.2

PMP0: 0x0000000080000000-0x000000008001ffff (A)
PMP1: 0x0000000000000000-0xffffffffffffffff (A,R,W,X)
phys memory add 0000000080000000 - 00000000bfffffff
memory resv 0000000080000000 - 00000000801fffff
add io 10000000 1000 for ns16550a
add io 10008000 1000 for virtio,mmio
add io 10007000 1000 for virtio,mmio
add io 10006000 1000 for virtio,mmio
add io 10005000 1000 for virtio,mmio
add io 10004000 1000 for virtio,mmio
add io 10003000 1000 for virtio,mmio
add io 10002000 1000 for virtio,mmio
add io 10001000 1000 for virtio,mmio
add io c000000 4000000 for riscv,plic0
add io 2000000 10000 for riscv,clint0
bootargs=-D /virtio_mmio@10007000
bootpath=/virtio_mmio@10007000
bpath=/virtio_mmio@10007000
zfs_lookup error /platform//boot_archive
zfs_lookup error /platform/SUNW,virt/boot_archive
zfs_lookup error /platform/riscv-virtio/boot_archive
vdev_probe error
mfg_name=SUNW,virt

bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
intr_init:1202 riscv,max-priority=1
intr_init:1203 riscv,ndev=53
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
rootnex_map_regspec: 10007000 -> ffffffc010007000
WARNING: Cannot mount /system/boot
rootnex_map_regspec: 10000000 -> ffffffc010000000
cpu0: RISC-V 64bit hart=0
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
cpu1: RISC-V 64bit hart=1
cpu1: QEMU VIRT CPU
cpu1 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
cpu2: RISC-V 64bit hart=2
cpu2: QEMU VIRT CPU
cpu2 initialization complete - online
plic_map: index 0 is not used. hart0
plic_map: plic index 1 for hart0
plic_map: index 2 is not used. hart1
plic_map: plic index 3 for hart1
plic_map: index 4 is not used. hart2
plic_map: plic index 5 for hart2
plic_map: index 6 is not used. hart3
plic_map: plic index 7 for hart3
cpu3: RISC-V 64bit hart=3
cpu3: QEMU VIRT CPU
cpu3 initialization complete - online
Hostname: unknown

unknown console login: 
```
