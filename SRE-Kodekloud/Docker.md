
## Docker introduction Kodekloud


Docker is very useful to use several different applications running in the same server, containerizing applications like web server like NodeJS, database like MongoDB,  messaging system and Ansible orchestration we can have all these pieces together in the same physical host using container for each application separating libraries and dependencies.

### Containers

Containers are completely isolated environments on top of OS kernel and Docker engine with their own processes, network and mounts volumes, they all share all the same kernel. 
Containers are not new in Linux, it is an old concept.


### How is it done


```sh
docker run ansible
docker run mongodb
docker run redis
docker run nodejs
docker run nodejs
```


### Container x image

Containers are images running and containerized. You can have one image and create many containers
Images are template that we can download to run containers. Containers are running images.
We can create our own image or download a ready one.
Usually Developers can build the own image and handoff to Ops team to deploy it in production. 


### Docker Editions

- Docker community edition
- Docker enterprise edition


### Docker Installation 


https://docs.docker.com/install


#### Docker engine

We can install the docker engine instead of docker Desktop

Go to: https://docs.docker.com/engine/

Check your OS version:

```sh
root@ubuntu-host ~ ➜  cat /etc/os-release 
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```


Make sure we have no firewall installed

```sh
root@ubuntu-host ~ ➜  ufw
-bash: ufw: command not found

root@ubuntu-host ~ ✖ iptables -L
-bash: iptables: command not found
```


Uninstall any docker package previously installed

```sh
root@ubuntu-host ~ ✖ sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
dpkg: no packages found matching docker.io
dpkg: no packages found matching docker-compose
dpkg: no packages found matching docker-compose-v2
dpkg: no packages found matching docker-doc
dpkg: no packages found matching podman-docker
dpkg: no packages found matching containerd
dpkg: no packages found matching runc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```


### Installing

```sh
root@ubuntu-host ~ ✖ curl -fsSL https://get.docker.com -o get-docker.sh

root@ubuntu-host ~ ➜  sudo sh get-docker.sh
# Executing docker install script, commit: c04fb16bb8bd8ed6ce884bb40570cbcd6101ae0c
+ sh -c apt-get -qq update >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install ca-certificates curl >/dev/null
debconf: delaying package configuration, since apt-utils is not installed
+ sh -c install -m 0755 -d /etc/apt/keyrings
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" -o /etc/apt/keyrings/docker.asc
+ sh -c chmod a+r /etc/apt/keyrings/docker.asc
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get -qq update >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin docker-model-plugin >/dev/null
debconf: delaying package configuration, since apt-utils is not installed
```

When you run docker command make sure you don't get error like this:

```sh
failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory
```

this is a sample that docker is not running and we might need start docker.


### Starting docker Engine

```sh
root@ubuntu-host /var/run ✖ sudo systemctl start docker

root@ubuntu-host /var/run ➜  sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-04-03 09:32:20 EDT; 3s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 7017 (dockerd)
      Tasks: 19
     Memory: 25.9M (peak: 27.9M)
        CPU: 266ms
     CGroup: /system.slice/docker.service
             └─7017 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```




#### Checking version

```sh
root@ubuntu-host /var/run ➜  docker version
Client: Docker Engine - Community
 Version:           29.3.1
 API version:       1.54
 Go version:        go1.25.8
 Git commit:        c2be9cc
 Built:             Wed Mar 25 16:13:43 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.3.1
  API version:      1.54 (minimum version 1.40)
  Go version:       go1.25.8
  Git commit:       f78c987
  Built:            Wed Mar 25 16:13:43 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.2
  GitCommit:        301b2dac98f15c27117da5c8af12118a041a31d9
 runc:
  Version:          1.3.4
  GitCommit:        v1.3.4-0-gd6d73eb8
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```


### Running first container

```sh
root@ubuntu-host /var/run ✖ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
d5e71e642bf5: Download complete 
Digest: sha256:452a468a4bf985040037cb6d5392410206e47db9bf5b7278d281f94d1c2d0931
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```



#### Docker run


```sh
docker run
```

docker run is a command to run containers from an image, example:

```sh
docker run nginx
```


If the image is not present on the node it goes to docker.hub and download the image, but this is done only the first time, then the image is stored in the local host.

docker ps: list containers
docker ps -a: list all containers running or not, to list all stopped containers

```sh
root@ubuntu-host ~ ➜  docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                      PORTS     NAMES
3d452bbe2650   nginx         "/docker-entrypoint.…"   16 minutes ago   Exited (0) 15 minutes ago             sad_albattani
42342f3c48f4   hello-world   "/hello"                 21 minutes ago   Exited (0) 21 minutes ago             romantic_villani
9f75fd75dd0e   hello-world   "/hello"                 23 minutes ago   Exited (0) 23 minutes ago             modest_jennings
```


To delete a container:

```sh
root@ubuntu-host ~ ➜  docker rm modest_jennings
modest_jennings
```


```sh
root@ubuntu-host ~ ➜  docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                      PORTS     NAMES
3d452bbe2650   nginx         "/docker-entrypoint.…"   18 minutes ago   Exited (0) 17 minutes ago             sad_albattani
42342f3c48f4   hello-world   "/hello"                 23 minutes ago   Exited (0) 23 minutes ago             romantic_villani
```

List all images:

```sh
root@ubuntu-host ~ ➜  docker images
                                                                                   i Info →   U  In Use
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   452a468a4bf9       25.9kB         9.49kB    U   
nginx:latest         7150b3a39203        240MB         65.8MB    U   
```



deleting another container:

```sh
root@ubuntu-host ~ ✖ docker rm sad_albattani
sad_albattani
```


Deleting an image:

```sh
root@ubuntu-host ~ ➜  docker rmi nginx:latest
Untagged: nginx:latest
Deleted: sha256:7150b3a39203cb5bee612ff4a9d18774f8c7caf6399d6e8985e97e28eb751c18
```


To download an image:

```sh
root@ubuntu-host ~ ➜  docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
5e815e07e569: Pull complete 
cde7a05ae428: Pull complete 
3189680c601f: Pull complete 
bb3d0aa29654: Pull complete 
510ddf6557d6: Pull complete 
587e3d84dbb5: Pull complete 
ec781dee3f47: Pull complete 
669e0ab8e7fa: Download complete 
96a6cfe061e0: Download complete 
Digest: sha256:7150b3a39203cb5bee612ff4a9d18774f8c7caf6399d6e8985e97e28eb751c18
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```


To run an new container

```sh
root@ubuntu-host ~ ➜  docker run ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
817807f3c64e: Pull complete 
e4a1e8de092c: Download complete 
Digest: sha256:186072bba1b2f436cbb91ef2567abca677337cfc786c86e107d25b7072feef0c
Status: Downloaded newer image for ubuntu:latest
```



check there is no container running

```sh
root@ubuntu-host ~ ➜  docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

root@ubuntu-host ~ ➜  
```


Check all stopped container

```sh
root@ubuntu-host ~ ➜  docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                      PORTS     NAMES
a7f6d49757f0   ubuntu        "/bin/bash"   59 seconds ago   Exited (0) 57 seconds ago             bold_dewdney
42342f3c48f4   hello-world   "/hello"      28 minutes ago   Exited (0) 28 minutes ago             romantic_villani
```

Containers will live only while the process is running, otherwise it will stop 


```sh
root@ubuntu-host ~ ➜  docker run ubuntu sleep 60
```


```sh
root@ubuntu-host ~ ➜  docker ps
CONTAINER ID   IMAGE     COMMAND      CREATED          STATUS         PORTS     NAMES
be6488fac87a   ubuntu    "sleep 60"   10 seconds ago   Up 9 seconds             strange_shaw
```


Lets now run another container and exec into it:

```sh
root@ubuntu-host ~ ✖ docker ps
CONTAINER ID   IMAGE     COMMAND      CREATED         STATUS         PORTS     NAMES
3546a71fcc62   ubuntu    "sleep 60"   5 seconds ago   Up 5 seconds             sad_curran

root@ubuntu-host ~ ➜  docker exec 3546a71fcc62 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::  ip6-localnet
ff00::  ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      3546a71fcc62
```


Attached mode:

```sh
root@ubuntu-host ~ ➜   docker run kodekloud/simple-webapp
Unable to find image 'kodekloud/simple-webapp:latest' locally
latest: Pulling from kodekloud/simple-webapp
e337be014a61: Pull complete 
7cf6a1d62200: Pull complete 
f0d690b9e495: Pull complete 
dd9b067ef6fd: Pull complete 
4fe2ade4980c: Pull complete 
fac5d45ad062: Pull complete 
7454877e71d0: Pull complete 
Digest: sha256:e5355b4c7804f453d79de75d6659ee702eeebbe30c02d9f1ce6602a96b576e57
Status: Downloaded newer image for kodekloud/simple-webapp:latest
 This is a sample web application that displays a colored background. 
 A color can be specified in two ways. 

 1. As a command line argument with --color as the argument. Accepts one of red,green,blue,blue2,pink,darkblue 
 2. As an Environment variable APP_COLOR. Accepts one of red,green,blue,blue2,pink,darkblue 
 3. If none of the above then a random color is picked from the above list. 
 Note: Command line argument precedes over environment variable.


No command line argument or environment variable. Picking a Random Color =red
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
```


Another sample:

```sh
admin@docker-host ✖ docker run -it bash bash

bash-5.3# cat /etc/os-release 
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.23.3
PRETTY_NAME="Alpine Linux v3.23"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```




### Running docker in detached mode

```sh
root@ubuntu-host ~ ➜  docker run -d kodekloud/simple-webapp
da24c0d3a2592f0276bb336c20fb190208c1f3a2e21f87ae52d7a92867186a0c

root@ubuntu-host ~ ➜  docker ps
CONTAINER ID   IMAGE                     COMMAND           CREATED         STATUS         PORTS      NAMES
da24c0d3a259   kodekloud/simple-webapp   "python app.py"   4 seconds ago   Up 3 seconds   8080/tcp   tender_knuth
```



```sh
root@ubuntu-host ~ ➜  docker ps
CONTAINER ID   IMAGE                     COMMAND           CREATED         STATUS         PORTS      NAMES
da24c0d3a259   kodekloud/simple-webapp   "python app.py"   4 seconds ago   Up 3 seconds   8080/tcp   tender_knuth

root@ubuntu-host ~ ➜  docker attach da24c0d3a259
```


### How to exit from a container keeping it running


```sh
CTRL + p followed by CTRL + q
```


### Running a command within the container 

We can use docker exec to execute commands within the container, example:

```sh
admin@docker-host ➜  docker exec hardcore_feynman cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.23.3
PRETTY_NAME="Alpine Linux v3.23"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```



## Docker basic commands - LAB


Check current docker server engine 

```sh
~ ➜  docker version
Client:
 Version:           25.0.5
 API version:       1.44
 Go version:        go1.20.11
 Git commit:        d260a54c81efcc3f00fe67dee78c94b16c2f8692
 Built:             Mon Apr 15 18:14:16 2024
 OS/Arch:           linux/amd64
 Context:           default

Server:
 Engine:
  Version:          25.0.5
  API version:      1.44 (minimum version 1.24)
  Go version:       go1.20.11
  Git commit:       e63daec8672d77ac0b2b5c262ef525c7cf17fd20
  Built:            Mon Apr 15 18:14:16 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.7.2
  GitCommit:        0cae528dd6cb557f7201036e9f43420650207b58
 runc:
  Version:          1.1.12
  GitCommit:        51d5e94601ceffbbd85688df1c928ecccbfa4685
 docker-init:
  Version:          0.19.0
  GitCommit:        
```



Example of no containers running on this host:

```sh
~ ➜  docker ps     
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

~ ➜  
```


Check docker images download on this node:

```sh
~ ➜  docker images
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
mysql                           latest    f6b0ca07d79d   5 months ago   934MB
postgres                        latest    a38f9f77ff88   5 months ago   456MB
alpine                          latest    706db57fb206   5 months ago   8.32MB
nginx                           alpine    5e7abcdd2021   5 months ago   52.7MB
nginx                           latest    657fdcd1c365   5 months ago   152MB
redis                           latest    466e5b1da2ef   6 months ago   137MB
ubuntu                          latest    97bed23a3497   6 months ago   78.1MB
kodekloud/simple-webapp-mysql   latest    129dd9f67367   7 years ago    96.6MB
kodekloud/simple-webapp         latest    c6e3cd9aae36   7 years ago    84.8MB
```


Notice that we have 09 containers:

```sh
~ ➜  docker images | wc -l
10
```


### Running a new container redis in detached mode


```sh
~ ➜  docker run -d redis  
58cc6b75624fd84ca302a8afec073f8e5162de898ca26b3fc12877254679e117

~ ➜  docker ps          
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS      NAMES
58cc6b75624f   redis     "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   6379/tcp   xenodochial_villani
```



### Stopping the container

```sh
~ ➜  docker stop xenodochial_villani 
xenodochial_villani

~ ➜  docker ps                      
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

~ ➜  
```


### Checking all container running

```sh
~ ➜  docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
f491dfed12fe   alpine         "sleep 1000"             24 seconds ago   Up 24 seconds             relaxed_beaver
132d512b79a5   nginx:alpine   "/docker-entrypoint.…"   25 seconds ago   Up 24 seconds   80/tcp    nginx-2
21eee384819f   nginx:alpine   "/docker-entrypoint.…"   26 seconds ago   Up 25 seconds   80/tcp    nginx-1
be3e4ca5598c   ubuntu         "sleep 1000"             27 seconds ago   Up 26 seconds             awesome_northcut
```


### checking all container running and stopped

```sh
~ ➜  docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                          PORTS     NAMES
a55b66a35a38   alpine         "/bin/sh"                47 seconds ago   Exited (0) 47 seconds ago                 hardcore_visvesvaraya
f491dfed12fe   alpine         "sleep 1000"             48 seconds ago   Up 48 seconds                             relaxed_beaver
132d512b79a5   nginx:alpine   "/docker-entrypoint.…"   49 seconds ago   Up 48 seconds                   80/tcp    nginx-2
21eee384819f   nginx:alpine   "/docker-entrypoint.…"   50 seconds ago   Up 49 seconds                   80/tcp    nginx-1
be3e4ca5598c   ubuntu         "sleep 1000"             51 seconds ago   Up 50 seconds                             awesome_northcut
58cc6b75624f   redis          "docker-entrypoint.s…"   2 minutes ago    Exited (0) About a minute ago             xenodochial_villani
```


Notice that the container called nginx-1 is using the image nginx:alpine
Also, notice that the container with image `ubuntu` is named `awesome_northcut` 
Still, notice that the container ID that uses `alpine` image and it is not running has ID a55b66a35a38 and it has the status `Exited` 


### Stopping all running containers

```sh
~ ➜  docker ps   
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
f491dfed12fe   alpine         "sleep 1000"             6 minutes ago   Up 6 minutes             relaxed_beaver
132d512b79a5   nginx:alpine   "/docker-entrypoint.…"   7 minutes ago   Up 6 minutes   80/tcp    nginx-2
21eee384819f   nginx:alpine   "/docker-entrypoint.…"   7 minutes ago   Up 6 minutes   80/tcp    nginx-1
be3e4ca5598c   ubuntu         "sleep 1000"             7 minutes ago   Up 7 minutes             awesome_northcut

~ ➜  docker stop relaxed_beaver nginx-2 nginx-1 awesome_northcut 
relaxed_beaver
nginx-2
nginx-1
awesome_northcut
```


```sh
~ ➜  docker ps                                                  
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

~ ➜  
```


### Deleting all containers stopped

```sh
~ ➜  docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                          PORTS     NAMES
a55b66a35a38   alpine         "/bin/sh"                8 minutes ago    Exited (0) 8 minutes ago                  hardcore_visvesvaraya
f491dfed12fe   alpine         "sleep 1000"             8 minutes ago    Exited (137) 57 seconds ago               relaxed_beaver
132d512b79a5   nginx:alpine   "/docker-entrypoint.…"   8 minutes ago    Exited (0) About a minute ago             nginx-2
21eee384819f   nginx:alpine   "/docker-entrypoint.…"   8 minutes ago    Exited (0) About a minute ago             nginx-1
be3e4ca5598c   ubuntu         "sleep 1000"             8 minutes ago    Exited (137) 57 seconds ago               awesome_northcut
58cc6b75624f   redis          "docker-entrypoint.s…"   10 minutes ago   Exited (0) 9 minutes ago                  xenodochial_villani
```


```sh
docker rm a55b66a35a38 f491dfed12fe 132d512b79a5 21eee384819f be3e4ca5598c 58cc6b75624f
```


### Deleting docker images

```sh
~ ➜  docker images        
REPOSITORY                      TAG       IMAGE ID       CREATED        SIZE
mysql                           latest    f6b0ca07d79d   5 months ago   934MB
postgres                        latest    a38f9f77ff88   5 months ago   456MB
alpine                          latest    706db57fb206   5 months ago   8.32MB
nginx                           latest    657fdcd1c365   5 months ago   152MB
nginx                           alpine    5e7abcdd2021   5 months ago   52.7MB
redis                           latest    466e5b1da2ef   6 months ago   137MB
ubuntu                          latest    97bed23a3497   6 months ago   78.1MB
kodekloud/simple-webapp-mysql   latest    129dd9f67367   7 years ago    96.6MB
kodekloud/simple-webapp         latest    c6e3cd9aae36   7 years ago    84.8MB
```


```sh
~ ➜  docker rmi ubuntu                  
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252
Deleted: sha256:97bed23a34971024aa8d254abbe67b7168772340d1f494034773bc464e8dd5b6
Deleted: sha256:073ec47a8c22dcaa4d6e5758799ccefe2f9bde943685830b1bf6fd2395f5eabc
```


### Pulling a docker image

```sh
~ ➜  docker pull nginx:1.14-alpine
1.14-alpine: Pulling from library/nginx
bdf0201b3a05: Pull complete 
3d0a573c81ed: Pull complete 
8129faeb2eb6: Pull complete 
3dc99f571daf: Pull complete 
Digest: sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
Status: Downloaded newer image for nginx:1.14-alpine
docker.io/library/nginx:1.14-alpine
```



### Creating a container with name webapp

```sh
~ ➜  docker run -d --name webapp nginx:1.14-alpine
f20f91f4cc1ff89f2178ff55e35a16c70b224204c1e886edabc6dfd472508ee9

~ ➜  docker ps                                    
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS     NAMES
f20f91f4cc1f   nginx:1.14-alpine   "nginx -g 'daemon of…"   5 seconds ago   Up 4 seconds   80/tcp    webapp
```


### Stopping and deleting a container webapp

```sh
~ ➜  docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED              STATUS              PORTS     NAMES
f20f91f4cc1f   nginx:1.14-alpine   "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp    webapp

~ ➜  docker stop webapp                                         
webapp
```



```sh
~ ➜  docker rm webapp                             
webapp
```


### Deleting all images

```sh
~ ➜  docker rmi $(docker images | awk '{print $1}' | grep -vi reposit)
```


```sh
~ ➜  docker rmi 5e7abcdd2021 8a2fb25a19f5 
```


```sh
~ ➜  docker images                        
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

~ ➜  
```



## Docker Run


### Docker tag

When we run docker run redis it is always taking in consideration the :latest tag which is the latest version. We can specify a previously version of the image.

```sh
docker run redis:latest
docker run redis:7.2
docker run redis:6
```


check in https://hub.docker.com/ for more images version.



### docker run -it

As default docker image doesn't listen for INPUT or send OUTPUT to default screen.
We need to use the -i for interactive and -t for terminal to attach terminal to default screen.

```sh
docker run -it kodekloud/simple-prompt-docker
```

This way we attached to terminal in an interactive mode.


### docker run PORT mapping 


As default the docker has internal IP that allows the container to be accessed from the host it self. 
To allow a container be accessed outside of the docker host we need map the port inside of the container to the docker host.

```sh
docker run -p 80:5000 kodekloud/webapp
docker run -p 8080:5000 kodekloud/webapp
docker run -p 8081:5000 kodekloud/webapp
docker run -p 3306:3306 mysql
```

You can not bind a port twice in the host level.


### Docker run Volume mapping

As default the data in the containers are not persistent. If we delete a container and recreate it won't have the data persistent on it. We lose the data when we delete a container.

To allow the data to be persistent we need to map a docker host directory into a directory in the container. Everything is saved in this container directory will be persistent, although everything saved outside of that directory will be wiped when a container is deleted.

```sh
docker run -v /opt/datadir:/var/lib/mysql mysql
```

Everything saved within the container in /var/lib/mysql will be actually be stored in the external volume /opt/datadir in the docker host even if the pod is deleted.

### Docker inspect

```sh
docker inspect container_name_or_ID
```


### Docker logs

```sh
docker logs container_name_or_ID
```



## Docker run - Advanced commands 



### Docker run image version


```sh
admin@docker-host ➜  docker run ubuntu cat /etc/os-release
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
817807f3c64e: Pull complete 
Digest: sha256:186072bba1b2f436cbb91ef2567abca677337cfc786c86e107d25b7072feef0c
Status: Downloaded newer image for ubuntu:latest
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```


```sh
admin@docker-host ➜  docker run ubuntu:22.04 cat /etc/os-release
Unable to find image 'ubuntu:22.04' locally
22.04: Pulling from library/ubuntu
de47083ed7d7: Pull complete 
Digest: sha256:eb29ed27b0821dca09c2e28b39135e185fc1302036427d5f4d70a41ce8fd7659
Status: Downloaded newer image for ubuntu:22.04
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```


```sh
admin@docker-host ✖ docker run tetchel/timer
Unable to find image 'tetchel/timer:latest' locally
latest: Pulling from tetchel/timer
57df1a1f1ad8: Pull complete 
71e126169501: Pull complete 
1af28a55c3f3: Pull complete 
03f1c9932170: Pull complete 
65b3db15f518: Pull complete 
3e3b8947ed83: Pull complete 
f156949921a1: Pull complete 
1c1931013093: Pull complete 
51fff639b6bf: Pull complete 
e0c7f3a6f9f6: Pull complete 
Digest: sha256:785d015cbe20ea46c1e43c3bb1952614dc78d55ceef18eca624c9006748e4165
Status: Downloaded newer image for tetchel/timer:latest
Counting for 10s
It's 04/04/2026 16:09:51
It's 04/04/2026 16:09:52
It's 04/04/2026 16:09:53
It's 04/04/2026 16:09:54
It's 04/04/2026 16:09:55
It's 04/04/2026 16:09:56
It's 04/04/2026 16:09:57
It's 04/04/2026 16:09:58
It's 04/04/2026 16:09:59
It's 04/04/2026 16:10:00
Done counting!
```


### Jenkins image for docker


```sh
docker run -d --name jenkins-server -p 8081:8080 -p 50000:50000   -v jenkins-data:/var/jenkins_home jenkins/jenkins
```


```
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure jenkins/jenkins
```



### Lab

Notice that we have just one container running

```sh
~ ➜  docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                                                NAMES
52f8e066c3d1   nginx:alpine   "/docker-entrypoint.…"   44 seconds ago   Up 43 seconds   0.0.0.0:3456->3456/tcp, :::3456->3456/tcp, 0.0.0.0:38080->80/tcp, :::38080->80/tcp   brave_maxwell
```



Notice that we have nginx image for that container

```sh
~ ➜  docker inspect 52f8e066c3d1 | grep -i "image"
        "Image": "sha256:5e7abcdd20216bbeedf1369529564ffd60f830ed3540c477938ca580b645dff5",
            "Image": "nginx:alpine",
```



Lets check how many ports are open in this container, notice that we have 02:


```sh
            "Ports": {
                "3456/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "3456"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "3456"
                    }
                ],
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "38080"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "38080"
                    }
                ]
            },
```


The open ports in the container are 3486 and 80
The open ports in the host are 3456 and 38080


### Docker run in detached mode


Lets create a new container, Run an instance of `kodekloud/simple-webapp:blue` and name the container `blue-app`, mapping port `8080` on the container to port `38282` on the host

```sh
docker run -d --name blue-app -p 38282:8080 kodekloud/simple-webapp:blue
```



## Docker images - built 


### How to create your own image


Lets create our manual steps first:

```
1. OS Ubuntu 
2. Update apt repo
3. Install dependencies using apt
4. Install python dependencies using pip
5. Copy source code to /opt folder
6. Run web server using "flask" command.
```



Lets now create our Dockerfile with the mentioned commands:

Dockerfile
```yaml
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```


To create your image run:

```sh
docker build Dockerfile -t degutos/my-custom-app
```


To make this image available in docker hub run:

```sh
docker push degutos/my-custom-app
```



To check all layers built use:

```sh
docker history degutos/my-custom-app
```



## Creating new docker image 


Repo with instructions: 
https://github.com/mmumshad/simple-webapp-flask


We may do a similar exercise that the one demonstrated in this above github page.
For this lets start a docker container:

### Docker run

```sh
docker run -p 5000:5000 -it ubuntu bash
```


### Installing packages and dependencies within the container

```sh
apt-get install -y python3 python3-setuptools python3-dev build-essential python3-pip default-libmysqlclient-dev
apt install python3.12-venv
```


#### Setup python venv

```sh
python3 -m venv venv
source venv/bin/activate
```


#### Installing pip dependencies

```sh
pip3 install flask
pip3 install flask-mysql
```


#### Creating app.py

```sh
cd /opt
cat > app.py
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
CTRL+C
```

When we CTRL+C we create this file called app.py. Make sure your file is set to port 5000.


#### Running Flask app.py website

```sh
FLASK_APP=app.py flask run --host=0.0.0.0
```


Once we open our flask website we can access on port 5000 on our browser.


### Creating a dockerfile

```sh
cat > Dockerfile
FROM ubuntu

RUN apt update
RUN apt install -y python3 python3-setuptools python3-dev build-essential python3-pip default-libmysqlclient-dev python3.12-venv


RUN pip3 install flask --break-system-packages
RUN pip3 install flask-mysql --break-system-packages

COPY app.py /opt/app.py

ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0
CTRL+C
```


### Creating app.py

```sh
cat > app.py
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


Lets confirm our files:

```sh
root@docker-host ~/my-simple-webapp via 🐍 ➜  ls
Dockerfile  app.py
```

### Building our image

```sh
root@docker-host ~/my-simple-webapp via 🐍 ➜  docker build . -t my-simple-webapp:latest
[+] Building 0.0s (11/11) FINISHED                                                                                                                                                docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                        0.0s
 => => transferring dockerfile: 382B                                                                                                                                                        0.0s
 => WARN: JSONArgsRecommended: JSON arguments recommended for ENTRYPOINT to prevent unintended behavior related to OS signals (line 12)                                                     0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                                                                           0.0s
 => => transferring context: 2B                                                                                                                                                             0.0s
 => [1/6] FROM docker.io/library/ubuntu:latest                                                                                                                                              0.0s
 => [internal] load build context                                                                                                                                                           0.0s
 => => transferring context: 28B                                                                                                                                                            0.0s
 => CACHED [2/6] RUN apt update                                                                                                                                                             0.0s
 => CACHED [3/6] RUN apt install -y python3 python3-setuptools python3-dev build-essential python3-pip default-libmysqlclient-dev python3.12-venv                                           0.0s
 => CACHED [4/6] RUN pip3 install flask --break-system-packages                                                                                                                             0.0s
 => CACHED [5/6] RUN pip3 install flask-mysql --break-system-packages                                                                                                                       0.0s
 => CACHED [6/6] COPY app.py /opt/app.py                                                                                                                                                    0.0s
 => exporting to image                                                                                                                                                                      0.0s
 => => exporting layers                                                                                                                                                                     0.0s
 => => writing image sha256:5a80946226155bbac64b8e50598b637605babaf082672b9ca8d324364e4af8e2                                                                                               0.0s
 => => naming to docker.io/library/my-simple-webapp:latest                                                                                                                                  0.0s
 1 warning found (use docker --debug to expand):
 - JSONArgsRecommended: JSON arguments recommended for ENTRYPOINT to prevent unintended behavior related to OS signals (line 12)

```


### Listing our images

```sh
docker images
```


### Running new container with new image created

```sh
docker run -p 5000:5000 my-simple-webapp
```


### Pushing image to docker rub

If you want to push your image to docker hub you will need to build your image with your docker hub username in front of your image name, example:

```sh
docker build . -t degutos/my-simple-webapp
```

Your new image will have the name of `degutos/my-simple-webapp` 

Then we can push the image to dockerhub 

```sh
docker login
```


```sh
docker push degutos/my-simple-webapp
```


### Another example of Dockerfile

```sh
~/webapp-color via 🐍 ➜  cat Dockerfile 
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```


#### docker build

```sh
~/webapp-color via 🐍 ➜  docker build . -t webapp-color
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            BuildKit is currently disabled; enable it by removing the DOCKER_BUILDKIT=0
            environment-variable.

Sending build context to Docker daemon  8.704kB
Step 1/6 : FROM python:3.6
3.6: Pulling from library/python
0e29546d541c: Pull complete 
9b829c73b52b: Pull complete 
cb5b7ae36172: Pull complete 
6494e4811622: Pull complete 
6f9f74896dfa: Pull complete 
5e3b1213efc5: Pull complete 
9fddfdc56334: Pull complete 
404f02044bac: Pull complete 
c4f42be2be53: Pull complete 
Digest: sha256:f8652afaf88c25f0d22354d547d892591067aa4026a7fa9a6819df9f300af6fc
Status: Downloaded newer image for python:3.6
 ---> 54260638d07c
Step 2/6 : RUN pip install flask
 ---> Running in 90cdee70a168
Collecting flask
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting click>=7.1.2
  Downloading click-8.0.4-py3-none-any.whl (97 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Collecting importlib-metadata
  Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp36-cp36m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (30 kB)
Collecting dataclasses
  Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Collecting zipp>=0.5
  Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, importlib-metadata, dataclasses, Werkzeug, Jinja2, itsdangerous, click, flask
Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 importlib-metadata-4.8.3 itsdangerous-2.0.1 typing-extensions-4.1.1 zipp-3.6.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.4; however, version 21.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
 ---> Removed intermediate container 90cdee70a168
 ---> b42ba6073cfe
Step 3/6 : COPY . /opt/
 ---> fe476a552a78
Step 4/6 : EXPOSE 8080
 ---> Running in db573ade1ca0
 ---> Removed intermediate container db573ade1ca0
 ---> b346a404c4f5
Step 5/6 : WORKDIR /opt
 ---> Running in e404177b197d
 ---> Removed intermediate container e404177b197d
 ---> 9ea0f8b1b0fa
Step 6/6 : ENTRYPOINT ["python", "app.py"]
 ---> Running in c7c6a4933f92
 ---> Removed intermediate container c7c6a4933f92
 ---> e314d68dab17
Successfully built e314d68dab17
Successfully tagged webapp-color:latest
```


#### Running container with new image created on port 8282 of the host to 8080 on container

```sh
docker run -p 8282:8080 webapp-color
```


### Dockerfile with small Alpine image


```sh
FROM python:alpine3.22

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```




## Environment Variables


We can have an application in Python for example that has a container that displays a simple website in background color as red. If this configuration is set and hardcoded into the application `color = "red"`, and if you need anytime to change that configuration to a different color, you 'll need to change your application code to different color and spin up a another container. 
It's a best practice to set your application code to use an external variable in the OS for this configuration then we can change that color any time.

```sh
color = "red"
```


to 

```sh
color = os.environ.get('APP_COLOR')
```

Then

```sh
export APP_COLOR=blue ; python app.py
```


To create a container with different color in execution time

```sh
docker run -e APP_COLOR=blue simple-webapp-color
```

OR

```sh
docker run -e APP_COLOR=green simple-webapp-color
```

OR 

``` 
docker run -e APP_COLOR pink simple-webapp-color
```


### Inspect environment variables


```sh
docker inspect container_id 
```



## Environment Variable - Lab

```sh
~ ➜  docker ps                                    
CONTAINER ID   IMAGE                     COMMAND           CREATED          STATUS          PORTS      NAMES
e765c8918b89   kodekloud/simple-webapp   "python app.py"   24 minutes ago   Up 24 minutes   8080/tcp   goofy_easley
```



```sh
~ ➜  docker inspect e765c8918b89 | grep -i env -A3    
            "Env": [
                "APP_COLOR=pink",
                "PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
```


Notice that the running container has the ENV VARIABLE like APP_COLOR set to pink.


### Exposing new app with different environment variable set


Lets create a container called blue-app using the image kodekloud/simple-webapp and expose port 8080 to port 38282 on the node.

```sh
docker run -p 38282:8080 -e APP_COLOR=blue  --name blue-app kodekloud/simple-webapp
```




### Exposing mysql database with password db set up


```sh
~ ➜  docker run --name mysql-db -e MYSQL_ROOT_PASSWORD="db_pass123" mysql     
```



## Commands vs Entrypoint


When we run `docker run ubuntu` the container runs and dies. 
Container is supposed to live just for the moment they need a specific task, it is different than virtual machine.

If we run a parameter bash it needs an interaction terminal with option `-it` 

If the website running on container stops, the container will crash and exit.

The `CMD` in Dockerfile defines the command the container has to run within the container.

```sh
docker run ubuntu [COMMAND]
```


Example:

```sh
docker run ubuntu sleep 5
```


There are two ways to use CMD

```
FROM Ubuntu

CMD sleep 5
```


or

```sh
FROM Ubuntu

CMD ["sleep","5"]
```


### Recommended way to set that CMD and ENTRYPOINT


```sh
From Ubuntu

ENTRYPOINT ["sleep"]
CMD ["5"]
```

Then we create our container

```sh
docker run ubuntu-sleeper
```

or 

```sh
docker run ubuntu-sleeper 20
```




### Example of Dockerfile

```sh
~ ➜  cat Dockerfile-ubuntu 
#
# Ubuntu Dockerfile
#
# https://github.com/dockerfile/ubuntu
#

# Pull base image.
FROM ubuntu:14.04

# Install.
RUN \
  sed -i 's/# \(.*multiverse$\)/\1/g' /etc/apt/sources.list && \
  apt-get update && \
  apt-get -y upgrade && \
  apt-get install -y build-essential && \
  apt-get install -y software-properties-common && \
  apt-get install -y byobu curl git htop man unzip vim wget && \
  rm -rf /var/lib/apt/lists/*

# Add files.
ADD root/.bashrc /root/.bashrc
ADD root/.gitconfig /root/.gitconfig
ADD root/.scripts /root/.scripts

# Set environment variables.
ENV HOME /root

# Define working directory.
WORKDIR /root

# Define default command.
CMD ["bash"]

```



Another sample

```sh
~ ➜  cat Dockerfile-mysql 
FROM debian:stretch-slim

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr && rm -rf /var/lib/apt/lists/*

# add gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
        && apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
        && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
        && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
        && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
        && gpgconf --kill all \
        && rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
        && chmod +x /usr/local/bin/gosu \
        && gosu nobody true \
        && apt-get purge -y --auto-remove ca-certificates wget

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-get update && apt-get install -y --no-install-recommends \
# for MYSQL_RANDOM_ROOT_PASSWORD
                pwgen \
# for mysql_ssl_rsa_setup
                openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
                perl \
        && rm -rf /var/lib/apt/lists/*

RUN set -ex; \
# gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
        key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
        export GNUPGHOME="$(mktemp -d)"; \
        gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
        gpg --batch --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
        gpgconf --kill all; \
        rm -rf "$GNUPGHOME"; \
        apt-key list > /dev/null

ENV MYSQL_MAJOR 8.0
ENV MYSQL_VERSION 8.0.17-1debian9

RUN echo "deb http://repo.mysql.com/apt/debian/ stretch mysql-${MYSQL_MAJOR}" > /etc/apt/sources.list.d/mysql.list

# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
                echo mysql-community-server mysql-community-server/data-dir select ''; \
                echo mysql-community-server mysql-community-server/root-pass password ''; \
                echo mysql-community-server mysql-community-server/re-root-pass password ''; \
                echo mysql-community-server mysql-community-server/remove-test-db select false; \
        } | debconf-set-selections \
        && apt-get update && apt-get install -y mysql-community-client="${MYSQL_VERSION}" mysql-community-server-core="${MYSQL_VERSION}" && rm -rf /var/lib/apt/lists/* \
        && rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
        && chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
        && chmod 777 /var/run/mysqld

VOLUME /var/lib/mysql
# Config files
COPY config/ /etc/mysql/
COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]

```




See another sample

```sh
~ ➜  cat Dockerfile-python 
#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "update.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM buildpack-deps:buster

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# extra dependencies (over what buildpack-deps already includes)
RUN apt-get update && apt-get install -y --no-install-recommends \
                tk-dev \
                uuid-dev \
        && rm -rf /var/lib/apt/lists/*

ENV GPG_KEY E3FF2839C048B25C084DEBE9B26995E310250568
ENV PYTHON_VERSION 3.8.0b3

RUN set -ex \
        \
        && wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
        && wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
        && gpg --batch --verify python.tar.xz.asc python.tar.xz \
        && { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
        && rm -rf "$GNUPGHOME" python.tar.xz.asc \
        && mkdir -p /usr/src/python \
        && tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
        && rm python.tar.xz \
        \
        && cd /usr/src/python \
        && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
        && ./configure \
                --build="$gnuArch" \
                --enable-loadable-sqlite-extensions \
                --enable-optimizations \
                --enable-shared \
                --with-system-expat \
                --with-system-ffi \
                --without-ensurepip \
        && make -j "$(nproc)" \
        && make install \
        && ldconfig \
        \
        && find /usr/local -depth \
                \( \
                        \( -type d -a \( -name test -o -name tests \) \) \
                        -o \
                        \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
                \) -exec rm -rf '{}' + \
        && rm -rf /usr/src/python \
        \
        && python3 --version

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
        && ln -s idle3 idle \
        && ln -s pydoc3 pydoc \
        && ln -s python3 python \
        && ln -s python3-config python-config

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 19.2.1
# https://github.com/pypa/get-pip
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/404c9418e33c5031b1a9ab623168b3e8a2ed8c88/get-pip.py
ENV PYTHON_GET_PIP_SHA256 56bb63d3cf54e7444351256f72a60f575f6d8c7f1faacffae33167afc8e7609d

RUN set -ex; \
        \
        wget -O get-pip.py "$PYTHON_GET_PIP_URL"; \
        echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum --check --strict -; \
        \
        python get-pip.py \
                --disable-pip-version-check \
                --no-cache-dir \
                "pip==$PYTHON_PIP_VERSION" \
        ; \
        pip --version; \
        \
        find /usr/local -depth \
                \( \
                        \( -type d -a \( -name test -o -name tests \) \) \
                        -o \
                        \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
                \) -exec rm -rf '{}' +; \
        rm -f get-pip.py

CMD ["python3"]#                                                                 

```


And another one

```sh
FROM php:7.1-apache

# install the PHP extensions we need (https://make.wordpress.org/hosting/handbook/handbook/server-environment/#php-extensions)
RUN set -ex; \
        \
        savedAptMark="$(apt-mark showmanual)"; \
        \
        apt-get update; \
        apt-get install -y --no-install-recommends \
                libjpeg-dev \
                libmagickwand-dev \
                libpng-dev \
        ; \
        \
        docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
        docker-php-ext-install -j "$(nproc)" \
                bcmath \
                exif \
                gd \
                mysqli \
                opcache \
                zip \
        ; \
        pecl install imagick-3.4.4; \
        docker-php-ext-enable imagick; \
        \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
        apt-mark auto '.*' > /dev/null; \
        apt-mark manual $savedAptMark; \
        ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
                | awk '/=>/ { print $3 }' \
                | sort -u \
                | xargs -r dpkg-query -S \
                | cut -d: -f1 \
                | sort -u \
                | xargs -rt apt-mark manual; \
        \
        apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
        rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
                echo 'opcache.memory_consumption=128'; \
                echo 'opcache.interned_strings_buffer=8'; \
                echo 'opcache.max_accelerated_files=4000'; \
                echo 'opcache.revalidate_freq=2'; \
                echo 'opcache.fast_shutdown=1'; \
        } > /usr/local/etc/php/conf.d/opcache-recommended.ini
# https://wordpress.org/support/article/editing-wp-config-php/#configure-error-logging
RUN { \
# https://www.php.net/manual/en/errorfunc.constants.php
# https://github.com/docker-library/wordpress/issues/420#issuecomment-517839670
                echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
                echo 'display_errors = Off'; \
                echo 'display_startup_errors = Off'; \
                echo 'log_errors = On'; \
                echo 'error_log = /dev/stderr'; \
                echo 'log_errors_max_len = 1024'; \
                echo 'ignore_repeated_errors = On'; \
                echo 'ignore_repeated_source = Off'; \
                echo 'html_errors = Off'; \
        } > /usr/local/etc/php/conf.d/error-logging.ini

RUN a2enmod rewrite expires

VOLUME /var/www/html

ENV WORDPRESS_VERSION 5.2.2
ENV WORDPRESS_SHA1 3605bcbe9ea48d714efa59b0eb2d251657e7d5b0

RUN set -ex; \
        curl -o wordpress.tar.gz -fSL "https://wordpress.org/wordpress-${WORDPRESS_VERSION}.tar.gz"; \
        echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c -; \
# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
        tar -xzf wordpress.tar.gz -C /usr/src/; \
        rm wordpress.tar.gz; \
        chown -R www-data:www-data /usr/src/wordpress

COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["apache2-foreground"]

```


### Running ubuntu image with sleep 1000s 


```sh
docker run -d ubuntu sleep 1000
45a75a4135b7564b1339bfe2b811220abc299cf4a6b7ca070a41b1832e29ccde
```



## Docker Compose

If you need spin up a few or many docker containers, you can create a docker compose yaml file with a recipe of all container you want to spin up. You can use `docker-compose up`  command to spin up all container mentioned in the docker compose yaml file. This is only possible when running all containers in a single docker host.


### Voting application - sample application


- voting-app
- in-memory DB
- worker
- db
- result-app

Lets consider all this containers working as micro-services.

- voting-app - web application based in python responsible to user to vote in dogs or cats
- in-memory DB - it's a container running in memory a DB called REDIS.
- worker - it is a container responsible to read from Redis and save into a persistent DB
- db - it is persistent database running postgresql. 
- result-app - it is web application in NodeJs just to display the results.


#### docker run


```
docker run -d --name=redis redis
docker run -d --name=db postgres postgres:9:4
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
docker run -d --name=result -p 50001:80 --link db:db result-app
docker run -d --name=worker --link db:db --link redis:redis worker
```

Although this method is deprecated 



docker-compose.yaml
```yaml
redis:
   image: redis
db:
   image: postgresql:9.4
  
vote:
   image: voting-app
   ports:
     - 5000:80
    links:
      - redis
result:
   image: result-app
   ports:
     - 5001:80
    links:
      - db

worker:
   image: worker
   links:
     - redis
     - db
```


```sh
docker-compose up
```


#### Building an image during the docker-compose

In the docker-compose file we refer to `image` if the image is stored in our github repository. Not always we will have the image created and stored in github repo.
In our example we may have redis and postgres already stored in the github repository, but the votin-app, result and worker are in-house developed app. Lets consider if we want build the image during the docker compose up, we can refer to build variable in yaml file.

docker-compose.yaml
```yaml
redis:
   image: redis
db:
   image: postgresql:9.4
  
vote:
   build: ./vote
   ports:
     - 5000:80
    links:
      - redis
result:
   build: ./result
   ports:
     - 5001:80
    links:
      - db

worker:
   build: ./worker
   links:
     - redis
     - db
```


#### Docker versions

The docker-compose.yaml file we created is recognized as version:1 of the docker compose. We may have different versions like version 2 and version 3.

##### docker compose version-2

In version-2 we can separate each container with different network. In our example we will create two networks: front-end and back-end.
We can also add a variable `depends-on`


```yaml
version: 2
services:
	redis:
	   image: redis
	networks:
	  - back-end
	    
	db:
	   image: postgresql:9.4
	   networks:
	     - back-end
	  
	vote:
	   build: ./vote
	   ports:
	     - 5000:80
	    links:
	      - redis
	    depends_on:
	      - redis
	    networks:
	      - front-end
	      - back-end
			
	result:
	   build: ./result
	   ports:
	     - 5001:80
	    links:
	      - db
	    networks:
	      - front-end
	      - back-end
	
	worker:
	   build: ./worker
	   links:
	     - redis
	     - db
	       
networks:
   front-end:
   back-end:	       
```



### Voting app from github repo

REPO: https://github.com/dockersamples/example-voting-app#

Clone with: git repo https://github.com/dockersamples/example-voting-app.git 

Then we can deploy our containers with:

```sh
docker-compose up
```


## Docker compose

Lets create a docker container running redis image

```sh
docker run -d --name=redis redis:alpine 
```


Lets create a container called click-counter exposing port 8085 on the node, the app runs on port 5000:

```sh
docker run -d --name=clickcounter -p 8085:5000 --link redis:redis kodekloud/
click-counter
```


We can access the app through the browser on port 8085

Check the container running and stop them:

```sh
~ ➜  docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                       NAMES
b9f4e9642877   kodekloud/click-counter   "flask run"              7 minutes ago    Up 7 minutes    0.0.0.0:8085->5000/tcp, :::8085->5000/tcp   clickcounter
83eaf036a25d   redis:alpine              "docker-entrypoint.s…"   13 minutes ago   Up 13 minutes   6379/tcp                                    redis

~ ➜  docker stop b9 83
b9
83
```


Lets delete the containers:

```sh
~ ➜  docker rm 83eaf036a25d clickcounter 
83eaf036a25d
clickcounter
```


### Create a docker-compose yaml file 

Create a `docker-compose.yml` file under the directory `/root/clickcounter`. Once done, run `docker-compose up`.

The compose file should have the exact specification as follows -  

1. `redis` service specification - Image name should be `redis:alpine`.
2. `clickcounter` service specification - Image name should be `kodekloud/click-counter`, app is run on port `5000` and expose it on the host port `8085` in the compose file.

`docker-compose.yml`
```yml
version: "3"
services: 
  redis:
    image: redis:alpine

  clickcounter:
    image: kodekloud/click-counter
    ports:
      - "8085:5000"
```


See more details in the docummentation:

https://docs.docker.com/compose/
https://docs.docker.com/reference/cli/docker/compose/


## Docker Engine and Docker storage

Docker engine components

- docker cli
- docker api
- docker daemon

The docker api can run in different node and access the docker api through the network

```sh
docker -H=remote_docker_engine:2375 run nginx 
```


### Containerization 

- Namespace:
	- process_id
	- network
	- mount 
	- unix timesharing
	- interprocess 

Docker container uses the host resources and usually has no limmit set but we can limit the amount of CPU and MEM a container uses

```sh
docker run --cpu=.5 ubuntu # limit container to use 50% of cpu 
docker run --memory=100m ubuntu # limit container to use 100Mb of memory 
```



## Docker Storage


Docker storage its data in:
- /var/lib/docker
	- volumes
	- container
	- images

- /var/lib/docker
	- volumes
		- data_volume

To create a volume:

```sh
docker run -v data_volume:/var/lib/mysql mysql
```


If you set a volume with the docker run and the volume folder doesn't exist the docker will create that folder for you.

If we have a storage folder different than the default one, like /data, we can set that with -v option, example:

```sh
docker run -v /data/mysql:/var/lib/mysql mysql
```


The -v is actually the deprecated option, we have another another way of doing that. We can use --mount parameter, example:

```sh
docker run \
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```


### Storage drivers

- AUFS
- ZFS
- BTRFS
- Device Mapper
- Overlay
- Overlay2



## Docker storage Lab


What location are the files related to the docker containers and images stored?

```sh
/var/lib/docker
```


```sh
/var/lib/docker ➜  ls
buildkit    containers  image       overlay2    runtimes    tmp
containerd  engine-id   network     plugins     swarm       volumes
```


What directory under `/var/lib/docker` are the files related to the container `alpine-3` image stored?

```sh
~ ➜  docker ps -a | grep alpine-3
9d4e229bfd4f   alpine    "/bin/sh"   6 minutes ago   Exited (0) 6 minutes ago             alpine-3
```

Notice that our container starts with `9d4e229bfd4f`

Then we can check 

```sh
lib/docker/containers ➜  ls
148167916ea14c75feff1c81afd05f01dab266bf7f3c21fe45a08498b47fecc9
7049f00783e094e1a29f5132f992c51156ecd009a794e6010d3f906aa059ea83
9d4e229bfd4f1654bef1d93ec65dd39bde8e1f004602796b07fee7d5dc77b6cc
```


Run a `mysql` container named `mysql-db` using the `mysql` image. Set database password to `db_pass123`

```sh
~ ➜  docker run -d -e MYSQL_ROOT_PASSWORD=db_pass123 --name mysql-db mysql
0e42069100a9b0fc88f39288b4f5ca13d26f1d49f047372e09becc82ba2c60f0
```



Lets get details from the database

```sh
~ ➜  cat get-data.sh 
docker exec mysql-db mysql -pdb_pass123 -e 'use foo; select * from myTable'
```


Notice that we have this get-data.sh script file and we can use it to gather data from the database

```sh
~ ➜  sh get-data.sh 
id      Name    Phone   Email
1       Kareem  130-5655        Duis.volutpat.nunc@quamCurabitur.org
2       Ruby    1-584-149-0770  Nulla.tempor@vitaeorciPhasellus.org
3       Rowan   199-8663        consectetuer.adipiscing.elit@Sedmalesuada.co.uk
4       Alisa   220-6017        elementum.sem.vitae@enimMauris.edu
5       Ella    731-0337        fermentum@nec.net
6       Tiger   658-4480        quis.diam@odiovelest.net
7       Felix   1-274-848-3378  Mauris.vel@arcu.com
8       Karina  1-390-796-3451  sagittis.semper@odioapurus.co.uk
9       Davis   605-8539        venenatis.vel@risusDonecnibh.com
10      Mohammad        1-590-174-1489  ornare.sagittis.felis@natoque.ca
11      Zane    362-1770        Aenean.euismod@condimentum.co.uk
12      Piper   1-231-386-6903  nunc.sed.pede@nascetur.ca
13      Marshall        1-383-729-4990  Cras.interdum.Nunc@neceuismod.ca
14      Zena    241-6641        Fusce.mollis.Duis@lobortis.org
15      Abdul   1-748-387-9935  eget.lacus.Mauris@Crasvehicula.com
16      Chase   1-401-241-9169  ante.dictum.mi@nascetur.org
17      Zahir   921-0663        non@nonummyutmolestie.edu
18      Brenda  1-691-909-5827  Quisque.ac@magnaCras.co.uk
19      Laura   1-562-983-9565  Quisque.ornare.tortor@sollicitudinadipiscing.ca
20      Madison 1-348-737-0587  Quisque.varius@Intinciduntcongue.org
21      Tanek   991-6278        dignissim.magna@Pellentesqueutipsum.net
22      Dakota  893-0792        Nullam.enim.Sed@nulla.net
23      Boris   1-297-302-5792  non.sollicitudin@eleifendegestasSed.co.uk
24      Celeste 723-6729        mauris.rhoncus@eunulla.edu
25      Connor  1-203-901-7531  et@loremipsumsodales.edu
26      Perry   1-756-607-9187  eros.turpis@tristiquepharetra.co.uk
27      Hayfa   1-609-407-3019  non.lobortis.quis@malesuadafringilla.net
28      Todd    343-0454        id.erat@arcu.org
29      Fuller  881-7273        non.feugiat.nec@adipiscingelit.net
30      Rama    1-927-605-0610  nonummy.ultricies.ornare@malesuada.co.uk
mysql: [Warning] Using a password on the command line interface can be insecure.
```


Lets check the container now, notice that the container is not running any more:

```sh
~ ✖ docker ps                                                            
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```

the script will fail:

```sh
~ ➜  sh get-data.sh 
Error response from daemon: No such container: mysql-db
```


Run a mysql container again, but this time map a volume to the container so that the data stored by the container is stored at `/opt/data` on the host.
Use the same name : `mysql-db` and same password: `db_pass123` as before. Mysql stores data at `/var/lib/mysql` inside the container.


```sh
docker run -d -e MYSQL_ROOT_PASSWORD=db_pass123 -v /opt/data:/var/lib/mysql --name mysql-db mysql
```


Now even if our container die we can recreate it with the same volume /opt/data in the node and mount it to /var/lib/mysql in the container


## Networking

- bridge
- none
- host

```sh
docdker run ubuntu
docker run ubuntu --network=none
docker run ubuntu --network=host
```


By default all containers are attached to the `bridge` network and all container get an ip usually under network 172.17.x
To access this container externally we need map a port in the node to a port to the container.

The option --network=host allows you to set a port in the host directly for example the port 5000 in the container will use the port 5000 in the host. That means we won't be able to run another container in the same port.

The option --network=none we are attaching the container to any network, it is isolated container.

By default docker creates only one network, although we can create a second network example:
- 172.17
- 182.18

```sh
docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-netwwork 
docker network 
```


Lets use inspect option to inspect the container network



## Docker Network Lab


```sh
~ ➜  docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
e47af24dea6d   bridge    bridge    local
c9ae16eb7e15   host      host      local
b90d714f40ac   none      null      local
```


Lets check the container network mode:

```sh
~ ➜  docker inspect alpine-1 | grep -i networkmode
            "NetworkMode": "host",
```

Lets run another container with network none:

```sh
~ ➜  docker run -d --name alpine-2 --network none alpine
b4b826c2f7d222cc022cdd27fe1a99aa75247dcda0d577c456bdd4f256e11a0b
```



Lets create another network

```sh
~ ➜  docker network create --driver bridge --subnet 182.18.0.0/24 --gateway 182.18.0.1  wp-mysql-network 
be40a899bc1cf95dc2665e8bf51f2c6522a686d853c3db33b42c5fc5054e93a2
```

```sh
~ ➜  docker run -d --name mysql-db --network wp-mysql-network  mysql:5.7
```


Lets create another container

```sh
~ ➜  docker run -d --name webapp -p 38080:8080 -e DB_Host=mysql-db -e DB_Password=db_pass123 --network wp-mysql-network kodekloud/simple-webapp-mysql 
cb6341722066be1b2854ecb3bf81a418b17e285ebc96dd32e2ba7e97518fcdde
```


## Docker registry 


Lets create our own registry locally, Let practice deploying a `registry server` on our own.  
Run a registry server with name equals to `my-registry` using `registry:2` image with host port set to `5000`, and restart policy set to `always`.  
Note: Registry server is exposed on port `5000` in the image.


```sh
 docker run -d -p 5000:5000 --restart=always  --name my-registry registry:2
```


Lets now check the registry container running

```sh
docker ps                                                                
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                                       NAMES
4d7fd31cfc3b   registry:2   "/entrypoint.sh /etc…"   53 seconds ago   Up 53 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   my-registry
```


Now lets pull 02 images

```sh
docker pull nginx:latest 
docker pull httpd:latest
```


Check the local repository 

```sh
curl -X GET localhost:5000/v2/_catalog
{"repositories":[]}
```


Lets push now

```sh
docker image tag nginx:latest localhost:5000/nginx:latest 
docker push localhost:5000/nginx:latest 
```

Lets now push httpd image

```sh
docker image tag httpd:latest localhost:5000/httpd:latest 
docker push localhost:5000/httpd:latest  
```


Lets now check the images in our local repository

```sh
curl -X GET localhost:5000/v2/_catalog
{"repositories":["httpd","nginx"]}
```


Now lets remove all images

```sh
docker image prune -a 
```



## Docker Swarm 


- docker swarm manager
- docker swarm host worker
- docker swarm host worker


```sh
docker swarm init --advertise-addr 192.168.1.12
```


```sh
docker swarm join --token SMITHUR-REDACTED-xxx
```


Note: run the join command across all worker

```sh
docker service create --replicas=3 my-web-server
```

Docker service create must be ran in the manager docker swarm 

```sh
docker service create --replicas=3 -p 8080:80 my-web-server
docker service create --replicas=3 --network frontend my-web-server
```



