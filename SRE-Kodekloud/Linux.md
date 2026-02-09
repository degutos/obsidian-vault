

## Basic Linux nice commands


### Check your current shell

```sh
➜  ~ echo $SHELL
/bin/zsh
```


### Create dir's and sub-dir's in one shot

```sh
mkdir -p /tmp/asia/india/bangalore
```


### Creating a file using cat and CTRL+D

```sh
➜  ~ cat > test.txt
this is a test
CTRL + D

➜  ~ cat test.txt
this is a test
```


### To see default EDITOR on Ubuntu

```sh
$ update-alternatives --display editor

# use --get-selections to see all
$ update-alternatives --get-selections
```


```sh
# check my user name
➜  ~ whoami
degutos

# check user id and groups
➜  ~ id
uid=1000(degutos) gid=1000(degutos) groups=1000(degutos),4(adm),20(dialout),24(cdrom),27(sudo),29(audio),44(video),46(plugdev),60(games),100(users),102(input),105(render),110(netdev),115(lpadmin),993(gpio),994(i2c),995(spi)

```


### Download files

```sh
curl https://www.some-site.com/some-file.txt -O
wget https://www.some-site.com/some-file.txt -O some-file.txt

# Example:
thor@host01 ~$ curl https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf -O
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13264  100 13264    0     0   3689      0  0:00:03  0:00:03 --:--:--  3688

thor@host01 ~$ wget https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf -O dummy.pdf
--2025-05-29 18:42:19--  https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf
Resolving www.w3.org (www.w3.org)... 104.18.23.19, 104.18.22.19, 2606:4700::6812:1613, ...
Connecting to www.w3.org (www.w3.org)|104.18.23.19|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13264 (13K) [application/pdf]
Saving to: ‘dummy.pdf’

dummy.pdf                  100%[========================================>]  12.95K  --.-KB/s    in 0s      

2025-05-29 18:42:25 (63.1 MB/s) - ‘dummy.pdf’ saved [13264/13264]

```



### check os version

```sh
➜  ~ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```


### RPM

```sh
rpm -i telnet.rpm # Install a package
rpm -e telnet # Uninstall a package
rpm -q telnet # query a package

# query packages
~$ rpm -q python3 telnet ansible openssh-server
python3-3.9.21-1.el9.x86_64
telnet-0.17-85.el9.x86_64
package ansible is not installed
openssh-server-8.7p1-44.el9.x86_64

# install package with RPM
$ sudo rpm -ivh ftp-0.17-89.el9.x86_64.rpm 
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:ftp-0.17-89.el9                  ################################# [100%]

# uninstall a package
$ sudo rpm -e ftp-0.17-89.el9.x86_64
thor@host01 /opt$ 
```

### YUM Repos

```sh
# check entire list of configured repository
yum repolist

ls /etc/yum.repos.d/
CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo  CentOS-aarch64-kernel.repo  CentOS-fasttrack.repo


# installing a package
$ sudo yum install -y maven
CentOS Stream 9 - BaseOS                                                    4.5 MB/s | 8.7 MB     00:01    
CentOS Stream 9 - AppStream                                                  11 MB/s |  24 MB     00:02    
CentOS Stream 9 - Extras packages                                           2.5 kB/s |  19 kB     00:07    
Extra Packages for Enterprise Linux 9 - x86_64                               12 MB/s |  20 MB     00:01    
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64        2.1 kB/s | 2.5 kB     00:01    
Extra Packages for Enterprise Linux 9 - Next - x86_64                       269 kB/s | 414 kB     00:01    
Dependencies resolved.
============================================================================================================
 Package                                       Architecture Version                   Repository       Size
============================================================================================================
Installing:
 maven                                         noarch       1:3.6.3-22.el9            appstream        18 k
Installing dependencies:
 apache-commons-cli             
 ...


# check a version of a package
$ yum list maven
Last metadata expiration check: 0:00:20 ago on Thu May 29 19:29:30 2025.
Installed Packages
maven.noarch                                    1:3.6.3-22.el9                                    @appstream

# Remove a package
$ sudo yum remove maven

# install a package on specific version
$ sudo yum install -y maven-1:3.6.3-19.el9.noarch


```



### Services

```sh
systemctl status sshd
systemctl start sshd
systemctl stop sshd
systemctl enable sshd

```


How to create a new service

```sh
vi /etc/systemd/system/my_app.service

[Unit]
Description=My python web application 

[service]
ExecStart=/usr/bin/python3 /opt/code/my_app.py
ExecStartPre=/opt/code/configure_db.sh
ExecStartPost=/opt/code/email_status.sh
Restart=always

[Install]
WantedBy=multi-user.target
```


```sh
systemctl daemon-reload
systemctl start my_app

systemctl status my_app
systemctl enable my_app
```

```sh
    1  systemctl status httpd
    2  sudo systemctl status httpd
    3  sudo systemctl start httpd
    4  sudo systemctl status httpd
    5  systemctl is-enabled httpd
    6  sudo systemctl enable httpd
    7  systemctl is-enabled httpd
    8  systemctl status httpd
    9  sudo systemctl stop httpd
   10  systemctl status httpd
   11  sudo systemctl disable httpd
   12  systemctl status httpd
   13  cd /usr/lib/systemd/
   14  ls
   15  cd system/
   16  ls
   17  pwd
   18  systemctl status app.service
   19  vi app.service 
   20  sudo systemctl start app
   21  systemctl status app
   22  sudo systemctl enable app
```



## VIM


## Network troubleshooting


### Tools to troubleshoot DNS issue

- nslookup
- dig



## Storage

### Disk partitions


- Blockstorage
- SSD (scsi disk number 8 in lsblk MAJ)

To see all volumes details:

```sh
$ lsblk
```

OR

```sh
$ ls -l /dev/ | grep "^b"
```

OR

```sh
sudo fdisk -l /dev/sda
```

OR

```sh
gdisk /dev/sdb
```


Types of partition:

- primary partition
- extended partition (can not be used without logical part.)
- logical partition


MBR - master boot record 
GPT - GUID Partition table (new partition scheme support 2TB)


### Lab partitions


```sh
 ➜  lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   10G  0 disk 
├─vda1  252:1    0  9.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0    1M  0 disk /mnt/app-config
vdc     252:32   0    1M  0 disk 
vdd     252:48   0    2G  0 disk 
vde     252:64   0    1G  0 disk 
vdf     252:80   0    1G  0 disk 
```

Notice we have 06 disks and 03 partitions.
Notice the size of vdd is 2G
The MAJOR number for devices starting with vd is 252

Creating a partition of 500M from /dev/vde as vde1

```sh
➜  sudo gdisk /dev/vde
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): n
Partition number (1-128, default 1): 1
First sector (34-2097118, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-2097118, default = 2097118) or {+-}size{KMGTP}: +500M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8300
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/vde.
The operation has completed successfully.
```


Lets check the new part created

```sh
➜  lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   10G  0 disk 
├─vda1  252:1    0  9.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0    1M  0 disk /mnt/app-config
vdc     252:32   0    1M  0 disk 
vdd     252:48   0    2G  0 disk 
vde     252:64   0    1G  0 disk 
└─vde1  252:65   0  500M  0 part 
vdf     252:80   0    1G  0 disk 
```



## Filesystem in linux


We must create a filesystem before saving data into the partition.

- ext2
- ext3
- ext4
- xfs

```sh
mkfs.ext4 /dev/sdb1
mkdir /mnt/ext4
mount /dev/sdb1 /mnt/ext4
mount | grep /dev/sdb1
df -hP | grep /dev/sdb1
```



Fstab

Example of fstab line configuration:

```
/dev/dbb1   /mnt/ext4    ext4    rw    0 0 
```



### Lab Filesystems


```sh
➜  lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   10G  0 disk 
├─vda1  252:1    0  9.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0    1M  0 disk /mnt/app-config
vdc     252:32   0    1M  0 disk 
vdd     252:48   0    2G  0 disk /mnt/backups
vde     252:64   0    1G  0 disk 
vdf     252:80   0    1G  0 disk 
```


Notice we have vdd mounted as /mnt/backups and so we have a filesystem there
Notice that vde is not mounted so might not have filesystem.

We can also use:

```sh
➜  df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           143M  4.4M  139M   4% /run
/dev/vda1       9.6G  1.1G  8.5G  11% /
tmpfs           712M     0  712M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda15      105M  6.1M   99M   6% /boot/efi
/dev/vdb        1.0M  1.0M     0 100% /mnt/app-config
tmpfs           143M     0  143M   0% /run/user/1002
/dev/vdd        2.0G   24K  1.9G   1% /mnt/backups
```


The command `blkid` is also helpful to identify partitions with filesystems:

```sh
➜  sudo blkid
/dev/vdb: BLOCK_SIZE="2048" UUID="2026-02-09-07-01-12-00" LABEL="cfgdata" TYPE="iso9660"
/dev/vdc: BLOCK_SIZE="2048" UUID="2026-02-09-07-01-13-00" LABEL="cidata" TYPE="iso9660"
/dev/vda15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="5CEC-7AE6" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="c24f5b29-988e-4d72-9c67-8e6bcfd056f5"
/dev/vda1: LABEL="cloudimg-rootfs" UUID="ea6b38a1-cc67-4624-8a47-dcb6190f25cd" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="5667eb4f-2bed-4312-87ad-064cc7f2be47"
/dev/vdd: UUID="6c4a0eaa-59c4-4c9f-ba43-bf47b04c9703" BLOCK_SIZE="4096" TYPE="ext2"
/dev/vda14: PARTUUID="8e4651e7-28fe-4315-a8bf-22d4152044b9"
```


Notice that /dev/vdd has filesystem ext2

```sh
➜  sudo blkid /dev/vdd
/dev/vdd: UUID="6c4a0eaa-59c4-4c9f-ba43-bf47b04c9703" BLOCK_SIZE="4096" TYPE="ext2"
```


Lets Create an `ext4` filesystem on the disk `/dev/vde` and mount it at `/mnt/data`


```sh
➜  lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   10G  0 disk 
├─vda1  252:1    0  9.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0    1M  0 disk /mnt/app-config
vdc     252:32   0    1M  0 disk 
vdd     252:48   0    2G  0 disk /mnt/backups
vde     252:64   0    1G  0 disk 
vdf     252:80   0    1G  0 disk 

➜  sudo mkfs.ext4 /dev/vde
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done                            
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: b5ea62e9-7524-41fb-97f9-af98f2078590
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done


➜  sudo mkdir /mnt/data

➜  sudo mount /dev/vde /mnt/data/

bob@caleston-lp10 ~ ➜  lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   10G  0 disk 
├─vda1  252:1    0  9.9G  0 part /
├─vda14 252:14   0    4M  0 part 
└─vda15 252:15   0  106M  0 part /boot/efi
vdb     252:16   0    1M  0 disk /mnt/app-config
vdc     252:32   0    1M  0 disk 
vdd     252:48   0    2G  0 disk /mnt/backups
vde     252:64   0    1G  0 disk /mnt/data
vdf     252:80   0    1G  0 disk 
```


Lets add this partition to our fstab


```sh
➜  sudo vi /etc/fstab 
```


```
/dev/vde        /mnt/data       ext4    rw      0 0 
```



## External Storage - DAS NAS and SAN


- DAS - Direct attached storage
- NAS - Network attached storage
- SAN - Storage Area network


### NFS filesystem


