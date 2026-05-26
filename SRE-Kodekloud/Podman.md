
## Container podman

### Installation


On redhat-10:

```sh
dnf install -y podman
```


### Playground

```sh
podman run -it --rm registry.access.redhat.com/ubi10/ubi
```

```sh
podman run -d --name nginx -p 8080:80 docker.io/nginx
```

```sh
[root@rhel ~]# curl http://127.0.0.1:8080
```

```sh
podman run -d -p 8080:80 rhel9-httpd
```

```sh
podman inspect docker.io/nginx | less
```

```sh
podman logs nginx
podman logs -f nginx
```

```sh
# podman ps -a
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS                     PORTS                 NAMES
e77c990d33db  docker.io/library/nginx:latest  nginx -g daemon o...  40 minutes ago  Exited (0) 13 seconds ago  0.0.0.0:8080->80/tcp  nginx
```

```sh
podman search rhel-httpd
```

```sh
podman run -d --name httpd docker.io/zubeirtech/rhel-httpd
```


```sh
[rhel@rhel app]$ cat index.html 
<H1>Super businessey? </H1>

$ podman run -d -v ./app:/usr/share/nginx/html:Z -p 8080:80 --name my-nginx2 docker.io/nginx
$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS         PORTS                 NAMES
6c058dc970a4  docker.io/library/nginx:latest  nginx -g daemon o...  41 seconds ago  Up 42 seconds  0.0.0.0:8080->80/tcp  my-nginx2

$ ls -lhdZ app
drwxr-xr-x. 2 rhel rhel system_u:object_r:container_file_t:s0:c383,c754 24 May 23 10:28 app # Notice that the context is set to container_file_t
```


### MariaDB

```sh
$ podman run -d --name mariadb -p 3306:3306 -v ./database/:/var/lib/mysql:Z -e MARIADB_ROOT_PASSWORD=redhat -e MARIADB_DATABASE=mydb docker.io/mariadb
```


```sh
ls database/
aria_log.00000001  ibtmp1                sys
aria_log_control   mariadb_upgrade_info  tc.log
ddl_recovery.log   multi-master.info     undo001
ib_buffer_pool     mydb                  undo002
ib_logfile0        mysql                 undo003
ibdata1            performance_schema
```

```sh
 mariadb -u root -p -h 127.0.0.1
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 12.2.2-MariaDB-ubu2404 mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```


```sh
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.001 sec)
```



### Podman build image

Containerfile
```
FROM docker.io/redhat/ubi10
RUN dnf -y install httpd && dnf clean all

EXPOSE 80
ENTRYPOINT [ "/usr/sbin/httpd","-DFOREGROUND" ]
```


```sh
$ podman build -t myapp .
```

```sh
podman images
REPOSITORY                 TAG         IMAGE ID      CREATED         SIZE
localhost/myapp            latest      0316b8da7516  53 seconds ago  249 MB
docker.io/library/mariadb  latest      f5d33f5426f2  2 days ago      341 MB
docker.io/redhat/ubi10     latest      5c4da5b52d0c  3 weeks ago     220 MB
```

```sh
# podman run -d --rm --name myhttp -p 80:80 myapp
2203cab524db3f2983d144284ed6d66d9226f24cae33a0bddc178fe6a846d041
```
need to run as root user if you want to use port 80 (low port number)


```sh
podman logs myhttp
```

```sh
curl http://127.0.0.1
```



### Clumsy-bird

```sh
# git clone https://github.com/ellisonleao/clumsy-bird
```




```sh
FROM docker.io/redhat/ubi10
RUN dnf -y install httpd && dnf clean all

COPY clumsy-bird /var/www/html

EXPOSE 80
ENTRYPOINT [ "/usr/sbin/httpd","-DFOREGROUND" ]
```

```sh
# podman build -t myapp .
```


```sh
# podman run -d --rm --name myhttp -p 80:80 myapp
025de498e27a09d6e5f450375167de8066d1b819c932c19be4208a54cd4c7c07
```


### Podman exec - to enter within a container with podman


```sh
[root@rhel myapp]# podman exec -it myhttp /bin/bash
[root@025de498e27a /]# 
```


## Buildah


```sh
$ buildah from registry.access.redhat.com/ubi9/ubi
```


```sh
buildah run ubi-working-container -- dnf -y install httpd
```


```sh
buildah run ubi-working-container systemctl enable httpd
```

```sh
buildah config --port 80 --cmd "/usr/sbin/init" ubi-working-container
```

```sh
buildah copy ubi-working-container index1.html /var/www/html/index.html
```


```sh
$ buildah commit ubi-working-container el-httpd
```


```sh
$ podman run -d -p 80:80 el-httpd
```


## Buildah with ubi-minimal

```sh
buildah from registry.access.redhat.com/ubi9/ubi-minimal
```


```sh
buildah run ubi-minimal-working-container -- microdnf -y install httpd
```


```sh
buildah copy ubi-minimal-working-container index.html /var/www/html/index.html
```

```sh
buildah config --port 80 --cmd "/usr/sbin/init" ubi-minimal-working-container 
```

```sh
buildah commit ubi-minimal-working-container el-httpd-min
```

```sh
podman images
```

```sh
podman run -d -p 80:80 el-httpd-min
```


## Using ss netstat from the host

```sh
$ podman inspect mariadb --format '{{.state.Pid}}'
```
This will give you the PID of the container

```sh
$ nsenter -n -t $PID ss -tulnp
```
This will use nsenter to use ss from the local host and send to $PID number the command ss -tulnp



