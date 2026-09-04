

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


## Less

```sh
$ less file.txt
```

To search some word:

```sh
/word # to search word
-i # ignore case sensitive 
-i # type again and turn case sensitive
```



## Grep

```sh
$ grep -i centos /etc/os-release # search for centos or Centos (case insensitive) 
$ grep -n centos /etc/os-release # search and show the line number
$ grep -r centos /etc/ # search centos for all files under a directory r = recursive 
$ grep -ri centos /etc # search centos case insensitive for all files in /etc/
$ grep -v centos /etc/os-release # search for lines with NO word centos 
$ grep -wi 'red' /etc/os-release # search for word 'red' and ignore redhat for example, it will show "red hat" for example because "red" is a word 
$ grep -o 'centos' /etc/os-release # only matching, it will display only the word "centos" and not showing the entire line that contains centos word. --only-matching
```



## Regular expressions


```sh
$ grep '^sam' names.txt # search by names that start with sam example samuel but not nisam 
$ grep '^PASS' /etc/login.defs # search for lines that starts with PASS
$ grep 'sam$' names.txt # search for lines that ends with sam
$ grep '7$' /etc/login.defs # search for lines which ends with 7
$ grep 'mail$' /etc/login.defs # search for lines which ends with mail
$ grep -r 'c.t' /etc/ # search for words like cat, cut, cot, execute, etc
$ grep -wr 'c.t' /etc/ # search for exactly word like cat, cut, cst, but not execute 
$ grep '\.' /etc/login.defs # search by period "." with backslash "\" we tell grep to look for . and not a regular expression like above
$ grep 'let*' file.txt # search by le, let, lett, lettt, letttt etc 
$ grep -r '/.*/' /etc/ # search for words that start with / has 0 or more characters and ends with /, example /usr/ , /usr/man, /usr/man/ etc
$ grep -r '0\+' /etc/ # search by word that start with 0 or 00 or 000 etc

##### EGREP

$ grep -Er '0+' /etc/ # same than below
$ egrep -r '0+' /etc/ # search for 0, 00 , 000 etc
$ egrep -r '0{3,}' /etc/ # search for 3x 0 at least, 000 or 0000 or 00000 etc
$ egrep -r '0{3}' /etc/ # search for exactly 3x 0 exemple: 000
$ egrep -r 'disabled?' /etc/ # search for word like disabled or disable or disables
$ egrep -r 'disable|disabled' /etc/ # search for disable or disabled only
$ egrep -r 'disabled?|enabled?' # search for disabled, disable , enabled, enable
# [a-z] show all letters, [0-9] show all numbers, [abz954] show each of these 
$ egrep -r 'c[au]t' /etc/ # search by cat or cut
$ egrep -r '/dev/[a-z]*' /etc/ # search for /dev/any_word example, /dev/hdc, /dev/hdd etc
$ egrep -r '/dev/[a-z]*[0-9]?' /etc/ # search for /dev/anyword_followed by a number or not example: /dev/hdc0 or /dev/hdc, or /dev/sdd
# #### [^] Negated ranges 
$ egrep -r 'http[^s]' /etc/ # search for http not https, also includes httpd
$ egrep -r '/[^a-z]' /etc/ # search for /number or /Capital_word 

```




## SED

Change enabled to disabled global (entire file)
```sh
$ sed -i 's/enabled/disabled/g' values.conf 
```


Change disabled to enabled globaly and ignore case sensitive
```sh
$ sed -i 's/disabled/enabled/gi' values.conf 
```


Change from line 500 to 2000 from enabled to disabled globally 
```sh
$ sed -i '500,2000s/enabled/disabled/g' values.conf 
```


Replace all occurrence of string `#%$2jh//238720//31223` with `$2//23872031223` in `/home/bob/data.txt` file
```sh
$ sed -i 's/#%$2jh\/\/238720\/\/31223/$2\/\/23872031223/g' data.txt 
```


Filter out the lines that contain any word that starts with a `capital` letter and are then followed by exactly `two lowercase letters`
```sh
$ egrep -w '[A-Z][a-z]{2}' /etc/nsswitch.conf > /home/bob/filtered1 
```

