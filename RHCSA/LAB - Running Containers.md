#podman #container 


### Install container tools

```sh
$ yum module install container-tools
```


### Running a basic container

```sh
$ podman login registry.lab.example.com
$ podman pull registry.lab.example.com/rhel8/httpd-24:latest
$ podman images
# Run a basic container from image, connect to a terminal, assign a name, interactive bash.
$ podman run --name myweb -it registry.alb.example.com/rhel8/httpd-24 /bin/bash
```

In the container we may try:

```sh
$ ps aux
$ id
$ exit
```

Run httpd -v command in a container using rhel8/httpd-24 deleting container

```sh
$ podman run --rm registry.lab.example.com/rhel8/httpd-24 httpd -v 
```


### Set a directory in the container for web server

```sh
$ mkdir /srv/web
$ chown -r containers: /srv/web
```


### Create a detached container

Create detached container name web port 8080 in container and 8888 on the host, mount /srv/web on /var/www in the container declare variable HTTPD_MPM=EVENT

```sh
$ podman login registry.lab.example.com
$ podman pull registry.lab.example.com/rhel8/httpd-24:latest
$ podman images
$ podman run --name web -p 8888:8080 -v /srv/web:/var/www:Z -E HTTPD_MPM=EVENT registry.lab.example.com/rhel8/httpd-24:1-105
$ podman ps
$ curl http://localhost:8888
```


### Set up a container to start up during boot

Configuring Systemd to start container web automatically during boot (as a user)

```sh
$ mkdir -p ~/.config/systemd/user
$ cd ~/.config/systemd/user
$ podman generate systemd --name web --files --new
$ podman stop web
$ podman rm web
$ systemctl --user daemon-reload
$ systemctl --user enable --now container-web.service
$ curl httpd//locahost:8888 
# or we can use this option
$ podman ps
$ loginctl enable-linger
$sudo systemctl reboot
```



### Finding and managing container images

Podman uses registries.com to get information it can use

```sh
$ cat /etc/containers/registries.conf
# $HOME/.config/containers/registries.conf # will override system/etc/ config
$ podman info # display configuration information
```



### Finding container registry

```sh
$ podman search registry.redhat.io/rhel8
$ podman search --no-trunc registry.access.rehat.com/rhel8 # to see more details
```


### Inspecting container images

```sh
$ skopeo inspect docker://registry.redhat.io/rhel8/python-36
# we can also inspect local image with $ podman inspect ....
```


#### List local images stored:

```sh
$ podman images
$ podman inspect registry.redhat.io/rhel8/python-36:latest # this will bring more information than skopeo inspect
```


#### Removing local images

```sh
$ podman rmi registry.redhat.io/rhel8/python-36:latest
```



# LAB - Finding and managing container images


### Display the container registry configuration

```sh
$ cat /etc/contianers/registries.conf
$ cat /home/student/.config/containers/registries.conf
```



#### Search registry for images

```sh
$ podman search registry.lab.example.com/ubi # will search for images that starts with ubi
$ podman search registry.lab.example.com/ # to search all images
```


#### Login into registry

```sh
$ podman login registry.lab.example.com
```



#### Inspect to see images details 

```sh
$ skopeo inspect docker://registry.lab.example.com/rhel8/httpd-24
```


#### Pull an image

```sh
$ podman pull registry.lab.example.com/rhel8/httpd-24
$ podman images
```


#### Inspect image locally

```sh
$ podman inspect registry.lab.example.com/rhel8/httpd-24
```


#### Remove an image

```sh
$ podman rmi registry.lab.example.com/rhel8/httpd-24
$ podman images
```


### Performing advanced container management


```sh
$ podman run -d -p 8000:8000 registry.redhat.io.rhel8/httpd-24
$ podman port -a # lisjt all port mapping in use

# must make sure firewall allows clients to connect
$ firewall-cmd --add-port=8000/tcp
# rootless container cannot open port below 1024, so 80:8080 won't work
# root can run taht, 8080:80 will work as normal user
```


#### Environment variables

```sh
$ podman inspect registry.lab.example.com/rhel8/mariadb-103:1-102
# usage: $ podman run -d -E MYSQL_USER=user -E MYSQL.PASSSWROD=password \
# -E MYSQL_DATABASE=DB -p 3306:3306 rhel8/mariadb-103

$ podman run -d --name container -E MYSQL_USER=user -E MYSQL_PASSWORD=pass \
-E MYSQL_DATABASE-=db -E MYSQL_ROOT_PASSWORD=pass -p 3306:3306 registry.lab.example.com/rhel8/mariadb-103:1-102

$ podman ps
$ podman ps -a 
$ podman stop my-thtpd-container
$ podman rm my-database
$ podman restart my-tthpd-container
$ podman kill my-httpd-container
$ podman kill -s SIGKILL my-httpd-container
```




