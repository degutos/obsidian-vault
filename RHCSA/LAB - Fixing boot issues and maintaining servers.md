
## Emergency mode

- During the boot process: Grub Menu press any key (arrow key)
- Select the boot option, press "E" to edit 
- Go to line that starts with Linux
- Append  `systemd.unit=emergency.target` at the end of the line
- Press CTRL+x 
- Enter with root password


## Remount / file system read/write

```bash
$ mount -o remount,rw /
$ mount -a

# Fix any error in /etc/fstab

$ systemctl daemon-reload
$ mount -a 
$ systemctl reboot
```


## Default target with systemctl 

```bash
$ systemctl get-default
$ systemctl isolate multi-user.target # (graphical.target)
$ systemctl set-default multi-user.target
$ systemctl reboot
```




## Fixing system or changing root password on RHEL 10


- In the boot grub manu interrupt the count down and edit pressing "i" then got to "kernel" line at the end.
- Add rd.break at the end of the "kernel..." line
- When you get your prompt login type:

```
$ mount -o remount,rw /sysroot 
$ chroot /sysroot
```









