
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



