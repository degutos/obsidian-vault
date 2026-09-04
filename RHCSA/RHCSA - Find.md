

## Find

```sh
$ find -mmin [minute] # modified minute
$ find -mmin 5 # modified only 5 minutes ago, during that minute only
$ find -mmin -5 # modified within last 5 min until current time
$ find -mmin +5 # modified before 5 minutes ago, all files modified until 5 minutes ago, except the last 5 minutes.
$ find -mtime 2 # modified the last 24-hours 
$ find -cmin -5 # changed metadata (perm rwx) within the last 5 minutes
$ find -size [size]
$ find -size 512k # find file with exactly 512 KB (kilobyte) (c=bytes, k=kilobytes, M=megabytes, G=gigabytes)
$ find -size +512k # find files greater than 512kb
$ find -size -512k # find files less than 512 Kb
$ find -name "f*" # find files that start with letter "f"
$ find -iname felix # find files with name felix or FELIX (not case sensitive)
$ find -name "f*" -size 512k # find files that starts with "f" AND size 512 kb
$ find -name "f*" -o -size 512k # find files that starts with "f" OR size 512 Kb
$ find -not -name "f*" # find files that does NOT start with letter "f" 
$ find -perm 664 # find files that has perm exactly 664 (rw, rw, r)
$ find -perm -664 # find files that has at least this perm 664 
$ find -perm /664 # find files with any of these perm, (rw, rw, r) if user has r or w will show up
# another way of using perm 
$ find -perm u=rw,g=rw,o=r # find exactly 665 permissions
$ find -perm -u=rw,g=rw,o=r # find files with at least 664 permissions
$ find -perm /u=rw,g=42,o=r # find files with any of these permissions 
```