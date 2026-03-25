
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


### How to view kube config using kubeadm

```sh
kubeadm config view 
```


Example:

```sh
kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/degutos/.bluemix/plugins/container-service/clusters/cluster-fra-test-delme-full-cfgm29vf0a9mevqne4ag/ca-aaa00-cluster-fra-test-delme-full.pem
    server: https://c110.eu-de.containers.cloud.ibm.com:31610
  name: cluster-fra-test-delme-full/cfgm29vf0a9mevqne4ag
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:50192
  name: docker-desktop
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:49662
  name: kind-girus
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:57420
  name: kind-kind
- cluster:
    certificate-authority: /Users/degutos/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Sat, 10 Feb 2024 14:35:50 GMT
        provider: minikube.sigs.k8s.io
        version: v1.29.0
      name: cluster_info
    server: https://127.0.0.1:49524
  name: minikube
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:26443
  name: orbstack
- cluster:
    certificate-authority: /Users/degutos/.bluemix/plugins/container-service/clusters/test-cluster-delme-fra-cfg16bcf0b5r1jatqen0/ca-aaa00-test-cluster-delme-fra.pem
    server: https://c109.eu-de.containers.cloud.ibm.com:31564
  name: test-cluster-delme-fra/cfg16bcf0b5r1jatqen0
contexts:
- context:
    cluster: cluster-fra-test-delme-full/cfgm29vf0a9mevqne4ag
    namespace: default
    user: andre.gonzaga1@ibm.com/a97f936a990faca0fa2cc5edf374f192/iam.cloud.ibm.com-identity
  name: cluster-fra-test-delme-full/cfgm29vf0a9mevqne4ag
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
- context:
    cluster: kind-girus
    user: kind-girus
  name: kind-girus
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
- context:
    cluster: minikube
    user: minikube
  name: minikube
- context:
    cluster: orbstack
    user: orbstack
  name: orbstack
- context:
    cluster: test-cluster-delme-fra/cfg16bcf0b5r1jatqen0
    namespace: default
    user: andre.gonzaga1@ibm.com/a97f936a990faca0fa2cc5edf374f192/iam.cloud.ibm.com-identity
  name: test-cluster-delme-fra/cfg16bcf0b5r1jatqen0
current-context: orbstack
kind: Config
preferences: {}
users:
- name: andre.gonzaga1@ibm.com/a97f936a990faca0fa2cc5edf374f192/iam.cloud.ibm.com-identity
  user:
    auth-provider:
      config:
        client-id: kube
        client-secret: kube
        id-token: eyJraWQiOiIyMDIzMDExMTA4MjkiLCJhbGciOiJSUzI1NiJ9.eyJpYW1faWQiOiJJQk1pZC01MEhDMlQ4MFVNIiwiaXNzIjoiaHR0cHM6Ly9pYW0uY2xvdWQuaWJtLmNvbS9pZGVudGl0eSIsInN1YiI6IkFuZHJlLkdvbnphZ2ExQGlibS5jb20iLCJhdWQiOiJrdWJlIiwiZ2l2ZW5fbmFtZSI6IkFORFJFIEFVR1VTVE8iLCJmYW1pbHlfbmFtZSI6IkdPTlpBR0EiLCJuYW1lIjoiQU5EUkUgQVVHVVNUTyBHT05aQUdBIiwiZW1haWwiOiJBbmRyZS5Hb256YWdhMUBpYm0uY29tIiwiZXhwIjoxNjc1ODg4NzczLCJzY29wZSI6ImlibSBvcGVuaWQgY29udGFpbmVycy1rdWJlcm5ldGVzIiwiaWF0IjoxNjc1ODg1MjI1LCJhdXRobiI6eyJzdWIiOiJBbmRyZS5Hb256YWdhMUBpYm0uY29tIiwiaWFtX2lkIjoiSUJNaWQtNTBIQzJUODBVTSIsIm5hbWUiOiJBTkRSRSBBVUdVU1RPIEdPTlpBR0EiLCJnaXZlbl9uYW1lIjoiQU5EUkUgQVVHVVNUTyIsImZhbWlseV9uYW1lIjoiR09OWkFHQSIsImVtYWlsIjoiQW5kcmUuR29uemFnYTFAaWJtLmNvbSJ9LCJzdWJfYTk3ZjkzNmE5OTBmYWNhMGZhMmNjNWVkZjM3NGYxOTIiOiJhbmRyZS5nb256YWdhMUBpYm0uY29tIiwiaWFtX2lkX2E5N2Y5MzZhOTkwZmFjYTBmYTJjYzVlZGYzNzRmMTkyIjoiSUJNaWQtNTBIQzJUODBVTSIsInJlYWxtZWRfc3ViX2E5N2Y5MzZhOTkwZmFjYTBmYTJjYzVlZGYzNzRmMTkyIjoiSUJNaWQtYW5kcmUuZ29uemFnYTFAaWJtLmNvbSIsImdyb3Vwc19hOTdmOTM2YTk5MGZhY2EwZmEyY2M1ZWRmMzc0ZjE5MiI6W119.B-ETxh4-xh26uAu5penGTLOOepbLsdnYfmi382NOHv3fMqRsrsg-mTPr8VRwvA4O2O1376yWfptzn9l49GBPsdJoVbjUuboVE7c0FdlnZoV8Dio_08I7wXWIvQ_YZbSFC6vwKpwDNGz9fPN-zNGPrkpF9HPIImh89T14AHAuNNI5ahiIxLDB8KDDcXBv7Uy53ZrACYtHe8SRQK9ku2XfGFcKaqFOcVObzHbKktzIJ6mw69qkYMbn5SO6nOOnQ1L5SClutyouYjg3X7trD1hePFHrWKuLNmDdOQp9LRPZ7BuGea0xJB9E9lgWIcgZ7qpQFn1wUn1ad07aZXVqXScX5g
        idp-issuer-url: https://iam.cloud.ibm.com/identity
        refresh-token: eyJhbGciOiJydCJ9.eyJpYW1faWQiOiJJQk1pZC01MEhDMlQ4MFVNIiwiYWNjb3VudF9pZCI6ImE5N2Y5MzZhOTkwZmFjYTBmYTJjYzVlZGYzNzRmMTkyIn0.f8qKLhsAXtIMaWAM_2_qvchZJ18OPgR2yXyMWADPo-NCr0XvaGxrihimL7KZSfun8UKd07oQGqsAfo45vK3ZY2HFjEFrOHfRLhE-GqCXJYci3dCUsmc1n1Yr0VOxiuZ3GmiorR-etlFwTwxEbZSVMJjkIfeEbbG0ERtK1tPAFs796XN2oCspBhyZYiY-vkeZ_IzhfZi1-zpI52-ql-tECEEgILGHjd8sVusPRBhbAA-bUrjaG6-FDVOsmFbfw9esnCCoFZin7UX6-_yc0x-Gw1IOJLNH5gOd8mNxupq-VsnqT4cBiKpxO-gN8ryvQgM7pbB0JLvClocgJI_fz4Y3opc57qQQL9Hec7nyRYzLlaVVHBJ9xgsNbeS1pWVrPeJ_NzmCGFt28lyKRBjoF67qg-nNkmdrVL1j5w4zeLoafAq59wVd4S5-zzbQpLvak6qCi2pvtpqU3y0LIRemWsRLa9v3_kP0El41LKglWTzIQOrW4U2yu28rB8zSWfWxwjA24QZ2SE7xSfldLD75cP5I2SbfEa1knA7oRszXSNnH_t0z1yeywLlHnUldT97A9PrTGEm6BxH-mP75oPdiaSecB7O9g--cmuHHVXW-EunGo0orWZg7pMt22BOk_mEgmp7Yv4gAFvXX4KuixLfzN8wKB4afH7K_Zea1hm8nJdQcFhAbPL62Fs05LThekJA-Jh4uJjNbKqXUYunrKPHm8QtoVrNdru4UPf7ceq4e0HTsEJm1GFVROtd0i3nS1lJ_Fu-nb0tQg0E_BQx-NxXCCc8zcV1hpZjgZVHcGBzqoiibmrjHu218MI2duy1Ep9hiPsZEouQIsQgqIGRYF7BF3PDGurpytwf4JzovE77b3_tQ2ygSsU5v2MF8IPQbN4esm9r_NnAQfD6YG8HcirUXT7b48rFq7MsbrzRx-srrkZlAHDDQj5HC2eT8Pk8XNY7uDaDWLjaaZC8a6BLLDY9iGNnjYjw2UChy_mXuvNTaJLwgScEXljEUo3-BKKEeLqMGl16hnmnghs0eaogUPEw7ASPO9SVbCq8Tm-IF5wECKMvpYtrPhTjSD1ojzUy5YuflyMLJnfuKqzEp1ndz_PGzPr8ALjZrdnFC1DOdlBGwoiEpviBTYKsbGhTyYQqgzuM39xRU4OX3HkeHHeuzeZ8wKFvlDBpT2UTMrjeqt_wFT3j5qwnG9v3TUVNlqU0YWN5I9Ye_FzreaXYYJnRM91hBEW6Cjcaoup42wMvxYDYUAKwzW0xE6dNi9FVam5HeWg9RsrHBr3YEV2MSG0yZiybfDQEpzkXCUMr99zWIK6vSQBOlJHvWANl2qHKgmKP6ipI1w77QrsLNy03cWnt6JF-tOjWnn6J2
      name: oidc
- name: docker-desktop
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: kind-girus
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: kind-kind
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: minikube
  user:
    client-certificate: /Users/degutos/.minikube/profiles/minikube/client.crt
    client-key: /Users/degutos/.minikube/profiles/minikube/client.key
- name: orbstack
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```



To display with the certificate data:

```sh
➜  ~ kubectl config view --flatten
```




### How to merge 02 kubeconfig into a single config


```sh
KUBECONFIG=~/.kube/config-file-1:$HOME/.kube/config-file-2 kubectl config view --flatten > ~/.kube/config-merged 
```


Make sure you have different cluster name, different user and different context in both files.
If you have same user, you can edit your config file before merging and change the user name 



## Debuging communication between kubectl and API server


```sh
kubectl get no -v9
I0324 20:58:13.891638   51260 loader.go:402] Config loaded from file:  /Users/degutos/.kube/config
I0324 20:58:13.892141   51260 envvar.go:172] "Feature gate default state" feature="ClientsPreferCBOR" enabled=false
I0324 20:58:13.892155   51260 envvar.go:172] "Feature gate default state" feature="InformerResourceVersion" enabled=false
I0324 20:58:13.892158   51260 envvar.go:172] "Feature gate default state" feature="WatchListClient" enabled=false
I0324 20:58:13.892161   51260 envvar.go:172] "Feature gate default state" feature="ClientsAllowCBOR" enabled=false
I0324 20:58:13.894656   51260 helper.go:113] "Request Body" body=""
I0324 20:58:13.894850   51260 round_trippers.go:473] curl -v -XGET  -H "Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json" -H "User-Agent: kubectl/v1.32.1 (darwin/arm64) kubernetes/e9c9be4" 'https://127.0.0.1:26443/api/v1/nodes?limit=500'
I0324 20:58:13.895303   51260 round_trippers.go:517] HTTP Trace: Dial to tcp:127.0.0.1:26443 succeed
I0324 20:58:13.920761   51260 round_trippers.go:560] GET https://127.0.0.1:26443/api/v1/nodes?limit=500 200 OK in 25 milliseconds
I0324 20:58:13.920782   51260 round_trippers.go:577] HTTP Statistics: DNSLookup 0 ms Dial 0 ms TLSHandshake 6 ms ServerProcessing 19 ms Duration 25 ms
I0324 20:58:13.920786   51260 round_trippers.go:584] Response Headers:
I0324 20:58:13.920790   51260 round_trippers.go:587]     Audit-Id: bab1f395-7d2e-4b8a-8c29-8370a8172c6d
I0324 20:58:13.920793   51260 round_trippers.go:587]     Cache-Control: no-cache, private
I0324 20:58:13.920795   51260 round_trippers.go:587]     Content-Type: application/json
I0324 20:58:13.920797   51260 round_trippers.go:587]     X-Kubernetes-Pf-Flowschema-Uid: 24b5f764-0105-43c8-b411-02c2ae799b8c
I0324 20:58:13.920799   51260 round_trippers.go:587]     X-Kubernetes-Pf-Prioritylevel-Uid: 22750bef-cd53-4c91-a819-1cfe74467311
I0324 20:58:13.920800   51260 round_trippers.go:587]     Date: Tue, 24 Mar 2026 20:58:13 GMT
I0324 20:58:13.920868   51260 helper.go:113] "Response Body" body=<
	{"kind":"Table","apiVersion":"meta.k8s.io/v1","metadata":{"resourceVersion":"399737"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. Is required when creating resources, although some resources may allow a client to request the generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/names#names","priority":0},{"name":"Status","type":"string","format":"","description":"The status of the node","priority":0},{"name":"Roles","type":"string","format":"","description":"The roles of the node","priority":0},{"name":"Age","type":"string","format":"","description":"CreationTimestamp is a timestamp representing the server time when this object was created. It is not guaranteed to be set in happens-before order across separate operations. Clients may not set this value. It is represented in RFC3339 form and is in UTC.\n\nPopulated by the system. Read-only. Null for lists. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata","priority":0},{"name":"Version","type":"string","format":"","description":"Kubelet Version reported by the node.","priority":0},{"name":"Internal-IP","type":"string","format":"","description":"List of addresses reachable to the node. Queried from cloud provider, if available. More info: https://kubernetes.io/docs/reference/node/node-status/#addresses Note: This field is declared as mergeable, but the merge key is not sufficiently unique, which can cause data corruption when it is merged. Callers should instead use a full-replacement patch. See https://pr.k8s.io/79391 for an example. Consumers should assume that addresses can change during the lifetime of a Node. However, there are some exceptions where this may not be possible, such as Pods that inherit a Node's address in its own status or consumers of the downward API (status.hostIP).","priority":1},{"name":"External-IP","type":"string","format":"","description":"List of addresses reachable to the node. Queried from cloud provider, if available. More info: https://kubernetes.io/docs/reference/node/node-status/#addresses Note: This field is declared as mergeable, but the merge key is not sufficiently unique, which can cause data corruption when it is merged. Callers should instead use a full-replacement patch. See https://pr.k8s.io/79391 for an example. Consumers should assume that addresses can change during the lifetime of a Node. However, there are some exceptions where this may not be possible, such as Pods that inherit a Node's address in its own status or consumers of the downward API (status.hostIP).","priority":1},{"name":"OS-Image","type":"string","format":"","description":"OS Image reported by the node from /etc/os-release (e.g. Debian GNU/Linux 7 (wheezy)).","priority":1},{"name":"Kernel-Version","type":"string","format":"","description":"Kernel Version reported by the node from 'uname -r' (e.g. 3.16.0-0.bpo.4-amd64).","priority":1},{"name":"Container-Runtime","type":"string","format":"","description":"ContainerRuntime Version reported by the node through runtime remote API (e.g. containerd://1.4.2).","priority":1}],"rows":[{"cells":["orbstack","Ready","control-plane,master","24d","v1.33.5+orb1","192.168.139.2","\u003cnone\u003e","OrbStack","6.17.8-orbstack-00308-g8f9c941121b1","docker://28.5.2"],"object":{"kind":"PartialObjectMetadata","apiVersion":"meta.k8s.io/v1","metadata":{"name":"orbstack","uid":"b5fd7360-41a6-444f-89f6-8068c1c7fbed","resourceVersion":"399714","creationTimestamp":"2026-02-28T11:01:34Z","labels":{"beta.kubernetes.io/arch":"arm64","beta.kubernetes.io/instance-type":"k3s","beta.kubernetes.io/os":"linux","kubernetes.io/arch":"arm64","kubernetes.io/hostname":"orbstack","kubernetes.io/os":"linux","node-role.kubernetes.io/control-plane":"true","node-role.kubernetes.io/master":"true","node.kubernetes.io/instance-type":"k3s"},"annotations":{"alpha.kubernetes.io/provided-node-ip":"192.168.139.2,fd07:b51a:cc66::2","flannel.alpha.coreos.com/backend-data":"null","flannel.alpha.coreos.com/backend-type":"host-gw","flannel.alpha.coreos.com/backend-v6-data":"null","flannel.alpha.coreos.com/kube-subnet-manager":"true","flannel.alpha.coreos.com/public-ip":"192.168.139.2","flannel.alpha.coreos.com/public-ipv6":"fd07:b51a:cc66::2","k3s.io/hostname":"orbstack","k3s.io/internal-ip":"192.168.139.2,fd07:b51a:cc66::2","k3s.io/node-args":"[\"server\",\"--apiVersion\",\"kubelet.config.k8s.io/v1beta1\",\"--kind\",\"KubeletConfiguration\",\"--disable\",\"metrics-server,traefik,coredns\",\"--https-listen-port\",\"26443\",\"--lb-server-port\",\"26444\",\"--docker\",\"--protect-kernel-defaults\",\"--flannel-backend\",\"host-gw\",\"--disable-network-policy\",\"--cluster-cidr\",\"192.168.194.0/25,fd07:b51a:cc66:a::/72\",\"--service-cidr\",\"192.168.194.128/25,fd07:b51a:cc66:a:8000::/112\",\"--kube-controller-manager-arg\",\"node-cidr-mask-size-ipv4=25\",\"--kube-controller-manager-arg\",\"node-cidr-mask-size-ipv6=72\",\"--tls-san\",\"k8s.orb.local\",\"--tls-san\",\"docker.orb.local\",\"--write-kubeconfig\",\"/run/kubeconfig.yml\",\"--kubelet-arg\",\"--allowed-unsafe-sysctls\",\"net.*\",\"--kubelet-arg\",\"--config\",\"/etc/kubelet.conf\"]","k3s.io/node-config-hash":"QM7CWH5JDJDY2ONSQKPC2C65L3ZDEWAKWSADSH7ZH32MIPC5WICQ====","k3s.io/node-env":"{}","node.alpha.kubernetes.io/ttl":"0","volumes.kubernetes.io/controller-managed-attach-detach":"true"},"finalizers":["wrangler.cattle.io/node"],"managedFields":[{"manager":"k3s-supervisor@orbstack","operation":"Update","apiVersion":"v1","time":"2026-02-28T11:01:35Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:finalizers":{".":{},"v:\"wrangler.cattle.io/node\"":{}},"f:labels":{"f:node-role.kubernetes.io/control-plane":{},"f:node-role.kubernetes.io/master":{}}}}},{"manager":"k3s","operation":"Update","apiVersion":"v1","time":"2026-02-28T11:01:40Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:alpha.kubernetes.io/provided-node-ip":{},"f:k3s.io/hostname":{},"f:k3s.io/internal-ip":{},"f:k3s.io/node-args":{},"f:k3s.io/node-config-hash":{},"f:k3s.io/node-env":{},"f:node.alpha.kubernetes.io/ttl":{},"f:volumes.kubernetes.io/controller-managed-attach-detach":{}},"f:labels":{".":{},"f:beta.kubernetes.io/arch":{},"f:beta.kubernetes.io/instance-type":{},"f:beta.kubernetes.io/os":{},"f:kubernetes.io/arch":{},"f:kubernetes.io/hostname":{},"f:kubernetes.io/os":{},"f:node.kubernetes.io/instance-type":{}}},"f:spec":{"f:podCIDR":{},"f:podCIDRs":{".":{},"v:\"192.168.194.0/25\"":{},"v:\"fd07:b51a:cc66:a::/72\"":{}},"f:providerID":{}}}},{"manager":"k3s","operation":"Update","apiVersion":"v1","time":"2026-03-24T20:56:52Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{"f:flannel.alpha.coreos.com/backend-data":{},"f:flannel.alpha.coreos.com/backend-type":{},"f:flannel.alpha.coreos.com/backend-v6-data":{},"f:flannel.alpha.coreos.com/kube-subnet-manager":{},"f:flannel.alpha.coreos.com/public-ip":{},"f:flannel.alpha.coreos.com/public-ipv6":{}}},"f:status":{"f:allocatable":{"f:ephemeral-storage":{}},"f:capacity":{"f:ephemeral-storage":{}},"f:conditions":{"k:{\"type\":\"DiskPressure\"}":{"f:lastHeartbeatTime":{}},"k:{\"type\":\"MemoryPressure\"}":{"f:lastHeartbeatTime":{}},"k:{\"type\":\"PIDPressure\"}":{"f:lastHeartbeatTime":{}},"k:{\"type\":\"Ready\"}":{"f:lastHeartbeatTime":{},"f:message":{},"f:reason":{},"f:status":{}}},"f:images":{},"f:nodeInfo":{"f:bootID":{}}}},"subresource":"status"}]}}}]}
 >
NAME       STATUS   ROLES                  AGE   VERSION
orbstack   Ready    control-plane,master   24d   v1.33.5+orb1
```



> [!info]
> We can use a JSON formatter to display the above json content


We can try to curl and reach out the api

```sh
curl https://127.0.0.1:26443/api/v1/nodes?limit=500 -k -I
HTTP/2 401 
audit-id: 4d088f31-96d8-4283-bf53-b5bfe124df7c
cache-control: no-cache, private
content-type: application/json
content-length: 157
date: Tue, 24 Mar 2026 21:18:06 GMT
```


