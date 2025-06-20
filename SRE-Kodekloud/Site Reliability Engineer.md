


Network-Basics Overview:
https://kodekloud.com/kk-media/image/upload/v1702543781/course-resource-new/Networking-Basics.pdf



## Networking

### check the link 

```sh
ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 02:00:01:51:1b:cb brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname ens3
    altname enx020001511bcb
```

### create an IP manually

```sh
ip addr add 192.168.1.10/24 dev eth0
```

### check route

```sh
 ip r
default via 10.251.0.1 dev eth0 proto dhcp src 10.251.0.4 metric 100
10.251.0.0/24 dev eth0 proto kernel scope link src 10.251.0.4 metric 100
```

### add route manually

```sh
ip route add 192.168.1.2.0/24 via 192.168.1.1

# we can also add a route to a public IP, example:
ip route add 172.217.194.0/24 via 192.168.1.1

# we can also add a default route for a default gateway.
ip route add default via 192.168.2.1
```


Example:

```sh
ip a show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:00:01:51:1b:cb brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname ens3
    altname enx020001511bcb
    inet 10.251.0.4/24 brd 10.251.0.255 scope global dynamic noprefixroute eth0
       valid_lft 210sec preferred_lft 210sec
    inet6 fe80::1ff:fe51:1bcb/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

```

```sh
 ip route
default via 10.251.0.1 dev eth0 proto dhcp src 10.251.0.4 metric 100
10.251.0.0/24 dev eth0 proto kernel scope link src 10.251.0.4 metric 100
```


### Enabling packet forward to different network interface


```sh
cat /proc/sys/net/ipv4/ip_forward
0

echo 1 > /proc/sys/net/ipv4/ip_forward
1
```


#### Lab

1. We have four app server from `app01 to app04`. You can access each app from `jump host` using command `ssh app01` and similarly for other apps. Assign new IPs to each host as per details given below:

a. Assign `172.16.238.15/24` ip address to `app01`

b. Assign `172.16.238.16/24` ip address to `app02`

c. Assign `172.16.239.15/24` ip address to `app03`

d. Assign `172.16.239.16/24` ip address to `app04`

e. We also need to remove existing IPs from these apps after assigning them new IPs but do not remove them right now as it can break your connection, if you are sure you are done with required changes just click on `Check` button below, it will do the rest.


```sh
thor@app01 ~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.11/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
31: eth1@if32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.8/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever


thor@app01 ~$ sudo ip addr add 172.16.238.15/24 dev eth0
thor@app01 ~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.11/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.238.15/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
31: eth1@if32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.8/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```


```sh
thor@app02 ~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
19: eth0@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.12/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
29: eth1@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:07 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.7/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever



thor@app02 ~$ sudo ip addr add 172.16.238.16/24 dev eth0
thor@app02 ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
19: eth0@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.12/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.238.16/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
29: eth1@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:07 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.7/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```


```sh
thor@app03 ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.13/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
27: eth1@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:06 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.6/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever


thor@app03 ~$ sudo ip addr add 172.16.239.15/24 dev eth0
thor@app03 ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.13/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.239.15/24 scope global eth0
       valid_lft forever preferred_lft forever
27: eth1@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:06 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.6/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever    
```



```sh
thor@app04 ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
15: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.14/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
23: eth1@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
thor@app04 ~$ sudo ip addr add 172.16.239.16/24 dev eth0
thor@app04 ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
15: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.14/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.239.16/24 scope global eth0
       valid_lft forever preferred_lft forever
23: eth1@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```



Since now `app03` and `app04` are on different network range than `jump host` so you are not able to SSH into those hosts from jump host. To make `SSH` work make required changes on `jump host`.

a. Assign a new IP address `172.16.239.10/24` to `jump host` with same network range which `app03` and `app04` are using.

```sh
thor@jumphost ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
21: eth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.10/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
31: eth1@if32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.8/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever


thor@jumphost ~$ sudo ip addr add 172.16.239.10/24 dev eth0
thor@jumphost ~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
21: eth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.10/24 brd 172.16.238.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.239.10/24 scope global eth0
       valid_lft forever preferred_lft forever
31: eth1@if32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.8/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```


```sh
thor@jumphost ~$ ssh app03
Last login: Fri May 30 09:54:42 2025 from 172.16.238.10
thor@app03 ~$ 
logout
Connection to app03 closed.

thor@jumphost ~$ ssh app04
Last login: Fri May 30 09:52:26 2025 from 172.16.238.10
thor@app04 ~$
```


---

Lets add a route in app01 and app02 pointing to app03 and app04 networking, and vice-versa.


```sh
thor@app01 ~$ ip route
172.16.238.0/24 dev eth0 proto kernel scope link src 172.16.238.15 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.5 
thor@app01 ~$ sudo ip route add 172.16.239.0/24 via 172.16.238.10
thor@app01 ~$ ip route
172.16.238.0/24 dev eth0 proto kernel scope link src 172.16.238.15 
172.16.239.0/24 via 172.16.238.10 dev eth0 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.5 
```


```sh
thor@app02 ~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
19: eth0@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.238.16/24 scope global eth0
       valid_lft forever preferred_lft forever
29: eth1@if30: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:07 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.7/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
thor@app02 ~$ sudo ip route add 172.16.239.0/24 via 172.16.238.10
thor@app02 ~$ ip route
172.16.238.0/24 dev eth0 proto kernel scope link src 172.16.238.16 
172.16.239.0/24 via 172.16.238.10 dev eth0 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.7 
```


```sh

thor@app03 ~$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 72.16.239.15/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.16.239.15/24 scope global eth0
       valid_lft forever preferred_lft forever
27: eth1@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:06 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.6/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
thor@app03 ~$ ip route
72.16.239.0/24 dev eth0 proto kernel scope link src 72.16.239.15 
172.16.239.0/24 dev eth0 proto kernel scope link src 172.16.239.15 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.6 
thor@app03 ~$ sudo ip route add 172.16.238.0/24 via 172.16.239.10
thor@app03 ~$ ip route
72.16.239.0/24 dev eth0 proto kernel scope link src 72.16.239.15 
172.16.238.0/24 via 172.16.239.10 dev eth0 
172.16.239.0/24 dev eth0 proto kernel scope link src 172.16.239.15 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.6 
```


```sh
thor@app04 ~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
15: eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:10:ee:0e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.239.16/24 scope global eth0
       valid_lft forever preferred_lft forever
23: eth1@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
thor@app04 ~$ ip route
172.16.239.0/24 dev eth0 proto kernel scope link src 172.16.239.16 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.4 
thor@app04 ~$ sudo ip route add 172.16.238.0/24 via 172.16.239.10
thor@app04 ~$ ip route
172.16.238.0/24 via 172.16.239.10 dev eth0 
172.16.239.0/24 dev eth0 proto kernel scope link src 172.16.239.16 
172.17.0.0/16 dev eth1 proto kernel scope link src 172.17.0.4 
```


ping now should work:

```sh
thor@app02 ~$ ping app03
PING app03 (172.16.239.15) 56(84) bytes of data.
64 bytes from app03 (172.16.239.15): icmp_seq=1 ttl=63 time=0.086 ms
64 bytes from app03 (172.16.239.15): icmp_seq=2 ttl=63 time=0.077 ms
^C
--- app03 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.077/0.081/0.086/0.004 ms
thor@app02 ~$ ping app04
PING app04 (172.16.239.16) 56(84) bytes of data.
64 bytes from app04 (172.16.239.16): icmp_seq=1 ttl=63 time=0.134 ms
64 bytes from app04 (172.16.239.16): icmp_seq=2 ttl=63 time=0.110 ms
^C
--- app04 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1006ms
rtt min/avg/max/mdev = 0.110/0.122/0.134/0.012 ms
```



### DNS


```sh
 cat /etc/hosts
# Loopback entries; do not change.
# For historical reasons, localhost precedes localhost.localdomain:
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# See hosts(5) for proper format and other examples:
# 192.168.1.10 foo.example.org foo
# 192.168.1.13 bar.example.org bar
```


```sh
 cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 161.26.0.10
nameserver 161.26.0.11
search mycompany.com # we can use search to force domain of your company example we don't need to ping web.mycompany.com we ping web instead.
```


```sh
cat /etc/nsswitch.conf | grep hosts
hosts:      files  dns myhostname
```


###### DNS Record Types

```sh
A web-server 192.168.1.1 # record for single host 
AAAA web-server 2001:0dbd8:85a3:0000:8aeiou: # record for ipv6
CNAME food.web-server eat.web-server,hungry.web-server # record for alias
```

##### nslookup

To query hostname from a DNS server. It doesn't look at the /etc/hosts file.

##### Dig

To test DNS Name Server Resolution

#### JAVA


**Check Java version**

```sh
thor@host01 ~$ java --version
java 21.0.7 2025-04-15 LTS
Java(TM) SE Runtime Environment (build 21.0.7+8-LTS-245)
Java HotSpot(TM) 64-Bit Server VM (build 21.0.7+8-LTS-245, mixed mode, sharing)
```

**Giving a Java source code**


```sh
thor@host01 /opt/app$ cat MyClass.java 
/**
 * Prints Hello World Message
 *
 */

public class MyClass {
    
    /**
     * Default constructor for the MyClass class.
     */

    public MyClass() {
    }

    /**
     * The main method that prints the message.
     *
     * @param args the command line arguments
     */
    public static void main(String[] args) {
        System.out.println("Hello Kodekloud");
    }
}
```


**Lets compile Java MyClass.java**

```sh
thor@host01 /opt/app$ javac MyClass.java 
thor@host01 /opt/app$ ls -lh
total 8.0K
-rw-r--r-- 1 thor thor 427 May 31 15:55 MyClass.class
-rw-r--r-- 1 thor thor 384 Jan 27 12:24 MyClass.java
```
Notice that it created a new file MyClass.class

```sh
thor@host01 /opt/app$ file MyClass.class 
MyClass.class: compiled Java class data, version 65.0
```


**Lets run the Java application MyClass**

```sh
thor@host01 /opt/app$ java MyClass
Hello Kodekloud
```
Notice that we called MyClass we don't need type the extension .class, this is because we are involking the MyClass class and not the program name


**Lets now create or documentation**

```sh
thor@host01 /opt/app$ javadoc -d doc MyClass.java
Loading source file MyClass.java...
Constructing Javadoc information...
Building index for all the packages and classes...
Standard Doclet version 21.0.7+8-LTS-245
Building tree for all the packages and classes...
Generating doc/MyClass.html...
Generating doc/package-summary.html...
Generating doc/package-tree.html...
Generating doc/overview-tree.html...
Building index for all classes...
Generating doc/allclasses-index.html...
Generating doc/allpackages-index.html...
Generating doc/index-all.html...
Generating doc/search.html...
Generating doc/index.html...
Generating doc/help-doc.html...
```

#### NodeJS

```sh
thor@host01 ~$ sudo yum install nodejs
CentOS Stream 9 - BaseOS                                                  698 kB/s | 8.7 MB     00:12    
CentOS Stream 9 - AppStream                                               8.7 MB/s |  24 MB     00:02    
CentOS Stream 9 - Extras packages                                         8.9 kB/s |  19 kB     00:02    
Extra Packages for Enterprise Linux 9 - x86_64                            2.3 MB/s |  20 MB     00:08    
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64      1.3 kB/s | 2.5 kB     00:01    
Extra Packages for Enterprise Linux 9 - Next - x86_64                     183 kB/s | 414 kB     00:02    
Dependencies resolved.
==========================================================================================================
 Package                    Architecture     Version                            Repository           Size
==========================================================================================================
Installing:
 nodejs                     x86_64           1:16.20.2-8.el9                    appstream           112 k
Installing dependencies:
 nodejs-libs                x86_64           1:16.20.2-8.el9                    appstream            14 M
Installing weak dependencies:
 nodejs-docs                noarch           1:16.20.2-8.el9                    appstream           7.2 M
 nodejs-full-i18n           x86_64           1:16.20.2-8.el9                    appstream           8.2 M
 npm                        x86_64           1:8.19.4-1.16.20.2.8.el9           appstream           2.2 M

Transaction Summary
==========================================================================================================
Install  5 Packages

Total download size: 32 M
Installed size: 168 M
Is this ok [y/N]: y
Downloading Packages:
(1/5): nodejs-16.20.2-8.el9.x86_64.rpm                                     23 kB/s | 112 kB     00:04    
(2/5): nodejs-full-i18n-16.20.2-8.el9.x86_64.rpm                          1.0 MB/s | 8.2 MB     00:08    
(3/5): nodejs-docs-16.20.2-8.el9.noarch.rpm                               884 kB/s | 7.2 MB     00:08    
(4/5): npm-8.19.4-1.16.20.2.8.el9.x86_64.rpm                              5.2 MB/s | 2.2 MB     00:00    
(5/5): nodejs-libs-16.20.2-8.el9.x86_64.rpm                               3.5 MB/s |  14 MB     00:04    
----------------------------------------------------------------------------------------------------------
Total                                                                     3.3 MB/s |  32 MB     00:09     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: npm-1:8.19.4-1.16.20.2.8.el9.x86_64                                              1/1 
  Preparing        :                                                                                  1/1 
  Installing       : nodejs-libs-1:16.20.2-8.el9.x86_64                                               1/5 
  Installing       : nodejs-docs-1:16.20.2-8.el9.noarch                                               2/5 
  Installing       : nodejs-full-i18n-1:16.20.2-8.el9.x86_64                                          3/5 
  Installing       : npm-1:8.19.4-1.16.20.2.8.el9.x86_64                                              4/5 
  Installing       : nodejs-1:16.20.2-8.el9.x86_64                                                    5/5 
  Running scriptlet: nodejs-1:16.20.2-8.el9.x86_64                                                    5/5 
  Verifying        : nodejs-1:16.20.2-8.el9.x86_64                                                    1/5 
  Verifying        : nodejs-docs-1:16.20.2-8.el9.noarch                                               2/5 
  Verifying        : nodejs-full-i18n-1:16.20.2-8.el9.x86_64                                          3/5 
  Verifying        : nodejs-libs-1:16.20.2-8.el9.x86_64                                               4/5 
  Verifying        : npm-1:8.19.4-1.16.20.2.8.el9.x86_64                                              5/5 

Installed:
  nodejs-1:16.20.2-8.el9.x86_64                          nodejs-docs-1:16.20.2-8.el9.noarch               
  nodejs-full-i18n-1:16.20.2-8.el9.x86_64                nodejs-libs-1:16.20.2-8.el9.x86_64               
  npm-1:8.19.4-1.16.20.2.8.el9.x86_64                   

Complete!
thor@host01 ~$ 
```



**Lets check the NodeJs version

```sh
thor@host01 ~$ node -v
v16.20.2
```


**Lets run the NodeJs app

```sh
thor@host01 ~$ node ../add.js 
Addition : 15
```


##### NPM - Node package management

```sh
npm -v # check npm version
npm search file # to list all package with name file
npm install file # to install a new package
npm install file -g # install a package globaly not just inside of your project
```

See the commands with more details

```sh
# lets search and see details for the package file
thor@host01 ~$ npm search file
file
Higher level path and file manipulation functions.
Version 0.2.2 published 2014-02-21 by aconbere
Maintainers: aconbere
https://npm.im/file

# Lets install the package file
thor@host01 ~$ npm install file
added 1 package in 718ms

# we installed under our home directory, lets list those files
thor@host01 ~$ ls -lh
total 12K
drwxr-xr-x 3 thor thor 4.0K Jun  1 06:27 node_modules
-rw-r--r-- 1 thor thor  430 Jun  1 06:27 package-lock.json
-rw-r--r-- 1 thor thor   49 Jun  1 06:27 package.json

# this is json file configuration for the package
thor@host01 ~$ ls -lh node_modules/file/package.json 
-rw-r--r-- 1 thor thor 500 Jun  1 06:27 node_modules/file/package.json

# lets install file globally
thor@host01 ~$ sudo npm install file -g
added 1 package in 745ms

# lets clone a git repo
thor@host01 ~$ git clone https://github.com/contentful/the-example-app.nodejs
Cloning into 'the-example-app.nodejs'...
remote: Enumerating objects: 2935, done.
remote: Counting objects: 100% (19/19), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 2935 (delta 12), reused 0 (delta 0), pack-reused 2916 (from 2)
Receiving objects: 100% (2935/2935), 6.81 MiB | 28.80 MiB/s, done.
Resolving deltas: 100% (1770/1770), done.

# lets check what is the dependency for helmet
thor@host01 ~/the-example-app.nodejs$ cat package.json | grep helmet
    "helmet": "^3.11.0",
```



#### Python

check the python version

```sh
bob@host01:~$ python -V
Python 2.7.17

bob@host01:~$ python3 -V
Python 3.6.9
```


Install python3.8

```sh
bob@host01:~$ sudo apt update && sudo apt install -y python3.8

bob@host01:~$ python3.8 -V
Python 3.8.0
```

```sh
bob@host01:~$ python3 main.py 
Hello new World!
```

Lets run this app with python2 and python3

```sh
bob@host01:~$ python3 main.py 
Hello new World!

bob@host01:~$ python2 main.py
Hello old World!
```


```sh
bob@host01:~$ cat main.py 
import sys

def print_message():
   if sys.version_info[0] < 3:
     print("Hello old World!")
   else:
     print("Hello new World!")

if __name__ == '__main__':
    print_message()
```



##### PIP - python package manager


Lets check the PIP version and python version used by

```sh
thor@host01 ~$ pip -V
pip 21.3.1 from /usr/lib/python3.9/site-packages/pip (python 3.9)
```

Lets install flask with pip

```sh
thor@host01 ~$ sudo pip install flask
```


Lets check more details about the flask

```sh
thor@host01 ~$ sudo pip show flask
Name: Flask
Version: 3.1.1
Summary: A simple framework for building complex web applications.
Home-page: 
Author: 
Author-email: 
License: 
Location: /usr/local/lib/python3.9/site-packages
Requires: blinker, click, importlib-metadata, itsdangerous, jinja2, markupsafe, werkzeug
Required-by: 
```


Lets install many packages from requirements.txt list

```sh
thor@host01 ~$ pip install -r requirements.txt 
```


Lets check the gunicorn version

```sh
thor@host01 ~$ pip show gunicorn
Name: gunicorn
Version: 18.0
Summary: WSGI HTTP Server for UNIX
Home-page: http://gunicorn.org
Author: Benoit Chesneau
Author-email: benoitc@e-engura.com
License: MIT
Location: /usr/local/lib/python3.9/site-packages
Requires: 
Required-by: 
```


Lets upgrade the unicorn version

```sh
thor@host01 ~$ pip install gunicorn --upgrade
```

Lets uninstall gunicorn

```sh
thor@host01 ~$ pip uninstall gunicorn
Found existing installation: gunicorn 23.0.0
Uninstalling gunicorn-23.0.0:
  Would remove:
    /home/thor/.local/bin/gunicorn
    /home/thor/.local/lib/python3.9/site-packages/gunicorn-23.0.0.dist-info/*
    /home/thor/.local/lib/python3.9/site-packages/gunicorn/*
Proceed (Y/n)? Y
  Successfully uninstalled gunicorn-23.0.0
```





### GIT

Lets install git

```sh
thor@host01 ~$ sudo yum install git
```

Let initiate git  repo now

```sh
thor@host01 ~$ git init myrepo
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
Initialized empty Git repository in /home/thor/myrepo/.git/
```

if we don't have a dir myrepo git will create it for us.

Lets git status

```sh
thor@host01 ~/myrepo$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index.html

nothing added to commit but untracked files present (use "git add" to track)
```


Lets add index.html

```sh
thor@host01 ~/myrepo$ git add index.html 
thor@host01 ~/myrepo$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   index.html

thor@host01 ~/myrepo$ git commit -m "Any message"
[master c20dc85] Any message
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```


###### Lets review the git commands

```sh
git init
git add .
git commit -m "Initial Commit"

# now lets create a new GITHUB repository in GitHub webpage.
# then we need tell our local git repo that we have to add a new remote repo
git remote add <repo-local-name> <url>

# exemple
git remote add github-myapplication https://github.com/degutos/my-application.git

# then we can push the code
git push <repo-local-name> <branch>

# example:
# master is a local branch, because we still don't have it in GitHub, we need to use the param -u to create it remotely
git push -u github-myapplication master

# other users can now clone your repo
git clone https://github.com/degutos/my-application.git

# when we clone the repo we don't need specifiy the remote repo, and we can see them here
git remote -v
```


Notice that we also clone remote repository from localhost

```sh
thor@host01 ~$ git clone /opt/remoterepo.git
Cloning into 'remoterepo'...
warning: You appear to have cloned an empty repository.
done.

thor@host01 ~/remoterepo$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index.html

nothing added to commit but untracked files present (use "git add" to track)
thor@host01 ~/remoterepo$ git add index.html 
thor@host01 ~/remoterepo$ git commit -m "new index.html file created"
[master (root-commit) 037130d] new index.html file created
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
thor@host01 ~/remoterepo$ git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 239 bytes | 239.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/remoterepo.git
 * [new branch]      master -> master
```


### Web Server

### Apache Web Server

Lets install apache on our server

```sh
yum install httpd
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

we may need to add a rule to allow http traffic

```sh
firewall-cmd --permanent --add-service=http
```


Checking Logging files

```sh
cat /var/log/httpd/access_log
cat /var/log/httpd/error_log
```


Config file

```sh
vi /etc/httpd/conf/httpd.conf
```

Important setup

```sh
Listen 80
DocumentRoot "/var/www/html"
ServerName www.houses.com:80
```


Virtual hosts

```sh
/etc/httpd/conf/hourses.conf
<VirtualHost *:80>
	ServerName www.houses.com
	DocumentRoot /var/www/houses
</VirtualHost>

/etc/httpd/conf/oranges.cond
<VirtualHost *:80>
	ServerName www.oranges.com
	DocumentRoot /var/www/oranges
</VirtualHost>
```

we can include the virtual hosts files in the main config file /etc/httpd/conf/httpd.conf

```sh
Include conf/houses.conf
Include conf/oranges.conf
```


### Apache Tomcat

#### Installation

```sh
sudo wget https://downloads.apache.org/tomcat/tomcat-11/v11.0.7/bin/apache-tomcat-11.0.7.tar.gz

tar -xvf apache-tomcat-11.0.7.tar.gz 

sudo mv apache-tomcat-11.0.7 /opt/apache-tomcat-11
```


Lets now start Tomcat

```sh
thor@host01 /opt/apache-tomcat-11/bin$ ./startup.sh 
Using CATALINA_BASE:   /opt/apache-tomcat-11
Using CATALINA_HOME:   /opt/apache-tomcat-11
Using CATALINA_TMPDIR: /opt/apache-tomcat-11/temp
Using JRE_HOME:        /usr/lib/jvm/java-21-openjdk-21.0.6.0.7-2.el9.x86_64
Using CLASSPATH:       /opt/apache-tomcat-11/bin/bootstrap.jar:/opt/apache-tomcat-11/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
```

```sh
thor@host01 /opt/apache-tomcat-11/bin$ ps -ef | grep tomcat
thor        5593       1  0 06:03 pts/1    00:00:06 /usr/lib/jvm/java-21-openjdk-21.0.6.0.7-2.el9.x86_64/bin/java -Djava.util.logging.config.file=/opt/apache-tomcat-11/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED --enable-native-access=ALL-UNNAMED -classpath /opt/apache-tomcat-11/bin/bootstrap.jar:/opt/apache-tomcat-11/bin/tomcat-juli.jar -Dcatalina.base=/opt/apache-tomcat-11 -Dcatalina.home=/opt/apache-tomcat-11 -Djava.io.tmpdir=/opt/apache-tomcat-11/temp org.apache.catalina.startup.Bootstrap start
thor        5663    5084  0 06:03 pts/1    00:00:00 grep --color=auto tomcat
```

Tomcat run on port 8081

```sh
thor@host01 /opt/apache-tomcat-11/bin$ ss -tupan | grep 8081
tcp   LISTEN 0      100          0.0.0.0:8081       0.0.0.0:*     users:(("java",pid=5593,fd=44))
```


Lets change the port from 8081 to 9090

```sh
thor@host01 /opt/apache-tomcat-11/conf$ sudo sed -i "s/8081/9090/g" /opt/apache-tomcat-11/conf/server.xml
```

```sh
thor@host01 /opt/apache-tomcat-11/bin$ sudo ./shutdown.sh 
Using CATALINA_BASE:   /opt/apache-tomcat-11
Using CATALINA_HOME:   /opt/apache-tomcat-11
Using CATALINA_TMPDIR: /opt/apache-tomcat-11/temp
Using JRE_HOME:        /usr/lib/jvm/java-21-openjdk-21.0.6.0.7-2.el9.x86_64
Using CLASSPATH:       /opt/apache-tomcat-11/bin/bootstrap.jar:/opt/apache-tomcat-11/bin/tomcat-juli.jar
Using CATALINA_OPTS:   

thor@host01 /opt/apache-tomcat-11/bin$ sudo ./startup.sh 
Using CATALINA_BASE:   /opt/apache-tomcat-11
Using CATALINA_HOME:   /opt/apache-tomcat-11
Using CATALINA_TMPDIR: /opt/apache-tomcat-11/temp
Using JRE_HOME:        /usr/lib/jvm/java-21-openjdk-21.0.6.0.7-2.el9.x86_64
Using CLASSPATH:       /opt/apache-tomcat-11/bin/bootstrap.jar:/opt/apache-tomcat-11/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
```


Lets check if it is running on port 9090

```sh
thor@host01 /opt/apache-tomcat-11/bin$ curl localhost:9090 | tail
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11215    0 11215    0     0  2190k      0 --:--:-- --:--:-- --:--:-- 2190k
                        </ul>
                    </div>
                </div>
                <br class="separator" />
            </div>
            <p class="copyright">Copyright &copy;1999-2025 Apache Software Foundation.  All Rights Reserved</p>
        </div>
    </body>

</html>
```


```sh
thor@host01 /opt/apache-tomcat-11/bin$ ps -ef | grep tomcat
root        6250       1  0 06:14 pts/3    00:00:08 /usr/lib/jvm/java-21-openjdk-21.0.6.0.7-2.el9.x86_64/bin/java -Djava.util.logging.config.file=/opt/apache-tomcat-11/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED --enable-native-access=ALL-UNNAMED -classpath /opt/apache-tomcat-11/bin/bootstrap.jar:/opt/apache-tomcat-11/bin/tomcat-juli.jar -Dcatalina.base=/opt/apache-tomcat-11 -Dcatalina.home=/opt/apache-tomcat-11 -Djava.io.tmpdir=/opt/apache-tomcat-11/temp org.apache.catalina.startup.Bootstrap start
thor        6516    5870  0 06:18 pts/3    00:00:00 grep --color=auto tomcat
```



Lets move a simple application to its default folder

```sh
hor@host01 /opt$ cp sample.war apache-tomcat-11/webapps/

thor@host01 /opt/apache-tomcat-11/webapps$ curl localhost:9090/sample/index.html
<html>
<head>
<title>Sample "Hello, World" Application</title>
</head>
<body bgcolor=white>

<table border="0">
<tr>
<td>
<img src="images/tomcat.gif">
</td>
<td>
<h1>Sample "Hello, World" Application</h1>
<p>This is the home page for a sample application used to illustrate the
source directory organization of a web application utilizing the principles
outlined in the Application Developer's Guide.
</td>
</tr>
</table>

<p>To prove that they work, you can execute either of the following links:
<ul>
<li>To a <a href="hello.jsp">JSP page</a>.
<li>To a <a href="hello">servlet</a>.
</ul>

</body>
</html>
```


### Python with Web application - Flask app

```sh
thor@host01 /opt$ sudo git clone https://github.com/mmumshad/simple-webapp-flask
Cloning into 'simple-webapp-flask'...
remote: Enumerating objects: 54, done.
remote: Counting objects: 100% (25/25), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 54 (delta 19), reused 13 (delta 13), pack-reused 29 (from 1)
Receiving objects: 100% (54/54), 14.51 KiB | 14.51 MiB/s, done.
Resolving deltas: 100% (22/22), done.
thor@host01 /opt$ ls
simple-webapp-flask
```


Lets install requirements

```sh
thor@host01 /opt/simple-webapp-flask$ sudo pip install -r requirements.txt 
Collecting Flask (from -r requirements.txt (line 1))
  Downloading flask-3.1.1-py3-none-any.whl.metadata (3.0 kB)
Collecting blinker>=1.9.0 (from Flask->-r requirements.txt (line 1))
  Downloading blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting click>=8.1.3 (from Flask->-r requirements.txt (line 1))
...
```

we used this pip version and this python version:

```sh
thor@host01 /opt/simple-webapp-flask$ pip -V
pip 25.1.1 from /usr/local/lib/python3.9/site-packages/pip (python 3.9)
```


Lets check the app.py

```sh
thor@host01 /opt/simple-webapp-flask$ cat app.py 
import os
from flask import Flask
app = Flask(__name__)

@app.route("/")
def main():
    return "Welcome!"

@app.route('/how are you')
def hello():
    return 'I am good, how about you?'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```


Lets change the port from 8080 to 5000

```sh
thor@host01 /opt/simple-webapp-flask$ sudo sed -i 's/8080/5000/g' app.py 

thor@host01 /opt/simple-webapp-flask$ cat app.py 
import os
from flask import Flask
app = Flask(__name__)

@app.route("/")
def main():
    return "Welcome!"

@app.route('/how are you')
def hello():
    return 'I am good, how about you?'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```


Lets start our application 

```sh
thor@host01 /opt/simple-webapp-flask$ python3 app.py 
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.16.238.3:5000
Press CTRL+C to quit

```


```sh
thor@host01 ~$ curl http://localhost:5000
Welcome!
```

Now lets run gunicorn

```sh
hor@host01 /opt/simple-webapp-flask$ gunicorn app:app
[2025-06-04 06:41:24 +0000] [3044] [INFO] Starting gunicorn 23.0.0
[2025-06-04 06:41:24 +0000] [3044] [INFO] Listening at: http://127.0.0.1:8000 (3044)
[2025-06-04 06:41:24 +0000] [3044] [INFO] Using worker: sync
[2025-06-04 06:41:24 +0000] [3051] [INFO] Booting worker with pid: 3051
```


```sh
thor@host01 ~$ curl http://localhost:8000
Welcome!
```


Lets run Gunicorn with 3 workers in background using nohup

```sh
thor@host01 /opt/simple-webapp-flask$ nohup gunicorn app:app -w 3 &
[1] 3120
```


```sh
thor@host01 /opt/simple-webapp-flask$ curl http://localhost:8000
Welcome!
```





## SSL and TLS basics


When send user and password to a website like a bank website if we don't encrypt a hacker can sniffer or data and stole our credentials, then we need to encrypt our data on transit.

## Symmetric key

Symmetric key is the process of encrypt our key but we need to send our key with the encrypt key. If a hacker sniffer a data he will be to use the key to decrypt the data.

It's a bit secure way of encrypt the key but since we need to use and send the data a hacker can stole data and key.

## Asymmetric encryption

We have private key and public key or public lock.
The private key is with me and public is based on the server.
The lock is public everyone has access to it, but the private key only you have access.


Questions:



On server side which file keeps the public keys of SSH clients who are allowed to SSH into the server.

~/.ssh/authorized_keys

What is the main drawback of using `symmetric encryption` over `asymmetric encryption`?

Symmetric encryption has to use the same key to encrypt and decrypt and it has to send the key through the network which could be less secure.

How do you create a legitimate certificate for your web server that web browsers can trust?

Get it signed by an authorized certificate authority


How do browsers identify if the certificate is actually signed by the authorized certificate authority itself?

Browsers have CAs public key built in whose respective private key is used to sign the certificate

Create a SSH public/private key called mykey under ~/.ssh/

```sh
thor@host01 ~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/thor/.ssh/id_rsa): /home/thor/.ssh/mykey
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/thor/.ssh/mykey
Your public key has been saved in /home/thor/.ssh/mykey.pub
The key fingerprint is:
SHA256:P831Cn/luIRp3hMMxX1nKoG4UkOfBjUJDBNZs1l/fJg thor@host01.mycorp.org
The key's randomart image is:
+---[RSA 3072]----+
|      +B*++o . . |
|      ..=Bo+..oo=|
|       .oo+ .oEo+|
|      . ..  o... |
|       .S    +.  |
|         . o +o..|
|          o B .+o|
|           + =o.o|
|            . =+ |
+----[SHA256]-----+
```


Copy your key ~/.ssh/mykey.pub to the server

```sh
thor@host01 ~$ ssh-copy-id -i ~/.ssh/mykey.pub thor@app01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/mykey.pub"
The authenticity of host 'app01 (172.16.238.15)' can't be established.
ED25519 key fingerprint is SHA256:85uqfR8u38ctYPdsk5W5TV0WrniIMs52s7orwLj3NAc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'thor@app01'"
and check to make sure that only the key(s) you wanted were added.
```


Install openssl

```sh
$ yum install openssl
```


create a `CSR` (certificate signing request) `/etc/httpd/csr/app01.csr` (key name should be `app01.key`). Below are the required details which should be used while creating CSR.

```sh
thor@app01 /etc/httpd/csr$ sudo openssl req -new -newkey rsa:2048 -nodes -keyout app01.key -out app01.csr
....+++++++++++++++++++++++++++++++++++++++*..+.........+..+..........+..............+..........+..+...+....+..+...+.+............+..+.............+.....+.........+++++++++++++++++++++++++++++++++++++++*...+..+.......+...+....................+.........+.+.....+.............+.....+..........+..+...+.+........+.+..............+.+.....+....+..+.........+....++++++
.........+...+......+.+.....+++++++++++++++++++++++++++++++++++++++*......+..+..........+.....+....+..+...............+.............+.....+.+........+......+....+...+........+.+...+++++++++++++++++++++++++++++++++++++++*.+...+..........+..+..........+.........+........+......+.+...+...+..+..........+...........+.......+...............+..+..........+..+.+..............+....+..+...............+....+...+...........+..........+......+......+.....+...+.........+......+.+...+...+...............+..+..................+.......+........+.........+...+......+.+......+...+...........+...+...+.......+..+.........+......+....+.........+........+..........+.....+........................+...+.+..+...+....+...........+......................+...+......+...+.....+.+..+....+........+...+...................+.........+.........+...+..+............+.+............+.....+...+.......+.....+.+..+...+.+..............+....+.................+.+...+.....+......+.+........+..........+.........+...+.........+.....+...+......+.+......+..+...+....+..............+............+...+.+...+...+.....+...+.+...+...............+......+..+.........+...+..........+.....+...+....+.....+...+.............+..+...+...+....+.....+...+....+...+...+...+..+.........++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:SG
State or Province Name (full name) []:Capital Tower
Locality Name (eg, city) [Default City]:CT
Organization Name (eg, company) [Default Company Ltd]:KodeKloud
Organizational Unit Name (eg, section) []:Education
Common Name (eg, your name or your server's hostname) []:app01.com
Email Address []:admin@kodekloud.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
thor@app01 /etc/httpd/csr$ 
thor@app01 /etc/httpd/csr$ ls
app01.csr  app01.key
```


Lets create another CSR

```sh
thor@app01 /etc/httpd/certs$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout app01.key -out app01.crt
.+++++++++++++++++++++++++++++++++++++++*.......+.+.....+.+......+.....+....+...+........+...+................+...+......+..+..........+..+...+...+++++++++++++++++++++++++++++++++++++++*................+..............+.+........................+......+..+..........+..+.+........+.+.........+......+...+...+..............+......+...+.......+..+.....................+.+...+...........+.+...+...+..+......+..........+......+......+.........+.....+.+..............+...+...+....+.....+....+.....+....+........+.......+.....+......+.+........+.+..+....+...........+.........+.+...+..+...+......+....+..............+.......+.........+.....+.+......+.........+......+........+......+................+........+......+...+..................+....+..................+...........+.+.....+.+.....+.......+..+...+.............+........+............+.......+..+.........+.+...+.....+...+...+................+......+.....+.......+...+..+......+...................+......+......+.....+...+...+...+.........+.+.....+.......+..............+...+.+..+.......+.....+......+.+.....+......+...+......++++++
......+.+......+...............+++++++++++++++++++++++++++++++++++++++*.........+..+...+......+....+....................+.+++++++++++++++++++++++++++++++++++++++*......+..+.+......+.....................+...+..+.......+...........+.+..................+......+......+...........+....++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:SG
State or Province Name (full name) []:Capital Tower
Locality Name (eg, city) [Default City]:CT
Organization Name (eg, company) [Default Company Ltd]:Kodekloud
Organizational Unit Name (eg, section) []:Education
Common Name (eg, your name or your server's hostname) []:app01.com
Email Address []:admin@kodekloud.com
```


configured and ssl mode is already enabled. In the `/etc/httpd/conf.d/ssl.conf` file update the SSL certificate and key to use your `app01.crt` and `app01.key`

```sh
thor@app01 /etc/httpd/certs$ sudo vi /etc/httpd/conf.d/ssl.conf 
```

You need to modify the `/etc/httpd/conf.d/ssl.conf` file. Specifically, update the `SSLCertificateFile` and `SSLCertificateKeyFile` directives to point to your `/etc/httpd/certs/app01.crt` and `/etc/httpd/certs/app01.key` files.



## YAML and JSON


How to search a criteria in JSON

```json
[
12,
43,
12,
23,
8,
19
25,
7
]
```


Lets create a criteria that checks if item in the array is > 20
```json
$[?(@>20)]
```

```sh
$[ start of List 
?() check the criteria
@ each item in the list
> greater than
```

Other samples of criteria

```json
@ == 20 each item in the list iqual 20
@ != 20 each item in the list different than 20
@ in [20,43,45] check if each item in the list is in the new list
@ nin [20.43,45] check if each item in the list is not in the new list
```

Lets see an example to display an item with criteria

```json
$.car.wheels[?(@.location -- "rear-right")].model
```


Lets do this filter check this file:

```json
{
    "prizes": [
        {
            "year": "2018",
            "category": "physics",
            "overallMotivation": "\"for groundbreaking inventions in the field of laser physics\"",
            "laureates": [
                {
                    "id": "960",
                    "firstname": "Arthur",
                    "surname": "Ashkin",
                    "motivation": "\"for the optical tweezers and their application to biological systems\"",
                    "share": "2"
                },
                {
                    "id": "961",
                    "firstname": "Gérard",
                    "surname": "Mourou",
                    "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
                    "share": "4"
                },
                {
                    "id": "962",
                    "firstname": "Donna",
                    "surname": "Strickland",
                    "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2018",
            "category": "chemistry",
            "laureates": [
                {
                    "id": "963",
                    "firstname": "Frances H.",
                    "surname": "Arnold",
                    "motivation": "\"for the directed evolution of enzymes\"",
                    "share": "2"
                },
                {
                    "id": "964",
                    "firstname": "George P.",
                    "surname": "Smith",
                    "motivation": "\"for the phage display of peptides and antibodies\"",
                    "share": "4"
                },
                {
                    "id": "965",
                    "firstname": "Sir Gregory P.",
                    "surname": "Winter",
                    "motivation": "\"for the phage display of peptides and antibodies\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2018",
            "category": "medicine",
            "laureates": [
                {
                    "id": "958",
                    "firstname": "James P.",
                    "surname": "Allison",
                    "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
                    "share": "2"
                },
                {
                    "id": "959",
                    "firstname": "Tasuku",
                    "surname": "Honjo",
                    "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2018",
            "category": "peace",
            "laureates": [
                {
                    "id": "966",
                    "firstname": "Denis",
                    "surname": "Mukwege",
                    "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
                    "share": "2"
                },
                {
                    "id": "967",
                    "firstname": "Nadia",
                    "surname": "Murad",
                    "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2018",
            "category": "economics",
            "laureates": [
                {
                    "id": "968",
                    "firstname": "William D.",
                    "surname": "Nordhaus",
                    "motivation": "\"for integrating climate change into long-run macroeconomic analysis\"",
                    "share": "2"
                },
                {
                    "id": "969",
                    "firstname": "Paul M.",
                    "surname": "Romer",
                    "motivation": "\"for integrating technological innovations into long-run macroeconomic analysis\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2014",
            "category": "peace",
            "laureates": [
                {
                    "id": "913",
                    "firstname": "Kailash",
                    "surname": "Satyarthi",
                    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
                    "share": "2"
                },
                {
                    "id": "914",
                    "firstname": "Malala",
                    "surname": "Yousafzai",
                    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2017",
            "category": "physics",
            "laureates": [
                {
                    "id": "941",
                    "firstname": "Rainer",
                    "surname": "Weiss",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "2"
                },
                {
                    "id": "942",
                    "firstname": "Barry C.",
                    "surname": "Barish",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "4"
                },
                {
                    "id": "943",
                    "firstname": "Kip S.",
                    "surname": "Thorne",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2017",
            "category": "chemistry",
            "laureates": [
                {
                    "id": "944",
                    "firstname": "Jacques",
                    "surname": "Dubochet",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                },
                {
                    "id": "945",
                    "firstname": "Joachim",
                    "surname": "Frank",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                },
                {
                    "id": "946",
                    "firstname": "Richard",
                    "surname": "Henderson",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                }
            ]
        },
        {
            "year": "2017",
            "category": "medicine",
            "laureates": [
                {
                    "id": "938",
                    "firstname": "Jeffrey C.",
                    "surname": "Hall",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                },
                {
                    "id": "939",
                    "firstname": "Michael",
                    "surname": "Rosbash",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                },
                {
                    "id": "940",
                    "firstname": "Michael W.",
                    "surname": "Young",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                }
            ]
        }
    ]
}
```


Lets display this content

```json
[
  {
    "id": "914",
    "firstname": "Malala",
    "surname": "Yousafzai",
    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
    "share": "2"
  }
]
```


Considering the original file is called q11.json lets create a filter

```sh
cat q11.json | jpath $.prizes[5].laureates[1]
```


```sh
➜  cat q11.json | jpath $.prizes[5].laureates[1]
[
  {
    "id": "914",
    "firstname": "Malala",
    "surname": "Yousafzai",
    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
    "share": "2"
  }
]
```


Lets do another sample with a list

```sh
➜  cat q12.json 
[
    "car",
    "bus",
    "truck",
    "bike"
]
 ➜  cat q12.json | jpath $[0]
[
  "car"
]
```

Another sample

```sh
➜  cat q13.json 
[
    "car",
    "bus",
    "truck",
    "bike"
]
 ➜  cat q13.json | jpath '$[0,3]'
[
  "car",
  "bike"
]
```
