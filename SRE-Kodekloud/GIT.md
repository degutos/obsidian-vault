
## GIT Installation

### Check OS version

```sh
➜  cat /etc/os-release 
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.22.2
PRETTY_NAME="Alpine Linux v3.22"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```


### Installing git on Alpine

```sh
sarah@dev01 ~ ➜  sudo apk update
v3.22.6-2-gae7e358ab5f [https://dl-cdn.alpinelinux.org/alpine/v3.22/main]
v3.22.5-142-g17748e49b18 [https://dl-cdn.alpinelinux.org/alpine/v3.22/community]
v20260805-2643-gd80c556eeb9 [http://dl-cdn.alpinelinux.org/alpine/edge/testing]
OK: 33663 distinct packages available

sarah@dev01 ~ ➜  sudo apk add git
(1/3) Installing pcre2 (10.46-r0)
(2/3) Installing git (2.49.1-r0)
(3/3) Installing git-init-template (2.49.1-r0)
Executing busybox-1.37.0-r19.trigger
OK: 114 MiB in 140 packages
```

### Check git version

```sh
sarah@dev01 ~ ➜  git --version
git version 2.49.1
```

To install git manual-pages

```sh
sarah@dev01 ~ ➜  sudo apk add git-doc
(1/1) Installing git-doc (2.49.1-r0)
OK: 115 MiB in 141 packages
```

PS: git branch is used to list, create or delete branches
PS: git fetch      Download objects and refs from another repository
PS: git init       Create an empty Git repository or reinitialize an existing one


### GIT Lab - Init

#### git init

```sh
sarah@dev01 ~/story-blog ➜  git init
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
Initialized empty Git repository in /home/sarah/story-blog/.git/
```


Check the hidden directory

```sh
sarah@dev01 story-blog on  master ➜  ls -lah
total 16K    
drwxr-sr-x    3 sarah    sarah       4.0K Sep 19 16:45 .
drwxr-sr-x    1 sarah    sarah       4.0K Sep 19 16:45 ..
drwxr-sr-x    6 sarah    sarah       4.0K Sep 19 16:45 .git
```



### Adding a file and managing git workflow

```sh
sarah@dev01 story-blog on  master ➜  echo "A Lion lay asleep in the forest" > lion-and-mouse.txt

sarah@dev01 story-blog on  master [?] ➜  git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        lion-and-mouse.txt

nothing added to commit but untracked files present (use "git add" to track)
```


### Staging a file

```sh
sarah@dev01 story-blog on  master [?] ➜  git add lion-and-mouse.txt 

sarah@dev01 story-blog on  master [+] ➜  git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   lion-and-mouse.txt
```



### Configuring git directory for the first time

```sh
sarah@dev01 story-blog on  master [+] ➜  git config user.email "sarah@example.com"

sarah@dev01 story-blog on  master [+] ➜  git config user.name sarah
```


### GIT Commit 

```sh
sarah@dev01 story-blog on  master [+] ➜  git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   lion-and-mouse.txt


sarah@dev01 story-blog on  master [+] ➜  git commit -m "Added the lion and mouse story"
[master (root-commit) 71cf733] Added the lion and mouse story
 1 file changed, 1 insertion(+)
 create mode 100644 lion-and-mouse.txt

sarah@dev01 story-blog on  master ➜  git status
On branch master
```


### New file created to keep out of staging

```sh
sarah@dev01 story-blog on  master ➜  ls
lion-and-mouse.txt  notes.txt

sarah@dev01 story-blog on  master [?] ➜  git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        notes.txt

nothing added to commit but untracked files present (use "git add" to track)
```


#### GIT ignore

```sh
sarah@dev01 story-blog on  master [?] ➜  vi .gitignore

sarah@dev01 story-blog on  master [?] ➜  git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore

nothing added to commit but untracked files present (use "git add" to track)

sarah@dev01 story-blog on  master [?] ➜  cat .gitignore 
notes.txt
```


#### Adding gitignore

```sh
sarah@dev01 story-blog on  master [?] ➜  git add .gitignore 

sarah@dev01 story-blog on  master [+] ➜  git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .gitignore
```


### Another REPO with one staged file and one modified file

```sh
sarah@dev01 learning-app-ecommerce on  master [!+⇡] via 🐘 ➜  git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   js/theme.js
```



### Commiting two different commits 

We need to make two separate commits in your repository:

  

1. Commit the `README.md` file
    
    - Use the commit message: `Add instructions for verification`
    - Note: `README.md` is already staged, so you can commit it directly.
2. Commit the `js/theme.js` file
    
    - Use the commit message: `Increase time from 400 to 500`
    - Note: This file is not staged yet, so you need to add and commit it in one step.


```sh
sarah@dev01 learning-app-ecommerce on  master [!+⇡] via 🐘 ➜  git commit -m "Add instructions for verification"
[master 656f863] Add instructions for verification
 1 file changed, 1 insertion(+)

sarah@dev01 learning-app-ecommerce on  master [!⇡] via 🐘 ➜  git add js/theme.js 

sarah@dev01 learning-app-ecommerce on  master [+⇡] via 🐘 ➜  git commit -m "Increase time from 400 to 500"
[master a27c1a6] Increase time from 400 to 500
 1 file changed, 1 insertion(+), 1 deletion(-)

sarah@dev01 learning-app-ecommerce on  master [⇡] via 🐘 ➜  git status
On branch master
Your branch is ahead of 'origin/master' by 3 commits.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean

```



### Git log

```sh
 ➜  git log
commit 049529b2a94a9ed396ca05d22e4619b44c9551ce (HEAD -> master)
Author: sarah <sarah@example.com>
Date:   Fri Sep 25 11:02:02 2026 +0000

    Added the lion and mouse story
```



Check which file is changed in the commit

```sh
➜  git log --name-only
commit 049529b2a94a9ed396ca05d22e4619b44c9551ce (HEAD -> master)
Author: sarah <sarah@example.com>
Date:   Fri Sep 25 11:02:02 2026 +0000

    Added the lion and mouse story

lion-and-mouse.txt
```


checking again logs when more commits done:

```sh
➜  git log --name-only
commit 9ceabfb55c7cd8c53913aca145140907af951616 (HEAD -> master)
Author: tom <tom@example.com>
Date:   Fri Sep 25 11:06:03 2026 +0000

    Added a new story

frogs-and-ox.txt

commit 049529b2a94a9ed396ca05d22e4619b44c9551ce
Author: sarah <sarah@example.com>
Date:   Fri Sep 25 11:02:02 2026 +0000

    Added the lion and mouse story

lion-and-mouse.txt
```



Get the logs in one line:

```sh
 ➜  git log --oneline
9ceabfb (HEAD -> master) Added a new story
049529b Added the lion and mouse story
```


To get the last 3 commits logs:

```sh
sarah@dev01 learning-app-ecommerce on  master [⇡] via 🐘 ✖ git log -3
commit 5ac38555429ffa6217cc32ee52ff000904335217 (HEAD -> master)
Author: tej <tej@example.com>
Date:   Fri Sep 25 10:59:32 2026 +0000

    Update color from red to green

commit d66bf77e8aa8fe59cf1a3d4f3464be7c492e5d87
Author: sarah <sarah@example.com>
Date:   Fri Sep 25 10:59:32 2026 +0000

    Add instructions to verify application

commit f6737e393f10848e157551a4fec29c70515d10f7
Author: max <max@example.com>
Date:   Fri Sep 25 10:59:32 2026 +0000

    Increase interval time to 500
```



Check the last 5 commit logs and files changed with --name-only

```sh
➜  git log -5 --name-only
commit 5ac38555429ffa6217cc32ee52ff000904335217 (HEAD -> master)
Author: tej <tej@example.com>
Date:   Fri Sep 25 10:59:32 2026 +0000

    Update color from red to green

css/style.css

commit d66bf77e8aa8fe59cf1a3d4f3464be7c492e5d87
Author: sarah <sarah@example.com>
Date:   Fri Sep 25 10:59:32 2026 +0000

    Add instructions to verify application

README.md

commit f6737e393f10848e157551a4fec29c70515d10f7
Author: max <max@example.com>
Date:   Fri Sep 25 10:59:32 2026 +0000

    Increase interval time to 500

js/theme.js

commit e01753fd80bb5efe5c1117bb1e487e7a779fa6a3 (origin/master, origin/HEAD)
Merge: 763e2c5 d0a698d
Author: Mumshad Mannambeth <2704615+mmumshad@users.noreply.github.com>
Date:   Thu Aug 1 22:58:51 2024 +0800

    Merge pull request #21 from nimeshmora/update-readme-guide-setting-up-env-file-and-fetch
    
    update readme file on guiding how to setup env file and fetch environment details

commit d0a698d482838f0397cd5ed6031875cac1d831f4
Author: nimesha <mhnimesh@gmail.com>
Date:   Thu Aug 1 19:26:01 2024 +0530

    update readme file on guiding how to setup env file and fetch details from it within the index.php

README.md
```


