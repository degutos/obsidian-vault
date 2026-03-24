
## Kube Architecture  


- Controle Plane
	- API Server
	- ETCD
	- Scheduler
	- Controller Manager


- Data Plane
	- Node
	- Kubelet
	- Kube-proxy



## Kind install


```sh
$ brew instlal kind
```

```sh
➜  ~ kind version
kind v0.31.0 go1.25.5 darwin/arm64
```

#### Kind cluster configuration


```sh
➜  ~ kind get clusters
girus
```


Lets create another cluster with:
- control-plane
- worker
- worker

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: "0.0.0.0"
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    listenAddress: "0.0.0.0"
    protocol: TCP
  - containerPort: 8080
    hostPort: 8080
    listenAddress: "0.0.0.0"
    protocol: TCP
- role: worker
- role: worker
```


## Creating new cluster

```sh
➜  cluster-kind kind create cluster --config config.yaml --name comunidade-devops --kubeconfig config 
Creating cluster "comunidade-devops" ...
 ✓ Ensuring node image (kindest/node:v1.35.0) 🖼 
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-comunidade-devops"
You can now use your cluster with:

kubectl cluster-info --context kind-comunidade-devops --kubeconfig config

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
➜  cluster-kind 
```


```sh
➜  ls
config      config.yaml
```


#### Listing pods

```sh
docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS         PORTS                                                                                         NAMES
e29cbc4665f7   kindest/node:v1.35.0         "/usr/local/bin/entr…"   3 minutes ago   Up 3 minutes                                                                                                 comunidade-devops-worker
933dc3abdedd   kindest/node:v1.35.0         "/usr/local/bin/entr…"   3 minutes ago   Up 3 minutes   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:8080->8080/tcp, 127.0.0.1:64889->6443/tcp   comunidade-devops-control-plane
69cbbe56fbb6   kindest/node:v1.35.0         "/usr/local/bin/entr…"   3 minutes ago   Up 3 minutes                                                                                                 comunidade-devops-worker2
09f719615336   97e04611ad43                 "/coredns -conf /etc…"   20 hours ago    Up 20 hours                                                                                                  k8s_coredns_coredns-6cc96b5c97-n2mj5_kube-system_8940f3b1-5bfe-4db9-829a-8cd367b8c7f8_2
7913c2c06264   b41714cf6249                 "local-path-provisio…"   20 hours ago    Up 20 hours                                                                                                  k8s_local-path-provisioner_local-path-provisioner-774c6665dc-bxcds_kube-system_a657ff29-1006-41c4-90c0-6cb327b6e734_2
47f2bc66283d   rancher/mirrored-pause:3.6   "/pause"                 20 hours ago    Up 20 hours                                                                                                  k8s_POD_coredns-6cc96b5c97-n2mj5_kube-system_8940f3b1-5bfe-4db9-829a-8cd367b8c7f8_2
79d8b7797c60   rancher/mirrored-pause:3.6   "/pause"                 20 hours ago    Up 20 hours                                                                                                  k8s_POD_local-path-provisioner-774c6665dc-bxcds_kube-system_a657ff29-1006-41c4-90c0-6cb327b6e734_2
73134fcc63a6   kindest/node:v1.32.0         "/usr/local/bin/entr…"   11 days ago     Up 20 hours    127.0.0.1:49662->6443/tcp                                                                     girus-control-plane
```



Notice that the container comunidade-devops-control-plane is running on:

```
127.0.0.1:64889->6443/tcp
```



### Checking all pods running within the new cluster

```sh
kubectl get pods -A --kubeconfig config
NAMESPACE            NAME                                                      READY   STATUS    RESTARTS   AGE
kube-system          coredns-7d764666f9-vgj4j                                  1/1     Running   0          9m19s
kube-system          coredns-7d764666f9-zcbsd                                  1/1     Running   0          9m19s
kube-system          etcd-comunidade-devops-control-plane                      1/1     Running   0          9m28s
kube-system          kindnet-c8lgc                                             1/1     Running   0          9m15s
kube-system          kindnet-s92vd                                             1/1     Running   0          9m19s
kube-system          kindnet-whgdd                                             1/1     Running   0          9m14s
kube-system          kube-apiserver-comunidade-devops-control-plane            1/1     Running   0          9m27s
kube-system          kube-controller-manager-comunidade-devops-control-plane   1/1     Running   0          9m27s
kube-system          kube-proxy-s4gw6                                          1/1     Running   0          9m15s
kube-system          kube-proxy-vjkx4                                          1/1     Running   0          9m14s
kube-system          kube-proxy-xvkcg                                          1/1     Running   0          9m19s
kube-system          kube-scheduler-comunidade-devops-control-plane            1/1     Running   0          9m27s
local-path-storage   local-path-provisioner-67b8995b4b-s7c7g                   1/1     Running   0          9m19s
```



## Installing kubernetes with kubeadm across 03 nodes - 01 master and 02 workers.


We are going to install 03 nodes running in cloud being 01 Master and 02 worker nodes.

We should follow the Kubernetes documentation in: https://kubernetes.io/docs/setup/production-environment/container-runtimes/


### Enabling IPv4 packet forwarding

We should enable packet forwarding across all 03 nodes on our environment:

```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```


Verify that `net.ipv4.ip_forward` is set to 1 with:

```bash
sysctl net.ipv4.ip_forward
```


### Installing containerd

Follow the documentation: https://github.com/containerd/containerd/blob/main/docs/getting-started.md


#### Install docker engine and containerd on Ubuntu


Remove any previously docker package installed:


```bash
 sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)
```


Install:

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```



Lets install only containerd.io

```bash
sudo apt install containerd.io
```


Check if containerd is running:

```sh
systemctl status containerd 
```


Lets check the containerd configuration 

```sh
cat /etc/containerd/config.toml
```

Lets create a default configuration for containerd

```sh
containerd config default > /etc/containerd/config.toml 
```

Lets restart containerd after creation of the new config.toml file:

```sh
systemctl restart containerd
```

Make sure the containerd is running

```sh
systemctl status containerd
```



## Installing kubeadm, kubelet and kubectl

Before we start make sure we have connection with port 6443, the Kubernetes components need to communicate with each other and required communication on port 6443. To check:

```bash
nc 127.0.0.1 6443 -zv -w 2
```


### Opening port with ufw

If we are not getting the port 6443 open and we are using ufw we should run:

```bash
sudo ufw allow 6443/tcp 
sudo ufw reload
```


if firewalld

```sh
sudo firewall-cmd --permanent --add-port=6443/tcp 
sudo firewall-cmd --reload
```


#### Installing 

```sh
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

```sh
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```


```shell
sudo systemctl enable --now kubelet
```



## Configuring Kubernetes cluster with kubeadm in the MASTER node


Let now set up only in the master node with kubeadm.
Lets consider our Master IP 172.31.108.209

```sh
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 06:d1:6a:0d:6c:a9 brd ff:ff:ff:ff:ff:ff
    inet 172.31.108.209/20 metric 100 brd 172.31.111.255 scope global dynamic ens5
```


Lets make sure that workers can ping the master

```sh
# ping -c3 172.31.108.209
PING 172.31.108.209 (172.31.108.209) 56(84) bytes of data.
64 bytes from 172.31.108.209: icmp_seq=1 ttl=64 time=0.798 ms
64 bytes from 172.31.108.209: icmp_seq=2 ttl=64 time=0.262 ms
64 bytes from 172.31.108.209: icmp_seq=3 ttl=64 time=0.251 ms

--- 172.31.108.209 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2051ms
rtt min/avg/max/mdev = 0.251/0.437/0.798/0.255 ms
```


Lets use the kubeadm help to understand all parameters 

```sh
# kubeadm init --help
```


### Installing cluster with kubeadm

Lets consider 172.31.108.209 as master IP and the pod-network-cidr 10.244.0.0/16 is the default for the flannel:


```sh
# kubeadm init --apiserver-advertise-address 172.31.108.209 --pod-network-cidr 10.244.0.0/16
```


You'll see this output at the end:

```sh
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.108.209:6443 --token iiu7bp.hay94ny2yhqlwpnz \
	--discovery-token-ca-cert-hash sha256:befcb2da442f8473c9331c12ce1dca4f8b6dbe8f3b8b7b19bb539a2d507aa676 
```



Lets configure our KUBECONFIG

```sh
# export KUBECONFIG=/etc/kubernetes/admin.conf
```


Now we can check kubectl command 

```sh
# kubectl get pods -A -o wide
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-7d764666f9-lcr8w         0/1     Pending   0          5m4s    <none>           <none>   <none>           <none>
kube-system   coredns-7d764666f9-pfgrb         0/1     Pending   0          5m4s    <none>           <none>   <none>           <none>
kube-system   etcd-master                      1/1     Running   0          5m10s   172.31.108.209   master   <none>           <none>
kube-system   kube-apiserver-master            1/1     Running   0          5m10s   172.31.108.209   master   <none>           <none>
kube-system   kube-controller-manager-master   1/1     Running   0          5m10s   172.31.108.209   master   <none>           <none>
kube-system   kube-proxy-n4kn9                 1/1     Running   0          5m5s    172.31.108.209   master   <none>           <none>
kube-system   kube-scheduler-master            1/1     Running   0          5m14s   172.31.108.209   master   <none>           <none>
```



Make sure we have cgroup2 enabled

```sh
# mount | grep cgroup
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime)
```


### Installing a network add-on

Doc: https://github.com/flannel-io/flannel?tab=readme-ov-file

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```


```
root@master:/etc/containerd# kubectl logs kube-flannel-ds-bfplp -n kube-flannel | tail -n3
Defaulted container "kube-flannel" out of: kube-flannel, install-cni-plugin (init), install-cni (init)
I0321 12:02:42.820849       1 main.go:244] Installing signal handlers
I0321 12:02:42.821251       1 main.go:523] Found network config - Backend type: vxlan
E0321 12:02:42.821348       1 main.go:278] Failed to check br_netfilter: stat /proc/sys/net/bridge/bridge-nf-call-iptables: no such file or directory
```


### Check if module br_netfilter is running 

```sh
root@master:/etc/containerd# lsmod | grep br_netfilter
root@master:/etc/containerd# 
```


```sh
root@master:/etc/containerd# modprobe br_netfilter
root@master:/etc/containerd# lsmod | grep br_netfilter
br_netfilter           32768  0
bridge                421888  1 br_netfilter
```

Now we see the flannel pod running:

```sh
# kubectl get pods -n kube-flannel
NAME                    READY   STATUS    RESTARTS       AGE
kube-flannel-ds-bfplp   1/1     Running   5 (118s ago)   3m44s
```


### Checking all pods running and cluster set up

```sh
# kubectl get pods -A
NAMESPACE      NAME                             READY   STATUS    RESTARTS        AGE
kube-flannel   kube-flannel-ds-bfplp            1/1     Running   5 (5m54s ago)   7m40s
kube-system    coredns-7d764666f9-lcr8w         1/1     Running   0               17m
kube-system    coredns-7d764666f9-pfgrb         1/1     Running   0               17m
kube-system    etcd-master                      1/1     Running   0               17m
kube-system    kube-apiserver-master            1/1     Running   0               17m
kube-system    kube-controller-manager-master   1/1     Running   0               17m
kube-system    kube-proxy-n4kn9                 1/1     Running   0               17m
kube-system    kube-scheduler-master            1/1     Running   0               17m
```



## joining workers to masters


Lets check the command in MASTER node to enable workers to join:

```sh
# kubeadm token create --print-join-command 
kubeadm join 172.31.108.209:6443 --token um75c2.msn86fdkd10cmrk0 --discovery-token-ca-cert-hash sha256:befcb2da442f8473c9331c12ce1dca4f8b6dbe8f3b8b7b19bb539a2d507aa676 
```

We should run this join command across all worker nodes

```sh
root@worker1:/etc/containerd# kubeadm join 172.31.108.209:6443 --token 7xakr7.jeu2s7h8t0ooazws --discovery-token-ca-cert-hash sha256:befcb2da442f8473c9331c12ce1dca4f8b6dbe8f3b8b7b19bb539a2d507aa676
[preflight] Running pre-flight checks
[preflight] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[preflight] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/instance-config.yaml"
[patches] Applied patch of type "application/strategic-merge-patch+json" to target "kubeletconfiguration"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 503.058652ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```


After adding each worker to the cluster we will see:

```sh
root@master:/etc/containerd# kubectl get no
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   21m   v1.35.3
worker1   Ready    <none>          92s   v1.35.3
worker2   Ready    <none>          23s   v1.35.3
```



## Kubectl

kubectl is a binary used to interact with kube cluster. It uses a config with your cluster configuration with user information, master api address, certificate to access the api.

We can install kubectl in different ways, like copying the binary with curl command, or install with apt or yum or dnf. 

In our master node we have the configuration file. 

```sh
cd /etc/kubernetes
cat admin.conf
```


This is the kubectl config file, the kubeconfig read this config file to authenticate to KUBE API.

We can copy this content to your localhost, where we are going to execute kubectl command.

Save this file in:

```sh
cd $HOME/.kube
vim config
# save your content copied from /etc/kubernetes/admin.conf and save it 
# in your localhost ~/.kube/config
```


Once you created your ~/.kube/config you should be able to run kubectl command

```sh
kubectl get pods -A
```



## Using kubeconfig temporarily 

```sh
export KUBECONFIG=$HOME/.kube/config.kubeadm
```

If we set KUBECONFIG variable we use that temp file while that variable is set.

Check all variables

```sh
env | grep KUBE
```

To unset variable

```sh
unset env
```


To use KUBECONFIG for a simple execution, use:

```sh
KUBECONFIG=$HOME/.kube/config.kubeadm kubectl get pods -A
```

Another way:

```sh
kubectl get pods -A --kubeconfig $HOME/.kube/config.kubeadm
```



### Creating alias for kubectl with k 


```sh
alias k=kubectl
```

To save this for always to use, lets config our profile:

```sh
vim ~/.bashrc
# OR
vim ~/.zshrc

alias k=kubectl
```


### To enable kube autocomplete

```sh
source <(kubectl completion zsh)
```

we can add this line to ~/.zshrc file













