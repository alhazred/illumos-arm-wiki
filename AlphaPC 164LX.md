AlphaPC 164LX supports the 21164A Alpha processor.  
This board supports SRM Console firmware and Alpha BIOS.  
OpenSolaris requires the SRM Console firmware.

<!-- toc -->

- [Supported devices](#supported-devices)
- [Booting from network](#booting-from-network)
- [Booting from HDD](#booting-from-hdd)

<!-- tocstop -->

## Supported devices
You can use a serial port as console.(A video console is not supported.)  

Supported PCI card

* DEC DE500 Fast-Ethernet
* Intel 82559 Fast-Ethernet
* Silicon Image 3124 SATA
* Silicon Image 3132 SATA

The other PCI card supported by illumos will work well.

Currently, the bootable SCSI Card is not supported.  
You can install bootloader to IDE Hard Disk. The system can boot from SATA Hard Disk because the bootloader can load kernel from SATA Hard Disk.  
If you write the device driver for LSI 53c875, the system can boot from SCSI Hard Disk. (It is easy that you port the device driver from U-Boot)

## Booting from network

The protocols of the boot device should be set to BOOTP in SRM Console.  
For example, if the boot device is eia0, eia0_protocols should be set to BOOTP.  
Run boot command in SRM Console.  
When booting from network, some errors occurr but you can ignore errors.  
You can login as root without using password.

```
>>>boot eia
(boot eia0.0.0.9.0)

Trying BOOTP boot.

Broadcasting BOOTP Request...
Received BOOTP Packet File Name is: /root_alpha/platform/SUNW,pc164lx/inetboot
local inet address: 192.168.5.51
remote inet address: 192.168.5.34
TFTP Read File Name: /root_alpha/platform/SUNW,pc164lx/inetboot
netmask = 255.255.255.0
Server is on same subnet as client.
...........
bootstrap code read in
base = 200000, image_start = 0, image_bytes = a9c88
initializing HWRPB at 2000
initializing page table at 3ffec000
initializing machine state
setting affinity to the primary CPU
jumping to bootstrap code
bootargs=
bootpath=BOOTP 0 9 0 0 0 9 0 00-90-27-57-71-AC 1
load_ramdisk:325 open /platform/alpha/boot_archive
vdev_probe error
\
bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
WARNING: cannot open system file: /etc/system
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
WARNING: Cannot mount /system/boot
Loading smf(5) service descriptions: 138/138
ERROR: svc:/system/filesystem/usr:default failed to mount remount  (see 'svcs -x' for details)
Feb  5 07:59:08 svc.startd[100003]: svc:/system/filesystem/usr:default: Method "/lib/svc/method/fs-usr" failed with exit status 95.
Feb  5 07:59:08 svc.startd[100003]: system/filesystem/usr:default failed fatally: transitioned to maintenance (see 'svcs -xv' for details)
Requesting System Maintenance Mode
(See /lib/svc/share/README for more information.)
Console login service(s) cannot run

Enter user name for system maintenance (control-d to bypass): Hostname: pc164lx


Enter user name for system maintenance (control-d to bypass): root
Enter root password (control-d to bypass): 
single-user privilege assigned to root on /dev/console.
Entering System Maintenance Mode

Feb  5 08:06:38 su: 'su root' succeeded for root on /dev/console
The Illumos Project	SunOS 5.11	SunOS Development	Feb. 05, 2020
SunOS Internal Development: non-nightly build
root@pc164lx:~# 
```

## Booting from HDD

You can install the bootloader to a IDE Hard Disk when booting from network.

```
installboot /platform/SUNW,pc164lx/inetboot /dev/rdsk/c2d0p0
```

You can install Solaris to SATA Hard Disk when booting from network.  
Run boot command with -D *SATA Hard Disk* option in SRM Console.  

```
>>>b -fl "-D /pci@0,0/pci1095,7124@3800,0/disk@0,0" dqa
Resetting I/O buses...
(boot dqa0.0.0.11.0 -flags -D /pci@0,0/pci1095,7124@3800,0/disk@0,0)
block 0 of dqa0.0.0.11.0 is a valid boot block
reading 1359 blocks from dqa0.0.0.11.0
bootstrap code read in
base = 200000, image_start = 0, image_bytes = a9e00
initializing HWRPB at 2000
initializing page table at 3ffec000
initializing machine state
setting affinity to the primary CPU
jumping to bootstrap code
bootargs=-D /pci@0,0/pci1095,7124@3800,0/disk@0,0
bootpath=/pci@0,0/pci1095,7124@3800,0/disk@0,0
zfs_bootfs=rpool/ROOT/illumos
zfs_lookup error /platform//boot_archive
zfs_lookup error /platform/SUNW,pc164lx/boot_archive
load_ramdisk:325 open /platform/alpha/boot_archive
vdev_probe error
\
bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
WARNING: cannot open system file: /etc/system
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
WARNING: Cannot mount /system/boot
Loading smf(5) service descriptions: 138/138
Hostname: unknown

WARNING: Reboot required.
The system has updated the cache of files (boot archive) that
is used during the early boot sequence. To avoid booting and
running the system with the previously out-of-sync version of
these files, the system will be restarted.

syncing file systems... done
reset

halted CPU 0

halt code = 5
HALT instruction executed
PC = fffffffdfb02f01c
>>>b -fl "-D /pci@0,0/pci1095,7124@3800,0/disk@0,0" dqa
Resetting I/O buses...
(boot dqa0.0.0.11.0 -flags -D /pci@0,0/pci1095,7124@3800,0/disk@0,0)
block 0 of dqa0.0.0.11.0 is a valid boot block
reading 1359 blocks from dqa0.0.0.11.0
bootstrap code read in
base = 200000, image_start = 0, image_bytes = a9e00
initializing HWRPB at 2000
initializing page table at 3ffec000
initializing machine state
setting affinity to the primary CPU
jumping to bootstrap code
bootargs=-D /pci@0,0/pci1095,7124@3800,0/disk@0,0
bootpath=/pci@0,0/pci1095,7124@3800,0/disk@0,0
zfs_bootfs=rpool/ROOT/illumos
zfs_lookup error /platform//boot_archive
zfs_lookup error /platform/SUNW,pc164lx/boot_archive
load_ramdisk:325 open /platform/alpha/boot_archive
vdev_probe error
\
bop_init
done
Opening /boot/solaris/bootenv.rc
fd is 1
SunOS Release 5.11 Version SunOS_Development 64-bit
Copyright (c) 1983, 2010, Oracle and/or its affiliates. All rights reserved.
DEBUG enabled
WARNING: Cannot mount /system/boot
Hostname: unknown

unknown console login: 
```

