

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



