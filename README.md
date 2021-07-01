# The OpenSolaris Porting Project

A goal of this project is to port OpenSolaris in some architectures.  
Most of source code is from [illumos Project](https://wiki.illumos.org/).

<!-- toc -->

- [Supported Hardware](#supported-hardware)
- [Build](#build)
  * [Build environment](#build-environment)
  * [Getting the source](#getting-the-source)
  * [Setting the environment](#setting-the-environment)
  * [Building](#building)
- [Install](#install)
  * [Overview](#overview)
  * [Server settings](#server-settings)
  * [Partitioning](#partitioning)
  * [Extracting](#extracting)

<!-- tocstop -->

## Supported Hardware

* DEC Alpha

    * [[AlphaPC 164LX]]

* 64bit ARM (ARMv8-A, aarch64)

    * [[ODROID-C2]]
    * [[QEMU AArch64]]

* 64bit RISC-V

    * [[HiFive Unleashed]]
    * [[QEMU RV64]]

## Build

### Build environment

GNU make and GCC are used for building on Linux. __Building is not supported on OpenSolaris__ because of incompatible changes.

Following packages are required on openSUSE Leap 15.1.
```
zypper install git-core gcc gcc-c++ make bison flex perl-XML-Parser libopenssl-devel  \
    liblz4-devel bzip2 patch zlib-devel ncurses-devel ksh ed mkisofs dtc \
    autoconf u-boot-tools
```

### Getting the source

```
git clone https://github.com/n-hys/illumos-gate.git
cd illumos-gate
git submodule update --init
```

### Setting the environment

You need to edit the following configuration files.

#### usr/src/mk/config.mk
|variable|default value|description|
|:-|:-|:-|
|MACH|aarch64|target architecture. aarch64, alpha and riscv64 are acceptable value.|
|ROOT|/data/proto/root_$(MACH)|build output directory. This directory is used as root directory when booting from NFS.|
|PIC_BASE|/tmp/solaris/$(MACH)/pic|output directory for temporary files|

### Building

Run make in the usr/src directory.  
It takes 13 minutes on my Threadripper 1950X machine.

```
cd usr/src
make tools
make -j

```

## Install

### Overview

There is no installer. You can install to a storage device by following the steps below.

1. Boot from the network (DHCP/BOOTP + TFTP + NFSv3)
1. Partition a hard drive (or a MMC device).
1. Initialize a file system (zfs).
1. Extract a tar ball.

### Server settings

#### DHCP

The configuration of Vendor Class Identifier is required.

|HW|Vendor Class Identifier|
|:-|:-|
|AlphaPC 164LX|SUNW.pc164lx|
|ODROID-C2|SUNW.meson|
|QEMU virt|SUNW.virt|
|HiFive Unleashed|SUNW.sifive|

Example configuration for the ISC DHCP Server.

```
option space SUNW;
option SUNW.root-mount-options code 1 = text;
option SUNW.root-server-ip-address code 2 = ip-address;
option SUNW.root-server-hostname code 3 = text;
option SUNW.root-path-name code 4 = text;
option SUNW.swap-server-ip-address code 5 = ip-address;
option SUNW.swap-file-path code 6 = text;
option SUNW.boot-file-path code 7 = text;
option SUNW.posix-timezone-string code 8 = text;
option SUNW.boot-read-size code 9 = unsigned integer 16;
option SUNW.install-server-ip-address code 10 = ip-address;
option SUNW.install-server-hostname code 11 = text;
option SUNW.install-path code 12 = text;
option SUNW.sysid-config-file-server code 13 = text;
option SUNW.JumpStart-server code 14 = text;
option SUNW.terminal-name code 15 = text;
option SUNW.standalone-boot-uri code 16 = text;
option SUNW.standalone-boot-http-proxy code 17 = text;

subnet 192.168.0.0 netmask 255.255.255.0 {
	option subnet-mask 255.255.255.0;
	option broadcast-address 192.168.0.255;
	#option routers 192.168.0.1;
	#option domain-name-servers 192.168.0.1;
	option domain-name "example.com";
	default-lease-time 14400;
	max-lease-time 172800;

	vendor-option-space SUNW;
	option SUNW.root-mount-options "rsize=32768";
	option SUNW.root-server-hostname "develop";
	option SUNW.root-server-ip-address 192.168.0.34;
	next-server 192.168.0.34;

	group { # id="Solaris-Alpha"
		option SUNW.root-path-name "/data/proto/root_alpha";
		host pc164lx {
			option vendor-class-identifier "SUNW.pc164lx";
			option host-name "pc164lx";
			filename "/root_alpha/platform/SUNW,pc164lx/inetboot";
			fixed-address 192.168.0.51;
			hardware ethernet 00:90:27:57:71:ac;
		}
	}

	group { # id="Solaris-ARMv8"
		option SUNW.root-path-name "/data/proto/root_aarch64";
		host meson {
			option host-name "meson";
			option vendor-class-identifier "SUNW.meson";
			filename "/root_aarch64/platform/SUNW,meson/inetboot";
			fixed-address 192.168.0.53;
			hardware ethernet 00:1e:06:33:77:91;
		}
	}

	group { # id="Solaris-RISC-V"
		option SUNW.root-path-name "/data/proto/root_riscv64";
		host virt {
			option vendor-class-identifier "SUNW.virt";
			option host-name "virt";
			filename "/root_riscv64/platform/SUNW,virt/inetboot";
			fixed-address 192.168.0.54;
			hardware ethernet 52:54:00:70:0a:e3;
		}
		host sifive {
			option vendor-class-identifier "SUNW.sifive";
			option host-name "sifive";
			filename "/root_riscv64/platform/SUNW,sifive/inetboot";
			fixed-address 192.168.0.55;
			hardware ethernet 70:b3:d5:92:f0:95;
		}
	}
}

```

#### TFTP
You need to change the tftp server directory to /data/proto.  
Configure TFTP_DIRECTORY in /etc/sysconfig/tftp when using openSUSE 15.1.

#### NFS
Configure a NFSv3 Server.  
The permission of root access to NFS is required.  
The following is a sample of /etc/exports.

```
/data/proto/root_aarch64	*(rw,no_root_squash,insecure,sync,no_subtree_check)
/data/proto/root_alpha		*(rw,no_root_squash,insecure,sync,no_subtree_check)
/data/proto/root_riscv64	*(rw,no_root_squash,insecure,sync,no_subtree_check)
```

### Partitioning
You can work with shell when booting from the network.  
Please see the pages of hardware for how to boot.

#### dos disk label

You need to mark the solaris partition as active.

format -> fdisk

```
             Total disk size is 3790 cylinders
             Cylinder size is 4096 (512 byte) blocks

                                               Cylinders
      Partition   Status    Type          Start   End   Length    %
      =========   ======    ============  =====   ===   ======   ===
          1                 Ext Win95         1    17      17      0
          2       Active    Solaris2         18  3789    3772    100
```

You need to mark the __tag__ of the root slice as __root__.

format -> partition

```
partition> p
Current partition table (original):
Total disk cylinders available: 3772 + 0 (reserved cylinders)

Part      Tag    Flag     Cylinders        Size            Blocks
  0       root    wm       1 - 3771        7.37GB    (3771/0/0) 15446016
  1 unassigned    wm       0               0         (0/0/0)           0
  2     backup    wu       0 - 3771        7.36GB    (3772/0/0) 15450112
  3 unassigned    wm       0               0         (0/0/0)           0
  4 unassigned    wm       0               0         (0/0/0)           0
  5 unassigned    wm       0               0         (0/0/0)           0
  6 unassigned    wm       0               0         (0/0/0)           0
  7 unassigned    wm       0               0         (0/0/0)           0
  8       boot    wu       0 -    0        2.00MB    (1/0/0)        4096
  9 unassigned    wm       0               0         (0/0/0)           0
```

#### GPT disk label

You need to use Solaris ROOT GUID (6A85CF4D-1DD2-11B2-99A6-080020736631) for the root slice.

### Extracting
You can work with shell when booting from the network.  
You need to initialize the file system and extract a tar ball.  
The following example installs to /dev/dsk/c1t0d0s0.

```
devfsadm
zpool create -f -o altroot=/mnt   rpool /dev/dsk/c1t0d0s0
zfs create rpool/ROOT
zfs create rpool/ROOT/illumos
gtar -o -xf /var/illumos.tar.gz -C /mnt/rpool/ROOT/illumos --warning=no-timestamp
chown -R dladm:netadm  /mnt/rpool/ROOT/illumos/etc/dladm
chown -R netadm:netadm /mnt/rpool/ROOT/illumos/etc/ipadm
chown -R netadm:netadm /mnt/rpool/ROOT/illumos/etc/nwam
mkdir -p /mnt/rpool/ROOT/illumos/etc/zfs
zfs unmount -a
zpool export rpool
zpool import -o cachefile=/tmp/zpool.cache -o altroot=/mnt rpool
cp /tmp/zpool.cache /mnt/rpool/ROOT/illumos/etc/zfs/
zfs unmount -a
zpool set bootfs=rpool/ROOT/illumos rpool
zpool set cachefile="" rpool
zfs set mountpoint=none   rpool
zfs set mountpoint=legacy rpool/ROOT
zfs set mountpoint=/           rpool/ROOT/illumos
zpool export rpool
```

