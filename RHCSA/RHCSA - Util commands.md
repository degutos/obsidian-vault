

## Download a file using wget


```sh
$ wget http://materials.example.com/labs/backup-home.sh
```


---

## grep by IP Address

```sh
$ ip a | grep --color=always -E '([0-9]{1,3}\.){3}[0-9]{1,3}|$'
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether dc:a6:32:4f:fd:f7 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether dc:a6:32:4f:fd:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.68.65/22 brd 192.168.71.255 scope global dynamic noprefixroute wlan0
       valid_lft 4262sec preferred_lft 4262sec
    inet6 fe80::34dd:b6f8:93b2:e94e/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

- This will show the entire output of "ip a" and highlight the IP address (red color). The |$ will show the no matching lines also.


---



