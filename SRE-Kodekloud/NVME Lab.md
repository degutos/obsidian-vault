
## Basic tools

### df and lsblk


```sh
degutos@centos:~$ df -hT
Filesystem                 Type      Size  Used Avail Use% Mounted on
/dev/mapper/cs_centos-root xfs        16G  4.3G   12G  28% /
devtmpfs                   devtmpfs  757M     0  757M   0% /dev
tmpfs                      tmpfs     784M     0  784M   0% /dev/shm
efivarfs                   efivarfs  256K  5.1K  251K   2% /sys/firmware/efi/efivars
tmpfs                      tmpfs     314M  6.4M  308M   3% /run
tmpfs                      tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                  xfs       2.0G  294M  1.7G  15% /boot
/dev/sda1                  vfat      599M   13M  586M   3% /boot/efi
tmpfs                      tmpfs     157M  144K  157M   1% /run/user/1000


degutos@centos:~$ lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0   20G  0 disk
├─sda1               8:1    0  600M  0 part /boot/efi
├─sda2               8:2    0    2G  0 part /boot
└─sda3               8:3    0 17.4G  0 part
  ├─cs_centos-root 253:0    0 15.4G  0 lvm  /
  └─cs_centos-swap 253:1    0    2G  0 lvm  [SWAP]
sr0                 11:0    1 1024M  0 rom
```


### Adding 02 Nvme's 10 Gb



```sh
degutos@centos:~$ df -hT
Filesystem                 Type      Size  Used Avail Use% Mounted on
/dev/mapper/cs_centos-root xfs        16G  4.3G   12G  28% /
devtmpfs                   devtmpfs  757M     0  757M   0% /dev
tmpfs                      tmpfs     784M     0  784M   0% /dev/shm
efivarfs                   efivarfs  256K  8.2K  248K   4% /sys/firmware/efi/efivars
tmpfs                      tmpfs     314M  6.4M  308M   3% /run
tmpfs                      tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                  xfs       2.0G  294M  1.7G  15% /boot
/dev/sda1                  vfat      599M   13M  586M   3% /boot/efi
tmpfs                      tmpfs     157M  112K  157M   1% /run/user/1000



degutos@centos:~$ lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0   20G  0 disk
├─sda1               8:1    0  600M  0 part /boot/efi
├─sda2               8:2    0    2G  0 part /boot
└─sda3               8:3    0 17.4G  0 part
  ├─cs_centos-root 253:0    0 15.4G  0 lvm  /
  └─cs_centos-swap 253:1    0    2G  0 lvm  [SWAP]
sr0                 11:0    1 1024M  0 rom
nvme0n1            259:0    0   10G  0 disk
nvme0n2            259:1    0   10G  0 disk
```


### blkid


```sh
degutos@centos:~$ sudo blkid
[sudo] password for degutos:
/dev/mapper/cs_centos-root: UUID="caed0bd4-535a-45d3-87b8-3727d59e929f" BLOCK_SIZE="512" TYPE="xfs"
/dev/sda3: UUID="zS3ikM-ZLmh-201A-3KId-BJh2-SDGj-xDudQG" TYPE="LVM2_member" PARTUUID="1b0f9027-812b-4096-b870-e1732ad58b10"
/dev/mapper/cs_centos-swap: UUID="4a958500-9512-4d12-b3c7-b8f114f85b55" TYPE="swap"
/dev/sda2: UUID="eaf8e197-6225-4931-b6e0-6a13c51efa89" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="671962e8-2b09-4a7c-8d7d-bae652d506cc"
/dev/sda1: UUID="5590-F109" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI System Partition" PARTUUID="2c49d39c-b411-4780-9bdd-beac63457573"
```



### Installing storage tools for debug

On the RHEL/CentOS VM:

```
sudo dnf install -y \
    nvme-cli \
    smartmontools \
    pciutils \
    lsscsi \
    fio \
    sysstat \
    iotop \
    util-linux \
    xfsprogs \
    e2fsprogs \
    lvm2 \
    device-mapper-multipath \
    sg3_utils \
    ethtool \
    strace \
    iproute
```


```sh
degutos@centos:~$ sudo dnf install -y nvme-cli smartmontools lsscsi pciutils util-linux hdparm sysstat iotop fio xfsprogs lvm2 device-mapper-multipath blktrace strace sg3_utils ipmitool ethtool dmesg
Last metadata expiration check: 1:18:31 ago on Sat 12 Sep 2026 11:21:36.
Package nvme-cli-2.16-2.el10.aarch64 is already installed.
Package smartmontools-1:7.4-9.el10.aarch64 is already installed.
Package lsscsi-0.32-17.el10.aarch64 is already installed.
Package pciutils-3.13.0-6.el10.aarch64 is already installed.
Package util-linux-2.40.2-20.el10.aarch64 is already installed.
Package hdparm-9.65-6.el10.aarch64 is already installed.
Package xfsprogs-6.16.0-1.el10.aarch64 is already installed.
Package lvm2-10:2.03.41-3.el10.aarch64 is already installed.
Package device-mapper-multipath-0.9.9-19.el10.aarch64 is already installed.
Package blktrace-1.3.0-13.el10.aarch64 is already installed.
Package strace-6.12-3.el10.aarch64 is already installed.
Package sg3_utils-1.48-10.el10.aarch64 is already installed.
Package ethtool-2:6.19-1.el10.aarch64 is already installed.
No match for argument: dmesg
Error: Unable to find a match: dmesg
```



#### Checking the important ones

```sh
degutos@centos:~$ nvme version
nvme version 2.16 (git 2.16)
libnvme version 1.16.2 (git 1.16.2)

degutos@centos:~$ smartctl -V
smartctl 7.4 2023-08-01 r5530 [aarch64-linux-6.12.0-266.el10.aarch64] (local build)
Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

smartctl comes with ABSOLUTELY NO WARRANTY. This is free
software, and you are welcome to redistribute it under
the terms of the GNU General Public License; either
version 2, or (at your option) any later version.
See https://www.gnu.org for further details.

smartmontools release 7.4 dated 2023-08-01 at 10:59:45 UTC
smartmontools SVN rev 5530 dated 2023-08-01 at 11:00:21
smartmontools build host: aarch64-redhat-linux-gnu
smartmontools build with: C++17, GCC 14.3.1 20251022 (Red Hat 14.3.1-4)
smartmontools configure arguments: [hidden in reproducible builds]
reproducible build SOURCE_DATE_EPOCH: 1765152000 (2025-12-08 00:00:00)

degutos@centos:~$ fio --version
fio-3.36

degutos@centos:~$ ipmitool -V
ipmitool version 1.8.19

degutos@centos:~$ xfs_info -V
xfs_info version 6.16.0
```



### Learning the inventory

On a real incident, **don't start running repair commands immediately**.

Start by establishing what the kernel sees.

#### Block devices

##### Checking lsblk

```sh
degutos@centos:~$ lsblk -e7 -o NAME,KNAME,TYPE,SIZE,FSTYPE,FSVER,MOUNTPOINTS,UUID,MODEL,SERIAL
NAME               KNAME   TYPE  SIZE FSTYPE      FSVER    MOUNTPOINTS UUID                                   MODEL                SERIAL
sda                sda     disk   20G                                                                         HARDDISK
├─sda1             sda1    part  600M vfat        FAT32    /boot/efi   5590-F109
├─sda2             sda2    part    2G xfs                  /boot       eaf8e197-6225-4931-b6e0-6a13c51efa89
└─sda3             sda3    part 17.4G LVM2_member LVM2 001             zS3ikM-ZLmh-201A-3KId-BJh2-SDGj-xDudQG
  ├─cs_centos-root dm-0    lvm  15.4G xfs                  /           caed0bd4-535a-45d3-87b8-3727d59e929f
  └─cs_centos-swap dm-1    lvm     2G swap        1        [SWAP]      4a958500-9512-4d12-b3c7-b8f114f85b55
sr0                sr0     rom  1024M                                                                         CD-ROM
nvme0n1            nvme0n1 disk   10G                                                                         ORCL-VBOX-NVME-VER12 VB1234-56789
nvme0n2            nvme0n2 disk   10G                                                                         ORCL-VBOX-NVME-VER12 VB1234-56789
```



##### Checking blkid


```sh
degutos@centos:~$ blkid
/dev/mapper/cs_centos-root: UUID="caed0bd4-535a-45d3-87b8-3727d59e929f" BLOCK_SIZE="512" TYPE="xfs"
/dev/sda3: UUID="zS3ikM-ZLmh-201A-3KId-BJh2-SDGj-xDudQG" TYPE="LVM2_member" PARTUUID="1b0f9027-812b-4096-b870-e1732ad58b10"
/dev/mapper/cs_centos-swap: UUID="4a958500-9512-4d12-b3c7-b8f114f85b55" TYPE="swap"
/dev/sda2: UUID="eaf8e197-6225-4931-b6e0-6a13c51efa89" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="671962e8-2b09-4a7c-8d7d-bae652d506cc"
/dev/sda1: UUID="5590-F109" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI System Partition" PARTUUID="2c49d39c-b411-4780-9bdd-beac63457573"
```


##### Findmnt

```sh
degutos@centos:~$ findmnt
TARGET                                        SOURCE                     FSTYPE          OPTIONS
/                                             /dev/mapper/cs_centos-root xfs             rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota
├─/dev                                        devtmpfs                   devtmpfs        rw,nosuid,seclabel,size=774552k,nr_inodes=193638,mode=755,inode64
│ ├─/dev/hugepages                            hugetlbfs                  hugetlbfs       rw,nosuid,nodev,relatime,seclabel,pagesize=2M
│ ├─/dev/mqueue                               mqueue                     mqueue          rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/dev/shm                                  tmpfs                      tmpfs           rw,nosuid,nodev,seclabel,inode64
│ └─/dev/pts                                  devpts                     devpts          rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000
├─/sys                                        sysfs                      sysfs           rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/fs/selinux                           selinuxfs                  selinuxfs       rw,nosuid,noexec,relatime
│ ├─/sys/kernel/debug                         debugfs                    debugfs         rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/kernel/tracing                       tracefs                    tracefs         rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/fs/fuse/connections                  fusectl                    fusectl         rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security                      securityfs                 securityfs      rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup                            cgroup2                    cgroup2         rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot
│ ├─/sys/fs/pstore                            pstore                     pstore          rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/firmware/efi/efivars                 efivarfs                   efivarfs        rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/bpf                               bpf                        bpf             rw,nosuid,nodev,noexec,relatime,mode=700
│ └─/sys/kernel/config                        configfs                   configfs        rw,nosuid,nodev,noexec,relatime
├─/proc                                       proc                       proc            rw,nosuid,nodev,noexec,relatime
│ └─/proc/sys/fs/binfmt_misc                  systemd-1                  autofs          rw,relatime,fd=36,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=5071
├─/run                                        tmpfs                      tmpfs           rw,nosuid,nodev,seclabel,size=321024k,nr_inodes=819200,mode=755,inode64
│ ├─/run/user/1000                            tmpfs                      tmpfs           rw,nosuid,nodev,relatime,seclabel,size=160508k,nr_inodes=40127,mode=700,uid=1000,gid=1000,inode64
│ │ ├─/run/user/1000/gvfs                     gvfsd-fuse                 fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
│ │ └─/run/user/1000/doc                      portal                     fuse.portal     rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
│ └─/run/credentials/systemd-journald.service tmpfs                      tmpfs           ro,nosuid,nodev,noexec,relatime,nosymfollow,seclabel,size=1024k,nr_inodes=1024,mode=700,inode64,no
└─/boot                                       /dev/sda2                  xfs             rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota
  └─/boot/efi                                 /dev/sda1                  vfat            rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=ascii,shortname=winnt,errors=remount-ro
```





```sh
degutos@centos:~$ lspci -nn
00:00.0 PCI bridge [0604]: Intel Corporation 82801 Mobile PCI Bridge [8086:2448] (rev f2)
00:01.0 System peripheral [0880]: InnoTek Systemberatung GmbH VirtualBox Guest Service [80ee:cafe]
00:02.0 Audio device [0403]: Intel Corporation 82801FB/FBM/FR/FW/FRW (ICH6 Family) High Definition Audio Controller [8086:2668] (rev 01)
00:03.0 SCSI storage controller [0100]: Red Hat, Inc. Virtio 1.0 SCSI [1af4:1048] (rev 01)
00:04.0 Non-Volatile memory controller [0108]: InnoTek Systemberatung GmbH Device [80ee:4e56]
00:06.0 USB controller [0c03]: Intel Corporation 7 Series/C210 Series Chipset Family USB xHCI Host Controller [8086:1e31]
00:08.0 Ethernet controller [0200]: Intel Corporation 82540EM Gigabit Ethernet Controller [8086:100e] (rev 02)
```


##### Filter nvme

```sh
lspci -nn | grep -i nvme
```

Eventually we could get something, if not we can check manually

**`00:04.0` is your NVMe controller**.

```
00:04.0 Non-Volatile memory controller [0108]:
    InnoTek Systemberatung GmbH Device [80ee:4e56]
```

The giveaway is:

```
[0108] → PCI class 01 = mass storage
          subclass 08 = Non-Volatile Memory
```

The **Virtio SCSI controller at `00:03.0` is a separate storage controller** and is not NVMe:

```
00:03.0 SCSI storage controller
         Red Hat Virtio 1.0 SCSI
```

But here's the important next step
You want to connect the **PCI controller** to the **Linux NVMe device**.

We need investigate the PCI_ADDRESS (00:04.0)

```sh
degutos@centos:~$ lspci -nnk -s 00:04.0
00:04.0 Non-Volatile memory controller [0108]: InnoTek Systemberatung GmbH Device [80ee:4e56]
        Kernel driver in use: nvme
        Kernel modules: nvme
```

OR

This is a really important step:

```sh
degutos@centos:~$ lspci -vv -s 00:04.0
00:04.0 Non-Volatile memory controller: InnoTek Systemberatung GmbH Device 4e56 (prog-if 02 [NVM Express])
        Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV+ VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 64
        Interrupt: pin A routed to IRQ 15
        Region 0: Memory at 88c30000 (32-bit, non-prefetchable) [size=32K]
        Region 2: I/O ports at 1020 [size=8]
        Region 3: Memory at 88000000 (32-bit, non-prefetchable) [size=8M]
        Kernel driver in use: nvme
        Kernel modules: nvme
```




### Kernel side of NVMe troubleshooting 

This is even more IMPORTANT than nvme-cli

```sh
degutos@centos:~$ sudo dmesg -T | grep -i nvme
[Sat Sep 12 01:00:53 2026] nvme nvme0: pci function 0000:00:04.0
[Sat Sep 12 01:00:53 2026] nvme nvme0: 1/0/0 default/read/poll queues
[Sat Sep 12 01:00:54 2026] nvme nvme0: using unchecked data buffer
[Sat Sep 12 01:00:54 2026] block nvme0n1: No UUID available providing old NGUID
```


```sh
degutos@centos:~$ journalctl -k | grep -Ei 'nvme|pcie|aer'
Sep 12 01:00:39 centos kernel: acpi PNP0A08:00: _OSC: OS requested [PCIeHotplug PME AER PCIeCapability LTR DPC]
Sep 12 01:00:39 centos kernel: acpi PNP0A08:00: _OSC: platform willing to grant [PCIeHotplug PME AER PCIeCapability LTR DPC]
Sep 12 01:00:39 centos kernel: acpi PNP0A08:00: _OSC: platform retains control of PCIe features (AE_ERROR)
Sep 12 01:00:53 centos kernel: nvme nvme0: pci function 0000:00:04.0
Sep 12 01:00:53 centos kernel: nvme nvme0: 1/0/0 default/read/poll queues
Sep 12 01:00:54 centos kernel: nvme nvme0: using unchecked data buffer
Sep 12 01:00:54 centos kernel: block nvme0n1: No UUID available providing old NGUID
```


Run full journalctl, this might give us large output and many log lines 

```sh
journalctl -k -b
```


Lets look for classic storage symptoms

```sh
$ journalctl -k | grep -Ei \
'blk_update_request|I/O error|Buffer I/O error|timeout|reset|abort|nvme|aer|pcie'
```


```sh
degutos@centos:~$ journalctl -k | grep -Ei \
'blk_update_request|I/O error|Buffer I/O error|timeout|reset|abort|nvme|aer|pcie'
Sep 12 01:00:39 centos kernel: acpi PNP0A08:00: _OSC: OS requested [PCIeHotplug PME AER PCIeCapability LTR DPC]
Sep 12 01:00:39 centos kernel: acpi PNP0A08:00: _OSC: platform willing to grant [PCIeHotplug PME AER PCIeCapability LTR DPC]
Sep 12 01:00:39 centos kernel: acpi PNP0A08:00: _OSC: platform retains control of PCIe features (AE_ERROR)
Sep 12 01:00:53 centos kernel: nvme nvme0: pci function 0000:00:04.0
Sep 12 01:00:53 centos kernel: nvme nvme0: 1/0/0 default/read/poll queues
Sep 12 01:00:54 centos kernel: nvme nvme0: using unchecked data buffer
Sep 12 01:00:54 centos kernel: block nvme0n1: No UUID available providing old NGUID
```

We'll want to become comfortable recognizing things such as:

```
nvme nvme0: I/O 123 QID 4 timeout
```

```
nvme nvme0: controller is down
```

```
nvme nvme0: resetting controller
```

```
blk_update_request: I/O error
```

```
pcieport ... AER: Corrected error
```

The **timeline** is often the key:

```
PCIe/AER event
       ↓
NVMe timeout
       ↓
controller reset
       ↓
I/O errors
       ↓
filesystem detects problems
       ↓
filesystem becomes read-only
       ↓
application starts failing
```

That's the chain I'd want you to practice recognizing.




### nvme-cli

The nvme-cli is the most important tool for this kind of troubleshooting:

```sh
degutos@centos:~$ nvme list
Node                  Generic               SN                   Model                                    Namespace  Usage                      Format           FW Rev
--------------------- --------------------- -------------------- ---------------------------------------- ---------- -------------------------- ---------------- --------
/dev/nvme0n1          /dev/ng0n1            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x1         10.74  GB /  10.74  GB    512   B +  0 B   1.0
/dev/nvme0n2          /dev/ng0n2            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x2         10.74  GB /  10.74  GB    512   B +  0 B   1.0
```


```sh
degutos@centos:~$ nvme list-subsys
nvme-subsys0 - NQN=nqn.2014.08.org.nvmexpress:80ee80eeVB1234-56789        ORCL-VBOX-NVME-VER12
               hostnqn=nqn.2014-08.org.nvmexpress:uuid:f5aab8b7-89ca-42c4-bf97-f28a5b78aec0
               iopolicy=numa
\
 +- nvme0 pcie 0000:00:04.0 live
```

Controller information:

```sh
degutos@centos:~$ sudo nvme id-ctrl /dev/nvme0
[sudo] password for degutos:
NVME Identify Controller:
vid       : 0x80ee
ssvid     : 0x80ee
sn        : VB1234-56789
mn        : ORCL-VBOX-NVME-VER12
fr        : 1.0
rab       : 0
ieee      : 000000
cmic      : 0
mdts      : 0
cntlid    : 0
ver       : 0x10200
rtd3r     : 0x1
rtd3e     : 0x1
oaes      : 0
ctratt    : 0
rrls      : 0
bpcap     : 0
nssl      : 0
plsi      : 0
cntrltype : 0
fguid     : 00000000-0000-0000-0000-000000000000
crdt1     : 0
crdt2     : 0
crdt3     : 0
crcap     : 0
nvmsr     : 0
vwci      : 0
mec       : 0
oacs      : 0
acl       : 4
aerl      : 4
frmw      : 0x2
lpa       : 0
elpe      : 0
npss      : 0
avscc     : 0
apsta     : 0
wctemp    : 343
cctemp    : 343
mtfa      : 0
hmpre     : 0
hmmin     : 0
tnvmcap   : 0
unvmcap   : 0
rpmbs     : 0
edstt     : 0
dsto      : 0
fwug      : 0
kas       : 0
hctma     : 0
mntmt     : 0
mxtmt     : 0
sanicap   : 0
hmminds   : 0
hmmaxd    : 0
nsetidmax : 0
endgidmax : 0
anatt     : 0
anacap    : 0
anagrpmax : 0
nanagrpid : 0
pels      : 0
domainid  : 0
kpioc     : 0
mptfawr   : 0
megcap    : 0
tmpthha   : 0
cqt       : 0
sqes      : 0x66
cqes      : 0x44
maxcmd    : 0
nn        : 2
oncs      : 0
fuses     : 0
fna       : 0
vwc       : 0
awun      : 0
awupf     : 0
icsvscc   : 0
nwpc      : 0
acwu      : 0
ocfs      : 0
sgls      : 0
mnan      : 0
maxdna    : 0
maxcna    : 0
oaqd      : 0
rhiri     : 0
hirt      : 0
cmmrtd    : 0
nmmrtd    : 0
minmrtg   : 0
maxmrtg   : 0
trattr    : 0
mcudmq    : 0
mnsudmq   : 0
mcmr      : 0
nmcmr     : 0
mcdqpc    : 0
subnqn    :
ioccsz    : 0
iorcsz    : 0
icdoff    : 0
fcatt     : 0
msdbd     : 0
ofcs      : 0
ps      0 : mp:0.01W operational enlat:0 exlat:0 rrt:0 rrl:0
            rwt:0 rwl:0 idle_power:- active_power:-
            active_power_workload:-
            emergency power fail recovery time: -
            forced quiescence vault time: -
            emergency power fail vault time: -
```


Namespace information:

```sh
degutos@centos:~$ sudo nvme id-ns /dev/nvme0n1
NVME Identify Namespace 1:
nsze    : 0x1400000
ncap    : 0x1400000
nuse    : 0x1400000
nsfeat  : 0
nlbaf   : 0
flbas   : 0
mc      : 0
dpc     : 0
dps     : 0
nmic    : 0
rescap  : 0
fpi     : 0
dlfeat  : 0
nawun   : 0
nawupf  : 0
nacwu   : 0
nabsn   : 0
nabo    : 0
nabspf  : 0
noiob   : 0
nvmcap  : 0
mssrl   : 0
mcl     : 0
msrc    : 0
kpios   : 0
nulbaf  : 0
kpiodaag: 0
anagrpid: 0
nsattr  : 0
nvmsetid: 0
endgid  : 0
nguid   : 55731efc946e9642bf4255f15f1344d0
eui64   : 0000000000000000
lbaf  0 : ms:0   lbads:9  rp:0 (in use)
```


Important health check:

```sh
degutos@centos:~$ sudo nvme smart-log /dev/nvme0
Smart Log for NVME device:nvme0 namespace-id:ffffffff
critical_warning                        : 0
temperature                             : -273 °C (0 K, -459 °F)
available_spare                         : 0%
available_spare_threshold               : 0%
percentage_used                         : 0%
endurance group critical warning summary: 0
Data Units Read                         : 0 (0.00 B)
Data Units Written                      : 0 (0.00 B)
host_read_commands                      : 0
host_write_commands                     : 0
controller_busy_time                    : 0
power_cycles                            : 0
power_on_hours                          : 0
unsafe_shutdowns                        : 0
media_errors                            : 0
num_err_log_entries                     : 0
Warning Temperature Time                : 0
Critical Composite Temperature Time     : 0
Thermal Management T1 Trans Count       : 0
Thermal Management T2 Trans Count       : 0
Thermal Management T1 Total Time        : 0
Thermal Management T2 Total Time        : 0
```


Error log:
This is command we inspect the NVMe error, which is really IMPORTANT

```sh
degutos@centos:~$ sudo nvme error-log /dev/nvme0
Error Log Entries for device:nvme0 entries:1
.................
 Entry[ 0]
.................
error_count     : 0
sqid            : 0
cmdid           : 0
status_field    : 0 (Successful Completion: The command completed without error)
phase_tag       : 0
parm_err_loc    : 0
lba             : 0
nsid            : 0
vs              : 0
trtype          : 0 (The transport type is not indicated or the error is not transport related)
csi             : 0
opcode          : 0
cs              : 0
trtype_spec_info: 0
log_page_version: 0
.................
```


Firmware:

```sh
degutos@centos:~$ sudo nvme id-ctrl /dev/nvme0 | grep -i fr
fr        : 1.0
frmw      : 0x2
```


Controller information:

```sh
degutos@centos:~$ sudo nvme show-regs /dev/nvme0
cap     : 0x200a010fff
version : 0x10200
intms   : 0
intmc   : 0
cc      : 0x460001
csts    : 0x1
nssr    : 0
aqa     : 0x1f001f
asq     : 0xffe5000
acq     : 0xffe6000
cmbloc  : 0x3
cmbsz   : 0x500003
bpinfo  : 0xffffffff
bprsel  : 0xffffffff
bpmbl   : 0xffffffffffffffff
cmbmsc  : 0xffffffffffffffff
cmbsts  : 0xffffffff
cmbebs  : 0xffffffff
cmbswtp : 0xffffffff
nssd    : 0xffffffff
crto    : 0xffffffff
pmrcap  : 0xffffffff
pmrctl  : 0xffffffff
pmrsts  : 0xffffffff
pmrebs  : 0xffffffff
pmrswtp : 0xffffffff
pmrmscl : 0xffffffff
pmrmscu : 0xffffffff
```


#### NVMe health check script

A nice little collection script for the lab:

```sh
#!/bin/bash

echo "===== DATE ====="
date

echo "===== KERNEL ====="
uname -a

echo "===== BLOCK DEVICES ====="
lsblk -e7 -o NAME,KNAME,TYPE,SIZE,FSTYPE,MOUNTPOINTS,MODEL,SERIAL

echo "===== NVME LIST ====="
nvme list

echo "===== NVME SUBSYSTEMS ====="
nvme list-subsys

for dev in /dev/nvme[0-9]; do
    [ -e "$dev" ] || continue

    echo
    echo "===== $dev SMART ====="
    nvme smart-log "$dev"

    echo
    echo "===== $dev ERROR LOG ====="
    nvme error-log "$dev"
done

echo
echo "===== PCI ====="
lspci -nn | grep -i nvme

echo
echo "===== KERNEL STORAGE LOGS ====="
dmesg -T | grep -Ei 'nvme|pcie|aer|blk_update|I/O error|filesystem|xfs|ext4'
```

That's a very realistic SRE-style diagnostic starting point.

Output script command:

```sh
degutos@centos:~$ sudo ./nvme-hc.sh
===== DATE =====
Sat 12 Sep 2026 13:58:30 IST
===== KERNEL =====
Linux centos 6.12.0-266.el10.aarch64 #1 SMP PREEMPT_DYNAMIC Fri Sep  4 09:05:26 UTC 2026 aarch64 GNU/Linux
===== BLOCK DEVICES =====
NAME               KNAME   TYPE  SIZE FSTYPE      MOUNTPOINTS MODEL                SERIAL
sda                sda     disk   20G                         HARDDISK
├─sda1             sda1    part  600M vfat        /boot/efi
├─sda2             sda2    part    2G xfs         /boot
└─sda3             sda3    part 17.4G LVM2_member
  ├─cs_centos-root dm-0    lvm  15.4G xfs         /
  └─cs_centos-swap dm-1    lvm     2G swap        [SWAP]
sr0                sr0     rom  1024M                         CD-ROM
nvme0n1            nvme0n1 disk   10G                         ORCL-VBOX-NVME-VER12 VB1234-56789
nvme0n2            nvme0n2 disk   10G                         ORCL-VBOX-NVME-VER12 VB1234-56789
===== NVME LIST =====
Node                  Generic               SN                   Model                                    Namespace  Usage                      Format           FW Rev
--------------------- --------------------- -------------------- ---------------------------------------- ---------- -------------------------- ---------------- --------
/dev/nvme0n1          /dev/ng0n1            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x1         10.74  GB /  10.74  GB    512   B +  0 B   1.0
/dev/nvme0n2          /dev/ng0n2            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x2         10.74  GB /  10.74  GB    512   B +  0 B   1.0
===== NVME SUBSYSTEMS =====
nvme-subsys0 - NQN=nqn.2014.08.org.nvmexpress:80ee80eeVB1234-56789        ORCL-VBOX-NVME-VER12
               hostnqn=nqn.2014-08.org.nvmexpress:uuid:f5aab8b7-89ca-42c4-bf97-f28a5b78aec0
               iopolicy=numa
\
 +- nvme0 pcie 0000:00:04.0 live

===== /dev/nvme0 SMART =====
Smart Log for NVME device:nvme0 namespace-id:ffffffff
critical_warning                        : 0
temperature                             : -273 °C (0 K, -459 °F)
available_spare                         : 0%
available_spare_threshold               : 0%
percentage_used                         : 0%
endurance group critical warning summary: 0
Data Units Read                         : 0 (0.00 B)
Data Units Written                      : 0 (0.00 B)
host_read_commands                      : 0
host_write_commands                     : 0
controller_busy_time                    : 0
power_cycles                            : 0
power_on_hours                          : 0
unsafe_shutdowns                        : 0
media_errors                            : 0
num_err_log_entries                     : 0
Warning Temperature Time                : 0
Critical Composite Temperature Time     : 0
Thermal Management T1 Trans Count       : 0
Thermal Management T2 Trans Count       : 0
Thermal Management T1 Total Time        : 0
Thermal Management T2 Total Time        : 0

===== /dev/nvme0 ERROR LOG =====
Error Log Entries for device:nvme0 entries:1
.................
 Entry[ 0]
.................
error_count     : 0
sqid            : 0
cmdid           : 0
status_field    : 0 (Successful Completion: The command completed without error)
phase_tag       : 0
parm_err_loc    : 0
lba             : 0
nsid            : 0
vs              : 0
trtype          : 0 (The transport type is not indicated or the error is not transport related)
csi             : 0
opcode          : 0
cs              : 0
trtype_spec_info: 0
log_page_version: 0
.................

===== PCI =====

===== KERNEL STORAGE LOGS =====
[Sat Sep 12 01:00:38 2026] acpi PNP0A08:00: _OSC: OS requested [PCIeHotplug PME AER PCIeCapability LTR DPC]
[Sat Sep 12 01:00:38 2026] acpi PNP0A08:00: _OSC: platform willing to grant [PCIeHotplug PME AER PCIeCapability LTR DPC]
[Sat Sep 12 01:00:38 2026] acpi PNP0A08:00: _OSC: platform retains control of PCIe features (AE_ERROR)
[Sat Sep 12 01:00:51 2026] SGI XFS with ACLs, security attributes, scrub, quota, no debug enabled
[Sat Sep 12 01:00:51 2026] XFS (dm-0): Mounting V5 Filesystem caed0bd4-535a-45d3-87b8-3727d59e929f
[Sat Sep 12 01:00:51 2026] XFS (dm-0): Starting recovery (logdev: internal)
[Sat Sep 12 01:00:51 2026] XFS (dm-0): Ending recovery (logdev: internal)
[Sat Sep 12 01:00:53 2026] nvme nvme0: pci function 0000:00:04.0
[Sat Sep 12 01:00:53 2026] nvme nvme0: 1/0/0 default/read/poll queues
[Sat Sep 12 01:00:53 2026] XFS (sda2): Mounting V5 Filesystem eaf8e197-6225-4931-b6e0-6a13c51efa89
[Sat Sep 12 01:00:53 2026] XFS (sda2): Ending clean mount
[Sat Sep 12 01:00:54 2026] nvme nvme0: using unchecked data buffer
[Sat Sep 12 01:00:54 2026] block nvme0n1: No UUID available providing old NGUID
```





```sh
degutos@centos:~$ ls -l /sys/class/nvme/
total 0
lrwxrwxrwx. 1 root root 0 Sep 12 01:00 nvme0 -> ../../devices/pci0000:00/0000:00:04.0/nvme/nvme0
```

You should see something along the lines of:

```
nvme0
```

Now you can trace the relationship:

```
PCIe device
    │
    │ 00:04.0
    ▼
NVMe controller
    │
    │ nvme0
    ▼
NVMe namespace
    │
    │ nvme0n1
    ▼
partition
    │
    ▼
filesystem
```


I'd actually run these **three commands together**:

```
lspci -nnk -s 00:04.0
nvme list
lsblk -o NAME,MODEL,SERIAL,SIZE,FSTYPE,MOUNTPOINTS
```

#### Important outputs

```sh
degutos@centos:~$ lspci -nnk -s 00:04.0
00:04.0 Non-Volatile memory controller [0108]: InnoTek Systemberatung GmbH Device [80ee:4e56]
        Kernel driver in use: nvme
        Kernel modules: nvme

degutos@centos:~$ nvme list
Node                  Generic               SN                   Model                                    Namespace  Usage                      Format           FW Rev
--------------------- --------------------- -------------------- ---------------------------------------- ---------- -------------------------- ---------------- --------
/dev/nvme0n1          /dev/ng0n1            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x1         10.74  GB /  10.74  GB    512   B +  0 B   1.0
/dev/nvme0n2          /dev/ng0n2            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x2         10.74  GB /  10.74  GB    512   B +  0 B   1.0

degutos@centos:~$ lsblk -o NAME,MODEL,SERIAL,SIZE,FSTYPE,MOUNTPOINTS
NAME               MODEL                SERIAL        SIZE FSTYPE      MOUNTPOINTS
sda                HARDDISK                            20G
├─sda1                                                600M vfat        /boot/efi
├─sda2                                                  2G xfs         /boot
└─sda3                                               17.4G LVM2_member
  ├─cs_centos-root                                   15.4G xfs         /
  └─cs_centos-swap                                      2G swap        [SWAP]
sr0                CD-ROM                            1024M
nvme0n1            ORCL-VBOX-NVME-VER12 VB1234-56789   10G
nvme0n2            ORCL-VBOX-NVME-VER12 VB1234-56789   10G
```


#### Summary

Your storage topology is:

```
PCI device
00:04.0
  │
  │ kernel driver = nvme
  ▼
NVMe controller
/dev/nvme0
  │
  ├── Namespace 1 → /dev/nvme0n1 → 10 GB
  │
  └── Namespace 2 → /dev/nvme0n2 → 10 GB
```


```sh
             00:04.0
          PCIe NVMe device
                │
                ▼
             nvme0
          NVMe controller
                │
          ┌─────┴─────┐
          ▼           ▼
      nvme0n1       nvme0n2
      NSID 1        NSID 2
       10 GB         10 GB
```


#### Now let's prove it through sysfs

Run:

```
readlink -f /sys/class/nvme/nvme0/device
```

```sh
degutos@centos:~$ readlink -f /sys/class/nvme/nvme0/device
/sys/devices/pci0000:00/0000:00:04.0
```
That proves:

```
nvme0 → PCI 00:04.0
```

then run:

```sh
degutos@centos:~$ ls -l /sys/class/nvme/
total 0
lrwxrwxrwx. 1 root root 0 Sep 12 13:14 nvme0 -> ../../devices/pci0000:00/0000:00:04.0/nvme/nvme0
```

and then run:

```sh
degutos@centos:~$ ls -l /sys/class/nvme/nvme0/
total 0
-r--r--r--. 1 root root 4096 Sep 12 01:00 address
-rw-r--r--. 1 root root 4096 Sep 12 13:26 admin_timeout
-r--r--r--. 1 root root 4096 Sep 12 13:26 cmb
-r--r--r--. 1 root root 4096 Sep 12 13:26 cmbloc
-r--r--r--. 1 root root 4096 Sep 12 13:26 cmbsz
-r--r--r--. 1 root root 4096 Sep 12 01:00 cntlid
-r--r--r--. 1 root root 4096 Sep 12 01:00 cntrltype
-r--r--r--. 1 root root 4096 Sep 12 01:00 dctype
-r--r--r--. 1 root root 4096 Sep 12 13:26 dev
lrwxrwxrwx. 1 root root    0 Sep 12 01:03 device -> ../../../0000:00:04.0
-r--r--r--. 1 root root 4096 Sep 12 01:00 firmware_rev
-rw-r--r--. 1 root root 4096 Sep 12 13:26 io_timeout
-r--r--r--. 1 root root 4096 Sep 12 13:26 kato
-r--r--r--. 1 root root 4096 Sep 12 01:00 model
drwxr-xr-x. 3 root root    0 Sep 12 01:00 ng0n1
drwxr-xr-x. 3 root root    0 Sep 12 01:00 ng0n2
-r--r--r--. 1 root root 4096 Sep 12 01:00 numa_node
drwxr-xr-x. 9 root root    0 Sep 12 01:00 nvme0n1
drwxr-xr-x. 9 root root    0 Sep 12 01:00 nvme0n2
-rw-r--r--. 1 root root 4096 Sep 12 13:26 passthru_err_log_enabled
drwxr-xr-x. 2 root root    0 Sep 12 13:26 power
-r--r--r--. 1 root root 4096 Sep 12 01:00 queue_count
-r--r--r--. 1 root root 4096 Sep 12 13:26 quirks
--w-------. 1 root root 4096 Sep 12 13:26 rescan_controller
--w-------. 1 root root 4096 Sep 12 13:26 reset_controller
-r--r--r--. 1 root root 4096 Sep 12 01:00 serial
-r--r--r--. 1 root root 4096 Sep 12 01:00 sqsize
-r--r--r--. 1 root root 4096 Sep 12 01:00 state
-r--r--r--. 1 root root 4096 Sep 12 01:00 subsysnqn
lrwxrwxrwx. 1 root root    0 Sep 12 01:00 subsystem -> ../../../../../class/nvme
-r--r--r--. 1 root root 4096 Sep 12 01:00 transport
-rw-r--r--. 1 root root 4096 Sep 12 01:00 uevent
```


You should see things including:

```
device
nvme0n1
nvme0n2
```

You can also query the namespace IDs directly:

```sh
degutos@centos:~$ cat /sys/class/nvme/nvme0/nvme0n1/nsid
1
degutos@centos:~$ cat /sys/class/nvme/nvme0/nvme0n2/nsid
2
```

So you've now established the full mapping without relying solely on `lsblk`:

```
PCI
 │
 └── 0000:00:04.0
          │
          │ nvme driver
          ▼
       /dev/nvme0
          │
          ├── NSID 1 → /dev/nvme0n1
          │
          └── NSID 2 → /dev/nvme0n2
```




### Debugging deeper on SRE part 

```sh
degutos@centos:~$ sudo nvme id-ctrl /dev/nvme0
[sudo] password for degutos:
NVME Identify Controller:
vid       : 0x80ee
ssvid     : 0x80ee
sn        : VB1234-56789
mn        : ORCL-VBOX-NVME-VER12
fr        : 1.0
rab       : 0
ieee      : 000000
cmic      : 0
mdts      : 0
cntlid    : 0
ver       : 0x10200
rtd3r     : 0x1
rtd3e     : 0x1
oaes      : 0
ctratt    : 0
rrls      : 0
bpcap     : 0
nssl      : 0
plsi      : 0
cntrltype : 0
fguid     : 00000000-0000-0000-0000-000000000000
crdt1     : 0
crdt2     : 0
crdt3     : 0
crcap     : 0
nvmsr     : 0
vwci      : 0
mec       : 0
oacs      : 0
acl       : 4
aerl      : 4
frmw      : 0x2
lpa       : 0
elpe      : 0
npss      : 0
avscc     : 0
apsta     : 0
wctemp    : 343
cctemp    : 343
mtfa      : 0
hmpre     : 0
hmmin     : 0
tnvmcap   : 0
unvmcap   : 0
rpmbs     : 0
edstt     : 0
dsto      : 0
fwug      : 0
kas       : 0
hctma     : 0
mntmt     : 0
mxtmt     : 0
sanicap   : 0
hmminds   : 0
hmmaxd    : 0
nsetidmax : 0
endgidmax : 0
anatt     : 0
anacap    : 0
anagrpmax : 0
nanagrpid : 0
pels      : 0
domainid  : 0
kpioc     : 0
mptfawr   : 0
megcap    : 0
tmpthha   : 0
cqt       : 0
sqes      : 0x66
cqes      : 0x44
maxcmd    : 0
nn        : 2
oncs      : 0
fuses     : 0
fna       : 0
vwc       : 0
awun      : 0
awupf     : 0
icsvscc   : 0
nwpc      : 0
acwu      : 0
ocfs      : 0
sgls      : 0
mnan      : 0
maxdna    : 0
maxcna    : 0
oaqd      : 0
rhiri     : 0
hirt      : 0
cmmrtd    : 0
nmmrtd    : 0
minmrtg   : 0
maxmrtg   : 0
trattr    : 0
mcudmq    : 0
mnsudmq   : 0
mcmr      : 0
nmcmr     : 0
mcdqpc    : 0
subnqn    :
ioccsz    : 0
iorcsz    : 0
icdoff    : 0
fcatt     : 0
msdbd     : 0
ofcs      : 0
ps      0 : mp:0.01W operational enlat:0 exlat:0 rrt:0 rrl:0
            rwt:0 rwl:0 idle_power:- active_power:-
            active_power_workload:-
            emergency power fail recovery time: -
            forced quiescence vault time: -
            emergency power fail vault time: -
```



```sh
degutos@centos:~$ sudo nvme id-ns /dev/nvme0n1
NVME Identify Namespace 1:
nsze    : 0x1400000
ncap    : 0x1400000
nuse    : 0x1400000
nsfeat  : 0
nlbaf   : 0
flbas   : 0
mc      : 0
dpc     : 0
dps     : 0
nmic    : 0
rescap  : 0
fpi     : 0
dlfeat  : 0
nawun   : 0
nawupf  : 0
nacwu   : 0
nabsn   : 0
nabo    : 0
nabspf  : 0
noiob   : 0
nvmcap  : 0
mssrl   : 0
mcl     : 0
msrc    : 0
kpios   : 0
nulbaf  : 0
kpiodaag: 0
anagrpid: 0
nsattr  : 0
nvmsetid: 0
endgid  : 0
nguid   : 55731efc946e9642bf4255f15f1344d0
eui64   : 0000000000000000
lbaf  0 : ms:0   lbads:9  rp:0 (in use)
```


```sh
degutos@centos:~$ sudo nvme id-ns /dev/nvme0n2
NVME Identify Namespace 2:
nsze    : 0x1400000
ncap    : 0x1400000
nuse    : 0x1400000
nsfeat  : 0
nlbaf   : 0
flbas   : 0
mc      : 0
dpc     : 0
dps     : 0
nmic    : 0
rescap  : 0
fpi     : 0
dlfeat  : 0
nawun   : 0
nawupf  : 0
nacwu   : 0
nabsn   : 0
nabo    : 0
nabspf  : 0
noiob   : 0
nvmcap  : 0
mssrl   : 0
mcl     : 0
msrc    : 0
kpios   : 0
nulbaf  : 0
kpiodaag: 0
anagrpid: 0
nsattr  : 0
nvmsetid: 0
endgid  : 0
nguid   : c6e9e54d1b91ec47a4dcb8f8162505fb
eui64   : 0000000000000000
lbaf  0 : ms:0   lbads:9  rp:0 (in use)
```


```sh
degutos@centos:~$ sudo nvme smart-log /dev/nvme0
Smart Log for NVME device:nvme0 namespace-id:ffffffff
critical_warning                        : 0
temperature                             : -273 °C (0 K, -459 °F)
available_spare                         : 0%
available_spare_threshold               : 0%
percentage_used                         : 0%
endurance group critical warning summary: 0
Data Units Read                         : 0 (0.00 B)
Data Units Written                      : 0 (0.00 B)
host_read_commands                      : 0
host_write_commands                     : 0
controller_busy_time                    : 0
power_cycles                            : 0
power_on_hours                          : 0
unsafe_shutdowns                        : 0
media_errors                            : 0
num_err_log_entries                     : 0
Warning Temperature Time                : 0
Critical Composite Temperature Time     : 0
Thermal Management T1 Trans Count       : 0
Thermal Management T2 Trans Count       : 0
Thermal Management T1 Total Time        : 0
Thermal Management T2 Total Time        : 0
```




### LAB building a filesystem XFS and EXT4 on NVMe


```sh
degutos@centos:~$ lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0   20G  0 disk
├─sda1               8:1    0  600M  0 part /boot/efi
├─sda2               8:2    0    2G  0 part /boot
└─sda3               8:3    0 17.4G  0 part
  ├─cs_centos-root 253:0    0 15.4G  0 lvm  /
  └─cs_centos-swap 253:1    0    2G  0 lvm  [SWAP]
sr0                 11:0    1 1024M  0 rom
nvme0n1            259:0    0   10G  0 disk
nvme0n2            259:1    0   10G  0 disk
```


```sh
degutos@centos:~$ sudo mkfs.xfs /dev/nvme0n1
[sudo] password for degutos:
meta-data=/dev/nvme0n1           isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
```


Checking the filesystem

```sh
degutos@centos:~$ lsblk -e7 -o NAME,KNAME,TYPE,SIZE,FSTYPE,FSVER,MOUNTPOINTS,UUID,MODEL,SERIAL | grep nvme
nvme0n1            nvme0n1 disk   10G xfs                              600c51cc-91fb-40d7-828f-39df017574c3   ORCL-VBOX-NVME-VER12 VB1234-56789
nvme0n2            nvme0n2 disk   10G                                                                         ORCL-VBOX-NVME-VER12 VB1234-56789
```


#### Mount 

```sh
degutos@centos:~$ sudo mkdir /mnt/xfs-test
degutos@centos:~$

degutos@centos:~$ sudo mount /dev/nvme0n1 /mnt/xfs-test
degutos@centos:~$
```

```sh
degutos@centos:~$ df -hT /mnt/xfs-test
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/nvme0n1   xfs    10G  228M  9.8G   3% /mnt/xfs-test
```


lets check all volumes mounted:
```sh
degutos@centos:~$ df -hT
Filesystem                 Type      Size  Used Avail Use% Mounted on
/dev/mapper/cs_centos-root xfs        16G  4.4G   12G  29% /
devtmpfs                   devtmpfs  757M     0  757M   0% /dev
tmpfs                      tmpfs     784M     0  784M   0% /dev/shm
efivarfs                   efivarfs  256K  8.2K  248K   4% /sys/firmware/efi/efivars
tmpfs                      tmpfs     314M  6.4M  308M   3% /run
tmpfs                      tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                  xfs       2.0G  294M  1.7G  15% /boot
/dev/sda1                  vfat      599M   13M  586M   3% /boot/efi
tmpfs                      tmpfs     157M  116K  157M   1% /run/user/1000
/dev/nvme0n1               xfs        10G  228M  9.8G   3% /mnt/xfs-test
```

##### findmnt

```sh
degutos@centos:~$ findmnt /mnt/xfs-test
TARGET        SOURCE       FSTYPE OPTIONS
/mnt/xfs-test /dev/nvme0n1 xfs    rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota
```

OR

```sh
degutos@centos:~$ findmnt /dev/nvme0n1
TARGET        SOURCE       FSTYPE OPTIONS
/mnt/xfs-test /dev/nvme0n1 xfs    rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota
```

##### xfs_info

```sh
degutos@centos:~$ xfs_info /mnt/xfs-test
meta-data=/dev/nvme0n1           isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
```


##### Creating some data

```sh
dd if=/dev/zero of=/mnt/xfs-test/testfile bs=1M count=100 status=progress
```


```sh
degutos@centos:~$ sudo dd if=/dev/zero of=/mnt/xfs-test/testfile bs=1M count=100 status=progress
[sudo] password for degutos:
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.0324139 s, 3.2 GB/s

degutos@centos:~$ sync

degutos@centos:~$ df -h
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/cs_centos-root   16G  4.4G   12G  29% /
devtmpfs                    757M     0  757M   0% /dev
tmpfs                       784M     0  784M   0% /dev/shm
efivarfs                    256K  8.2K  248K   4% /sys/firmware/efi/efivars
tmpfs                       314M  6.4M  308M   3% /run
tmpfs                       1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                   2.0G  294M  1.7G  15% /boot
/dev/sda1                   599M   13M  586M   3% /boot/efi
tmpfs                       157M  116K  157M   1% /run/user/1000
/dev/nvme0n1                 10G  328M  9.7G   4% /mnt/xfs-test
```


#### Troubleshooting XFS

```sh
degutos@centos:~$ sudo xfs_info /dev/nvme0n1
meta-data=/dev/nvme0n1           isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
```


Unmount the volume:

```sh
degutos@centos:~$ sudo umount /mnt/xfs-test
```


Then perform a **read-only/dry-run** check:

```sh
degutos@centos:~$ sudo xfs_repair -n /dev/nvme0n1
Phase 1 - find and verify superblock...
Phase 2 - using internal log
        - zero log...
        - scan filesystem freespace and inode maps...
        - found root inode chunk
Phase 3 - for each AG...
        - scan (but don't clear) agi unlinked lists...
        - process known inodes and perform inode discovery...
        - agno = 0
        - agno = 1
        - agno = 2
        - agno = 3
        - process newly discovered inodes...
Phase 4 - check for duplicate blocks...
        - setting up duplicate extent list...
        - check for inodes claiming duplicate blocks...
        - agno = 0
        - agno = 1
        - agno = 2
        - agno = 3
No modify flag set, skipping phase 5
Phase 6 - check inode connectivity...
        - traversing filesystem ...
        - traversal finished ...
        - moving disconnected inodes to lost+found ...
Phase 7 - verify link counts...
No modify flag set, skipping filesystem flush and exiting.
```

The `-n` option is a dry-run is not going to repair anything 


#### Troubleshooting and creating a partition EXT4

```sh
degutos@centos:~$ sudo mkfs.ext4 /dev/nvme0n2
[sudo] password for degutos:
mke2fs 1.47.1 (20-May-2024)
Creating filesystem with 2621440 4k blocks and 655360 inodes
Filesystem UUID: a28bcf60-1c3b-49e6-86a4-78818a5fbb49
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

Mounting volume

```sh
degutos@centos:~$ sudo mkdir /mnt/ext4-test
degutos@centos:~$ sudo mount /dev/nvme0n2 /mnt/ext4-test
degutos@centos:~$
```


```sh
degutos@centos:~$ findmnt /mnt/ext4-test
TARGET         SOURCE       FSTYPE OPTIONS
/mnt/ext4-test /dev/nvme0n2 ext4   rw,relatime,seclabel
```


```sh
degutos@centos:~$ sudo e2fsck -n /dev/nvme0n2
e2fsck 1.47.1 (20-May-2024)
/dev/nvme0n2: clean, 11/655360 files, 66753/2621440 blocks
```

Red Hat's storage documentation recommends `e2fsck` for ext2/3/4 and `xfs_repair` for XFS.

**Important SRE lesson:** filesystem repair tools fix filesystem metadata consistency; they don't magically repair a broken disk/controller. Red Hat explicitly calls this distinction out


### Adding LVM - Logical volume management


This is worth doing because real Linux storage stacks often look more like:

```
NVMe
 ↓
partition
 ↓
LVM PV
 ↓
VG
 ↓
LV
 ↓
filesystem
 ↓
mount
```


```sh
degutos@centos:~$ nvme list
Node                  Generic               SN                   Model                                    Namespace  Usage                      Format           FW Rev
--------------------- --------------------- -------------------- ---------------------------------------- ---------- -------------------------- ---------------- --------
/dev/nvme0n1          /dev/ng0n1            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x1         10.74  GB /  10.74  GB    512   B +  0 B   1.0
/dev/nvme0n2          /dev/ng0n2            VB1234-56789         ORCL-VBOX-NVME-VER12                     0x2         10.74  GB /  10.74  GB    512   B +  0 B   1.0
```


Create a partition:

```
sudo fdisk /dev/nvme0n1
```


then
```sh
sudo partprobe /dev/nvme0n1
```


```sh
degutos@centos:~$ lsblk -e7 -o NAME,KNAME,TYPE,SIZE,FSTYPE,FSVER,MOUNTPOINTS,UUID,MODEL,SERIAL | grep nvme
nvme0n1            nvme0n1 disk   10G                                                                         ORCL-VBOX-NVME-VER12 VB1234-56789
nvme0n2            nvme0n2 disk   10G ext4        1.0                  a28bcf60-1c3b-49e6-86a4-78818a5fbb49   ORCL-VBOX-NVME-VER12 VB1234-56789
```



##### Create a physical volume PV and Volume group VG

```sh
degutos@centos:~$ sudo pvcreate /dev/nvme0n1
WARNING: dos signature detected on /dev/nvme0n1 at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/nvme0n1.
  Physical volume "/dev/nvme0n1" successfully created.
```


```sh
degutos@centos:~$ sudo vgcreate labvg /dev/nvme0n1
  Volume group "labvg" successfully created
```


```sh
degutos@centos:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/nvme0n1
  VG Name               labvg
  PV Size               10.00 GiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               2559
  Allocated PE          0
  PV UUID               8h9zcf-Ev6m-CtoA-CdVo-qzQ9-0IPM-iYcHF1
```


```sh
degutos@centos:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               labvg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       0 / 0
  Free  PE / Size       2559 / <10.00 GiB
  VG UUID               FfRMro-moPT-f5ET-VBsP-lniZ-T9SV-qFmyvx
```


##### Create a Logical Volume LV

```sh
degutos@centos:~$ sudo lvcreate -n data -L 5G labvg
  Logical volume "data" created.
```


```sh
degutos@centos:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/labvg/data
  LV Name                data
  VG Name                labvg
  LV UUID                DbUEHt-YOXW-I8KZ-9O5z-9OxG-vB5Y-3UHXq9
  LV Write Access        read/write
  LV Creation host, time centos, 2026-09-12 14:55:38 +0100
  LV Status              available
  # open                 0
  LV Size                5.00 GiB
  Current LE             1280
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```



##### Formating and Mounting 


```sh
degutos@centos:~$ sudo mkfs.xfs /dev/labvg/data
meta-data=/dev/labvg/data        isize=512    agcount=4, agsize=327680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=1310720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
```


```sh
degutos@centos:~$ sudo mkdir /mnt/lvm-test
degutos@centos:~$ sudo mount /dev/labvg/data /mnt/lvm-test/
degutos@centos:~$
```


##### PVS VGS and LVS

```sh
degutos@centos:~$ sudo pvs
  PV           VG        Fmt  Attr PSize   PFree
  /dev/nvme0n1 labvg     lvm2 a--  <10.00g <5.00g
  /dev/sda3    cs_centos lvm2 a--   17.41g     0

degutos@centos:~$ sudo vgs
  VG        #PV #LV #SN Attr    VSize   VFree
  cs_centos   1   2   0 wz--n--  17.41g     0
  labvg       1   1   0 wz--n-- <10.00g <5.00g

degutos@centos:~$ sudo lvs
  LV   VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root cs_centos -wi-ao---- 15.41g
  swap cs_centos -wi-ao----  2.00g
  data labvg     -wi-ao----  5.00g
```


```sh
degutos@centos:~$ df -hT
Filesystem                 Type      Size  Used Avail Use% Mounted on
/dev/mapper/cs_centos-root xfs        16G  4.4G   12G  29% /
devtmpfs                   devtmpfs  757M     0  757M   0% /dev
tmpfs                      tmpfs     784M     0  784M   0% /dev/shm
efivarfs                   efivarfs  256K  8.2K  248K   4% /sys/firmware/efi/efivars
tmpfs                      tmpfs     314M  6.4M  308M   3% /run
tmpfs                      tmpfs     1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                  xfs       2.0G  294M  1.7G  15% /boot
/dev/sda1                  vfat      599M   13M  586M   3% /boot/efi
tmpfs                      tmpfs     157M  116K  157M   1% /run/user/1000
/dev/mapper/labvg-data     xfs       5.0G  130M  4.9G   3% /mnt/lvm-test
```



### Performance troubleshooting


Install/use `fio`:

```sh
$ sudo dnf install fio
```

Also

```sh
$ sudo dnf install sysstat
$ sudo dnf iotop-c
```


#### FIO

```sh
degutos@centos:/mnt/xfs-test$ sudo fio   --name=randread   --filename=/mnt/xfs-test/testfile   --size=1G   --rw=randread   --bs=4k   --iodepth=32   --numjobs=2   --ioengine=libaio   --runtime=30   --time_based
```


Then watch the disk from another terminal:

```
iostat -xz 1
```

and after `fio` finishes, you'll get useful metrics such as **IOPS, bandwidth, average latency, and latency percentiles**.

```sh
degutos@centos:/mnt/xfs-test$ sudo fio   --name=randread   --filename=/mnt/xfs-test/testfile   --size=1G   --rw=randread   --bs=4k   --iodepth=32   --numjobs=2   --ioengine=libaio   --runtime=30   --time_based
randread: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
...
fio-3.36
Starting 2 processes
Jobs: 2 (f=2): [r(2)][100.0%][r=59.9MiB/s][r=15.3k IOPS][eta 00m:00s]
randread: (groupid=0, jobs=1): err= 0: pid=6834: Sat Sep 12 15:17:49 2026
  read: IOPS=9615, BW=37.6MiB/s (39.4MB/s)(1127MiB/30001msec)
    slat (nsec): min=541, max=2336.2k, avg=103299.71, stdev=64356.60
    clat (nsec): min=1708, max=9561.7k, avg=3222158.65, stdev=680550.07
     lat (usec): min=144, max=9715, avg=3325.46, stdev=699.40
    clat percentiles (usec):
     |  1.00th=[ 1926],  5.00th=[ 2245], 10.00th=[ 2409], 20.00th=[ 2638],
     | 30.00th=[ 2802], 40.00th=[ 2966], 50.00th=[ 3097], 60.00th=[ 3294],
     | 70.00th=[ 3556], 80.00th=[ 3916], 90.00th=[ 4228], 95.00th=[ 4359],
     | 99.00th=[ 4621], 99.50th=[ 4686], 99.90th=[ 5145], 99.95th=[ 5735],
     | 99.99th=[ 8979]
   bw (  KiB/s): min=26872, max=46696, per=50.19%, avg=38607.42, stdev=6497.50, samples=59
   iops        : min= 6718, max=11674, avg=9651.83, stdev=1624.39, samples=59
  lat (usec)   : 2=0.01%, 250=0.01%, 500=0.01%, 750=0.01%, 1000=0.01%
  lat (msec)   : 2=1.73%, 4=80.51%, 10=17.75%
  cpu          : usr=1.16%, sys=6.56%, ctx=209500, majf=0, minf=45
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=288472,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32
randread: (groupid=0, jobs=1): err= 0: pid=6835: Sat Sep 12 15:17:49 2026
  read: IOPS=9615, BW=37.6MiB/s (39.4MB/s)(1127MiB/30001msec)
    slat (nsec): min=541, max=2371.1k, avg=103190.64, stdev=64343.75
    clat (nsec): min=1125, max=39287k, avg=3222587.29, stdev=774552.52
     lat (usec): min=157, max=39393, avg=3325.78, stdev=791.22
    clat percentiles (usec):
     |  1.00th=[ 1942],  5.00th=[ 2245], 10.00th=[ 2409], 20.00th=[ 2638],
     | 30.00th=[ 2802], 40.00th=[ 2933], 50.00th=[ 3097], 60.00th=[ 3294],
     | 70.00th=[ 3556], 80.00th=[ 3916], 90.00th=[ 4228], 95.00th=[ 4359],
     | 99.00th=[ 4621], 99.50th=[ 4686], 99.90th=[ 5145], 99.95th=[ 6063],
     | 99.99th=[37487]
   bw (  KiB/s): min=27209, max=46952, per=50.19%, avg=38610.93, stdev=6496.01, samples=59
   iops        : min= 6802, max=11738, avg=9652.71, stdev=1624.00, samples=59
  lat (usec)   : 2=0.01%, 250=0.01%, 500=0.01%, 750=0.01%, 1000=0.01%
  lat (msec)   : 2=1.52%, 4=80.44%, 10=18.02%, 50=0.01%
  cpu          : usr=0.76%, sys=7.08%, ctx=209250, majf=0, minf=45
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=288486,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=75.1MiB/s (78.8MB/s), 37.6MiB/s-37.6MiB/s (39.4MB/s-39.4MB/s), io=2254MiB (2363MB), run=30001-30001msec

Disk stats (read/write):
    dm-0: ios=417237/7, sectors=3338280/50, merge=0/0, ticks=36730/0, in_queue=36730, util=99.56%, aggrios=418745/17, aggsectors=3350344/2130, aggrmerge=0/250, aggrticks=52715/20, aggrin_queue=52736, aggrutil=23.88%
  sda: ios=418745/17, sectors=3350344/2130, merge=0/250, ticks=52715/20, in_queue=52736, util=23.88%
```


OR lets test the NVMe running on LVM partition

```sh
degutos@centos:/mnt/xfs-test$ sudo fio \
  --name=randread \
  --filename=/mnt/lvm-test/testfile \
  --size=2G \
  --rw=randread \
  --bs=4k \
  --iodepth=32 \
  --numjobs=2 \
  --ioengine=libaio \
  --runtime=30 \
  --time_based
[sudo] password for degutos:
randread: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
...
fio-3.36
Starting 2 processes
randread: Laying out IO file (1 file / 2048MiB)
Jobs: 2 (f=2): [r(2)][100.0%][r=51.3MiB/s][r=13.1k IOPS][eta 00m:00s]
randread: (groupid=0, jobs=1): err= 0: pid=11119: Mon Sep 14 06:08:38 2026
  read: IOPS=7038, BW=27.5MiB/s (28.8MB/s)(825MiB/30001msec)
    slat (nsec): min=708, max=17705k, avg=141062.87, stdev=79706.68
    clat (nsec): min=1375, max=39335k, avg=4404169.23, stdev=649618.78
     lat (usec): min=141, max=39498, avg=4545.23, stdev=662.18
    clat percentiles (usec):
     |  1.00th=[ 3425],  5.00th=[ 3654], 10.00th=[ 3818], 20.00th=[ 3982],
     | 30.00th=[ 4113], 40.00th=[ 4228], 50.00th=[ 4359], 60.00th=[ 4490],
     | 70.00th=[ 4621], 80.00th=[ 4752], 90.00th=[ 5014], 95.00th=[ 5211],
     | 99.00th=[ 5997], 99.50th=[ 6259], 99.90th=[ 8356], 99.95th=[10552],
     | 99.99th=[27132]
   bw (  KiB/s): min=25152, max=29672, per=50.10%, avg=28199.46, stdev=1227.27, samples=59
   iops        : min= 6288, max= 7418, avg=7049.90, stdev=306.75, samples=59
  lat (usec)   : 2=0.01%, 250=0.01%, 500=0.01%, 750=0.01%, 1000=0.01%
  lat (msec)   : 2=0.01%, 4=20.46%, 10=79.46%, 20=0.05%, 50=0.02%
  cpu          : usr=1.18%, sys=6.48%, ctx=189321, majf=0, minf=45
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=211155,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32
randread: (groupid=0, jobs=1): err= 0: pid=11120: Mon Sep 14 06:08:38 2026
  read: IOPS=7032, BW=27.5MiB/s (28.8MB/s)(824MiB/30001msec)
    slat (nsec): min=875, max=22876k, avg=141179.57, stdev=83717.26
    clat (nsec): min=1875, max=40261k, avg=4407659.11, stdev=648440.92
     lat (usec): min=252, max=40387, avg=4548.84, stdev=661.13
    clat percentiles (usec):
     |  1.00th=[ 3392],  5.00th=[ 3687], 10.00th=[ 3818], 20.00th=[ 3982],
     | 30.00th=[ 4146], 40.00th=[ 4228], 50.00th=[ 4359], 60.00th=[ 4490],
     | 70.00th=[ 4621], 80.00th=[ 4752], 90.00th=[ 5014], 95.00th=[ 5276],
     | 99.00th=[ 5997], 99.50th=[ 6325], 99.90th=[ 8356], 99.95th=[ 9634],
     | 99.99th=[27657]
   bw (  KiB/s): min=25144, max=29776, per=50.07%, avg=28182.64, stdev=1266.81, samples=59
   iops        : min= 6286, max= 7444, avg=7045.66, stdev=316.70, samples=59
  lat (usec)   : 2=0.01%, 500=0.01%, 750=0.01%, 1000=0.01%
  lat (msec)   : 2=0.01%, 4=20.17%, 10=79.77%, 20=0.03%, 50=0.01%
  cpu          : usr=0.83%, sys=6.81%, ctx=189286, majf=0, minf=45
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=210989,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=55.0MiB/s (57.6MB/s), 27.5MiB/s-27.5MiB/s (28.8MB/s-28.8MB/s), io=1649MiB (1729MB), run=30001-30001msec

Disk stats (read/write):
    dm-2: ios=377782/2, sectors=3022256/34, merge=0/0, ticks=49070/0, in_queue=49070, util=99.27%, aggrios=378585/2, aggsectors=3028680/34, aggrmerge=0/0, aggrticks=51851/0, aggrin_queue=51852, aggrutil=99.06%
  nvme0n1: ios=378585/2, sectors=3028680/34, merge=0/0, ticks=51851/0, in_queue=51852, util=99.06%
```


Lets notice the important things:

Last line:  util=99.06%

Also notice the IOPs

```sh
read: IOPS=7038
read: IOPS=7032
```

since we have 02 jobs we need to look at those 02 IOPS

Two jobs:

```
7038 + 7032 ≈ 14,070 IOPS
```

So our IOPS is 14k IOPS

and also look at this

```sh
Run status group 0 (all jobs):
   READ: bw=55.0MiB/s (57.6MB/s), 27.5MiB/s-27.5MiB/s (28.8MB/s-28.8MB/s), io=1649MiB (1729MB), run=30001-30001msec
```

and also

```sh
 read: IOPS=7032, BW=27.5MiB/s (28.8MB/s)(824MiB/30001msec)
```


Import to check

```sh
err= 0
```

Takeaway:

```
Workload:
  4K random read
  2 jobs × QD32
  30 seconds

Result:
  ~14K IOPS
  55 MiB/s
  ~4.55 ms average latency
  ~6.0 ms P99 latency
  ~8.36 ms P99.9 latency
  ~27 ms P99.99 latency
  0 I/O errors

Device:
  /dev/nvme0n1
  NVMe utilization ~99%
  Backed by LVM LV labvg-data
  Mounted as XFS at /mnt/lvm-test
```


And run again:

```
sudo nvme smart-log /dev/nvme0
sudo nvme error-log /dev/nvme0
dmesg -T | grep -Ei 'nvme|pcie|aer|error|timeout|reset'
```



### Smartctl 

Smartctl is worth learning too
On modern systems you'll commonly use `nvme-cli` for NVMe-specific information, while `smartctl` is another useful interface.



```
degutos@centos:/mnt/xfs-test$ sudo smartctl -a /dev/nvme0
smartctl 7.4 2023-08-01 r5530 [aarch64-linux-6.12.0-266.el10.aarch64] (local build)
Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Number:                       ORCL-VBOX-NVME-VER12
Serial Number:                      VB1234-56789
Firmware Version:                   1.0
PCI Vendor/Subsystem ID:            0x80ee
IEEE OUI Identifier:                0x000000
Controller ID:                      0
NVMe Version:                       1.2
Number of Namespaces:               2
Local Time is:                      Mon Sep 14 06:34:19 2026 IST
Firmware Updates (0x02):            1 Slot
Warning  Comp. Temp. Threshold:     70 Celsius
Critical Comp. Temp. Threshold:     70 Celsius

Supported Power States
St Op     Max   Active     Idle   RL RT WL WT  Ent_Lat  Ex_Lat
 0 +     0.01W       -        -    0  0  0  0        0       0

=== START OF SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

SMART/Health Information (NVMe Log 0x02)
Critical Warning:                   0x00
Temperature:                        -
Available Spare:                    0%
Available Spare Threshold:          0%
Percentage Used:                    0%
Data Units Read:                    0
Data Units Written:                 0
Host Read Commands:                 0
Host Write Commands:                0
Controller Busy Time:               0
Power Cycles:                       0
Power On Hours:                     0
Unsafe Shutdowns:                   0
Media and Data Integrity Errors:    0
Error Information Log Entries:      0
Warning  Comp. Temperature Time:    0
Critical Comp. Temperature Time:    0

Error Information (NVMe Log 0x01, 1 of 1 entries)
No Errors Logged

Self-tests not supported

degutos@centos:/mnt/xfs-test$
```



### IPMITOOL

`ipmitool` talks to an IPMI/BMC interface. Your VirtualBox VM doesn't suddenly acquire a physical server BMC

##### install

```sh
sudo dnf install ipmitool
```

On a physical server, the architecture is roughly:

```
Linux
  │
  ├── /dev/ipmi0
  │       │
  │       ↓
  │     IPMI driver
  │       │
  │       ↓
  │     BMC
  │       │
  │       ├── sensors
  │       ├── chassis
  │       ├── SEL/event log
  │       └── hardware inventory
  │
  └── NVMe driver → NVMe SSD
```


VirtualBox VM looks more like:

```
Linux VM
  │
  ├── nvme driver → VirtualBox NVMe controller → virtual disk
  │
  └── /dev/ipmi0  ❌ doesn't exist
```


See, we don't have that:
```sh
degutos@centos:/mnt/xfs-test$ ls -l /dev/ipmi*
ls: cannot access '/dev/ipmi*': No such file or directory
```


OR

```sh
degutos@centos:/mnt/xfs-test$ lsmod | grep ipmi
degutos@centos:/mnt/xfs-test$

degutos@centos:/mnt/xfs-test$ sudo dmesg | grep -i ipmi
degutos@centos:/mnt/xfs-test$
```


```sh
degutos@centos:/mnt/xfs-test$ sudo ipmitool mc info
Could not open device at /dev/ipmi0 or /dev/ipmi/0 or /dev/ipmidev/0: No such file or directory

degutos@centos:/mnt/xfs-test$ sudo ipmitool chassis status
Could not open device at /dev/ipmi0 or /dev/ipmi/0 or /dev/ipmidev/0: No such file or directory
```

VirtualBox VM doesn't magically get a BMC just because you installed `ipmitool`.



### LAB IPMI and Redfish


#### VM2: bmc-lab
-------------------

CentOS Stream 9
192.168.56.20

        ┌──────────────────────────┐
        │       BMC simulator      │
        │                          │
        │  OpenIPMI / ipmi_sim     │
        │          │               │
        │        IPMI              │
        │                          │
        │  DMTF Redfish Emulator   │
        │          │               │
        │       Redfish            │
        └────────────┬─────────────┘
                     │
              Host-only network
                     │
              192.168.56.0/24
      



```sh
sudo dnf install -y \
    git \
    python3 \
    python3-pip \
    python3-devel \
    gcc \
    make \
    curl \
    wget \
    ipmitool \
    OpenIPMI
```


Lets check versions:

```sh
degutos@centos:/mnt/xfs-test$ python3 --version
Python 3.12.14

degutos@centos:/mnt/xfs-test$ ipmitool -V
ipmitool version 1.8.19
```


Also install:

```sh
 sudo dnf install -y OpenIPMI-lanserv
```

```sh
degutos@centos:/mnt/xfs-test$ which ipmi_sim
/usr/bin/ipmi_sim
```


See what config file we have:

```sh
degutos@centos:/mnt/xfs-test$ rpm -ql OpenIPMI-lanserv
/etc/ipmi
/etc/ipmi/ipmisim1.emu
/etc/ipmi/lan.conf
/usr/bin/ipmi_sim
/usr/bin/ipmilan
/usr/bin/sdrcomp
/usr/lib/.build-id
/usr/lib/.build-id/17
/usr/lib/.build-id/17/f534011ec65247588d80f9fe3c9c0f24a41418
/usr/lib/.build-id/3e
/usr/lib/.build-id/3e/46b338e80c7df530ba129bea9713b7d3b7e187
/usr/lib/.build-id/ae
/usr/lib/.build-id/ae/7b685abc108f8e2515078767b2dcc07ceb38a9
/usr/lib/.build-id/db/82ce8e63ce9781281ee50f071a9ac96c7e1fa9
/usr/lib64/libIPMIlanserv.so.0
/usr/lib64/libIPMIlanserv.so.0.0.1
/usr/share/man/man1/ipmi_sim.1.gz
/usr/share/man/man5/ipmi_lan.5.gz
/usr/share/man/man5/ipmi_sim_cmd.5.gz
/usr/share/man/man8/ipmilan.8.gz
```


```sh
degutos@centos:/mnt/xfs-test$ rpm -ql OpenIPMI-lanserv | grep -E '/(lan|sim|emu|conf|examples?)|ipmi_sim'
/etc/ipmi/lan.conf
/usr/bin/ipmi_sim
/usr/share/man/man1/ipmi_sim.1.gz
/usr/share/man/man5/ipmi_sim_cmd.5.gz
```

