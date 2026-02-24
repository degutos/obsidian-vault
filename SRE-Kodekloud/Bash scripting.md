

### Basic sample

Lets create a very simple snippet to clone a git repository

```sh
[bob@student-node script]$ cat ./clone_project.sh 
#!/bin/bash
cd /home/bob/git
git_project="https://github.com/kodekloudhub/solar-system-9.git"
git clone ${git_project}
```


Lets now create a function for the same snippet

```bash
#!/bin/bash

clone_project() {
   cd /home/bob/git
   git_project="https://github.com/kodekloudhub/solar-system-9.git"
   git clone ${git_project}
}

clone_project
```


During the bash scripting, The `basename` command is used to extract the `filename` or `last` component of a path for a specified element we pass as an argument.


Example:

```bash
$ basename /home/bot/git
git

$ basename ../git/solar-system-9/
solar-system-9
```


another good example

```sh
$ basename abc/def/ghi/filename.txt .txt
filename
```

```sh
$ basename /home/user/documents.doc
documents.doc
```


Lets include the basename function to our code

```sh
#!/bin/bash

project_dir="$(basename https://github.com/kodekloudhub/solar-system-9 .git)"
echo $project_dir

clone_project() {
   cd /home/bob/git
   git clone https://github.com/kodekloudhub/solar-system-9.git
}

clone_project

find "/home/bob/git/${project_dir}" -type f | wc -l
```


Now lets create a function called find_files 

```sh
#!/bin/bash

project_dir="$(basename https://github.com/kodekloudhub/solar-system-9.git .git)"
echo $project_dir

clone_project() {
   cd /home/bob/git
   git clone https://github.com/kodekloudhub/solar-system-9.git
}

find_files() {
  find "/home/bob/git/${project_dir}" -type f | wc -l
}


clone_project
find_files
```


Now lets create a variable $1 to pass the project as argument in the CLI, the way we don't need hard code the git repo address.

```sh
#!/bin/bash
project=${1}
project_dir="$(basename ${project} .git)"

clone_project() {
  cd /home/bob/git/
  git clone ${project}
}

find_files() {
  find "/home/bob/git/${project_dir}" -type f | wc -l
}
clone_project
find_files
```



Lets create another version with git_checkout function 

```sh
#!/bin/bash
project=${1}
branch=${2}
project_dir="$(basename ${project} .git)"

clone_project() {
  if [ ! -d "/home/bob/git/${project_dir}" ] ; then
     cd /home/bob/git/
     git clone ${project}
     cd "${project_dir}"
     git checkout "${branch}"
  fi
}
git_checkout() {
     cd "${project_dir}"
     git checkout "${branch}"
}

find_files() {
  find "/home/bob/git/${project_dir}" -type f | wc -l
}
clone_project
git_checkout
find_files
```



## Builtin x keywords x binary



```
$ type cd
cd is a shell builtin
```


```
$ type kill
kill is a shell builtin
```


```
$ type $(which kill)
/usr/bin/kill is /usr/bin/kill
```


To list all keywords

```
$ compgen -k 
if
then
else
elif
fi
case
esac
for
select
while
until
do
done
in
function
time
{
}
!
[[
]]
coproc
```

Note: Keywords are special tokens parsed by the shell Interpreter. Keywords are generally associated with control block statements, or special words that alter the behavior of a command, or script, such as the words `if`, `while`, `for`, etc.



To list all builtin commands 

```
$ compgen -b
.
:
[
alias
bg
bind
break
builtin
caller
cd
command
compgen
complete
compopt
continue
declare
dirs
disown
echo
enable
eval
exec
exit
export
false
fc
fg
getopts
hash
help
history
jobs
kill
let
local
logout
mapfile
popd
printf
pushd
pwd
read
readarray
readonly
return
set
shift
shopt
source
suspend
test
times
trap
true
type
typeset
ulimit
umask
unalias
unset
wait
```


Shell built-in commands on the other hand are executable programs that are pre-packaged in the shell.


## Which shell we use

```sh
$ ps -p $$
    PID TTY          TIME CMD
   6699 pts/1    00:00:00 bash
```



## Guard Clause 

```
#!/bin/bash
# check if it was passed an argument in the command line
if [[ -z ${1} ]]; then 
   echo "Argument is empty in command line"
fi

git_url=${1}

clone_git() {
   git clone ${1}
}

git clone "${git_url}"

exit 0

```


Check if file exist

```
[[ -f "file.txt" ]] || echo "file does not exist"
```



Check if argument was passed

```sh
[[ -z ${1} ]] && echo "argument empty"
```






