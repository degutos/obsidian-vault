

## Scheduling a job (backup) as a normal user

```sh
$ crontab -e 
# this command edit the crontab file
0 19-21 * * mon-fri /home/student/backup-home
# min - hour (from 19 to 21h) - date - month - weekday - command
# 0 = min
# 19-21 = from 19 to 21h
# * = date 
# * = month
# mon-fri = weekday
# /home/student/backup-home = Command
$ crontab -l
```





