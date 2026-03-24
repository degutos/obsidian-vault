

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

```sh
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

```sh
[[ -f "file.txt" ]] || echo "file does not exist"
```



Check if argument was passed

```sh
[[ -z ${1} ]] && echo "argument empty"
```



## Stream overview

```
0: -> keyboard input -> <
1: -> output message -> > 
2: -> error message -> |
```




### Send error to default output

To send errors and default message to the same place use:

```sh
$ ls -z > file.txt 2>&1
```

Another example to send STDOUT to an STDERR
 
```sh
$ echo "I'm turning this Standard Output echo into a Standard Error" >&2
I'm turning this Standard Output echo into a Standard Error
```

Converting STDERR to STDOUT

```sh
$ ls -j 2>&1
ls: invalid option -- 'j'
Try 'ls --help' for more information.
```
### How to send STDOUT and STDERR to same file

```sh
cat file-no-exist.txt &> output.txt
```

Lets see this sample 

```bash
echo "This is a standard output"
echo "This is an error message" >&2
exit 0
```


Lets execute this file sending the default message to a file and error to another file

```sh
$ ./fd-practice1.sh > /home/bob/results/stdout.txt 2> /home/bob/results/stderr.txt
```

Lets see another sample to silent the errors

```sh
$ find / -type f -name "question8_directory_real" 2> /dev/null
```



## Heredocs

```sh
$ cat <<EOF
> Hello
> World
> From
> Heredocs
> Strings
> EOF
Hello
World
From
Heredocs
Strings
```




```sh
 cat > file.txt <<EOF
heredoc> Andre Gonzaga
heredoc> This is simple test
heredoc> EOF
```

```sh
cat file.txt 
Andre Gonzaga
This is simple test
```


We can use heredocs to run several commands through a ssh session

```sh
$ ssh -T bob@node01 <<EOF
> ls
> EOF
docker_files
kodekloud
nodejs
```


Redirecting content to a file

```sh
[bob@student-node ~]$ cat <<EOF > hello_world.txt
Hello
World
From
Multiple
Lines
EOF

[bob@student-node ~]$ cat hello_world.txt 
Hello
World
From
Multiple
Lines
```


Another example:

```sh
[bob@student-node ~]$ ssh -T  bob@node01 bash "<<EOF >/tmp/init.sql
cat /home/bob/docker_files/backup/sql_files/schema.sql
EOF"
```


How to use heredocs to access container and run a sql query

```sh
$ sudo docker exec my_postgres_container bash -c "psql -U postgres -d employees << EOF
select * from employee;
EOF"
 id_employee | first_name | last_name |   area    |      job_title       |              email              
-------------+------------+-----------+-----------+----------------------+---------------------------------
           1 | Kriti      | Shreshtha | Finance   | Financial Analyst    | kriti shreshtha@company.com
           2 | Rajasekar  | Vasudevan | Finance   | Senior Accountant    | rajasekar.vasudevan@company.com
           3 | Debbie     | Miller    | IT        | Software Developer   | debbie.miller@company.com
           4 | Enrique    | Rivera    | Marketing | Marketing Specialist | enrique.rivera@company.com
           5 | Feng       | Lin       | Sales     | Sales Manager        | feng.lin@company.com
(5 rows)

```


Fixing the email in the first line

```sh
[bob@node01 ~]$ sudo docker exec -i my_postgres_container bash -c "psql -U postgres -d employees <<EOF 
\pset tuples_only on
SELECT email FROM "employee" where first_name='Kriti';
EOF" > /home/bob/kodekloud/employee1_email.txt
```



Write a one-liner command in the shell to count the number of times the word `the` appears in `sample.txt`.

```sh
$ grep -o "the" sample.txt | wc -l
```

Explanation:
```
 -o, --only-matching       show only nonempty parts of lines that match
```


Another example of script with exit error code:

```sh
[bob@student-node ~]$ cat log_parser.sh 
#!/bin/bash

set -o pipefail

# This is the log file we're interested in
logfile="/etc/logs/error.log"

echo "Number of times each error message appears:"
cat "${logfile}" | grep "ERROR" | sort  | uniq  -c || exit 1

exit 0
```




## XARGS

XARGS is a program that helps you to pass arguments to a command that isn't used to be piped.

- sort
- unique 
- cat

These are example of commands that can be used with | pipe. 

Example:

- rm
- ls
- echo
- mkdir


```sh
cat /etc/passwd | xargs echo
```


another example:

```sh
echo "dir1 dir2 dir3" | xargs mkdir
```

this command will create 3 directories 



## Eval

eval help you to write a code and store variables

```sh
 eval 'echo "Hello!!"'
```

or

```sh
cmd="ls -l" ; eval $cmd
total 36
drwxr-xr-x 2 degutos degutos 4096 Mar 10 19:06 dir1
drwxr-xr-x 2 degutos degutos 4096 Mar 10 19:06 dir2
drwxr-xr-x 2 degutos degutos 4096 Mar 10 19:06 dir3
```

or

```sh
file=/etc/passwd ; cmd="cat $file" ; eval $cmd
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```


Try this

```sh
$ ls
$ echo "Hello!!"
echo "Hellols"
Hellols
```

now using eval

```sh
$ eval 'echo "Hello!!"'
Hello!!
```





## Printf

Printf is like echo but with more control.
Echo is command faster and builtin command
printf can format text in different way

\t - TAB
\n - new line
%s - placeholder for the variable string
\n - new line
%d - integers
%f - floating numbers

Example:

```sh
 name=andre ; age=49 ; printf "My name is %s and I am %d years old\n" $name $age
My name is andre and I am 49 years old
```


```sh
$ cat spinning.sh 
#!/usr/bin/env bash

spin='-\|/'

i=0
while [[ true ]]; do
  printf "\r${spin:$((i%4)):1}"
  sleep 0.1
  ((i++))
done

exit 0
```


```sh
[bob@student-node ~]$ ./spinning.sh 
\
```

Another example of spinning to download a package:

```bash
#!/usr/bin/env bash


file_url="https://testfileorg.netwet.net/500MB-CZIPtestfile.org.zip"

# Assign a value to this variable with the path of the spinning.sh script
spin_script_path="/home/bob/spinning.sh"

start_spinner() {
    echo "Beginning the Package Download..."
    # Add a source declaration below to source the spin_script_path while and send it to the background.
    source "${spin_script_path}" &
    spin_pid=$!
}

# Function to download the file
download_file() {
    wget -q "${file_url}" -O large_file
}

# Function to stop the spinner
stop_spinner() {
# Terminate the spin_pid below.
    kill "${spin_pid}"
    echo "Download complete!"
}

# Call the start_spinner function below this line.
start_spinner
download_file
stop_spinner
```


```sh
$ ./package.sh 
Beginning the Package Download...
-Download complete!
```


### Find command with other pipes

Nice command using pipes 

```sh
 find . -type f -name "*.txt" | xargs cat | grep ERROR | sort -k4 | uniq -f3 > errors.log
```


find files type f with extension.txt, cat the content of the files, filter only ERROR, sort by 4th column, and show only uniq by 3rd column till the end with -f3, and redirect everything to external file called errors.log.


### Lets use exec with finder 

```sh
find . -type f -name "*.txt" -exec cp {} /tmp/dir1/ \;
```

-exec is a command to be used with find and give exec an option to copy with cp each file find with command find.

or using RSYNC

```sh
find . -type f -name "*.txt" -exec rsync -R  {} /tmp/dir1 \;
```




