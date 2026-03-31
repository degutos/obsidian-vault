
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


## Working with certificates

First thing lets cat the kube/config file and see what certificate-authority we have for the cluster we are accessing

```sh
$ cat .kube/config
...
  - cluster:
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJlRENDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUzTnpJeU56WTBPRGt3SGhjTk1qWXdNakk0TVRFd01USTVXaGNOTXpZd01qSTJNVEV3TVRJNQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUzTnpJeU56WTBPRGt3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFTdytaMU5NeEoxSlNCcHdZWE51R3JvNlpucE1HM29GRzJUazZtT2ZkVWgKY3QvQ1NIVUxvVkprQk1zN29TUVFCbmE2OUxjazFyeko2UEUvaTd4RE5zbFBvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVVBUNFBPYVUzOXo4bGYvdzUxdVoxClIvQ2hTNjh3Q2dZSUtvWkl6ajBFQXdJRFNRQXdSZ0loQUpVU0lHbHl1bi9BMS9Ja0NPam9xUjNzdHJFV1dVSXUKbi96RHhxWmxRSjQ3QWlFQWlTcDVqUkxJd3ljUHIxNDF0bUNNdGdKeEZzeFIyMFYxS0sxZ1JiVUNMaGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
      server: https://127.0.0.1:26443
    name: orbstack
contexts:
...
```


We can copy the certificate now and decode it to a ca.crt file

```sh
➜  /tmp echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJlRENDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUzTnpJeU56WTBPRGt3SGhjTk1qWXdNakk0TVRFd01USTVXaGNOTXp
Zd01qSTJNVEV3TVRJNQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUzTnpJeU56WTBPRGt3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFTdytaMU5NeEoxSlNCcHdZWE51R3JvNlpucE1HM29GRzJUazZtT2ZkVWgKY3QvQ1NIVUxvVkprQk1zN29TUVFCbmE2OUxjazFyeko2UEUvaTd4RE5zbFBvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVVBUNFBPYVUzOXo4bGYvdzUxdVoxClIvQ2hTNjh3Q2dZSUtvWkl6ajBFQXdJRFNRQXdSZ0loQUpVU0lHbHl1bi9BMS9Ja0NPam9xUjNzdHJFV1dVSXUKbi96RHhxWmxRSjQ3QWlFQWlTcDVqUkxJd3ljUHIxNDF0bUNNdGdKeEZzeFIyMFYxS0sxZ1JiVUNMaGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K" | base64 -d > ca.crt
```



```sh
➜  /tmp cat ca.crt 
-----BEGIN CERTIFICATE-----
MIIBeDCCAR2gAwIBAgIBADAKBggqhkjOPQQDAjAjMSEwHwYDVQQDDBhrM3Mtc2Vy
dmVyLWNhQDE3NzIyNzY0ODkwHhcNMjYwMjI4MTEwMTI5WhcNMzYwMjI2MTEwMTI5
WjAjMSEwHwYDVQQDDBhrM3Mtc2VydmVyLWNhQDE3NzIyNzY0ODkwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAASw+Z1NMxJ1JSBpwYXNuGro6ZnpMG3oFG2Tk6mOfdUh
ct/CSHULoVJkBMs7oSQQBna69Lck1rzJ6PE/i7xDNslPo0IwQDAOBgNVHQ8BAf8E
BAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUPT4POaU39z8lf/w51uZ1
R/ChS68wCgYIKoZIzj0EAwIDSQAwRgIhAJUSIGlyun/A1/IkCOjoqR3strEWWUIu
n/zDxqZlQJ47AiEAiSp5jRLIwycPr141tmCMtgJxFsxR20V1KK1gRbUCLhc=
-----END CERTIFICATE-----
```


Then lets select the .kube/config part for the user client certificate and and client key:

```sh
  - name: orbstack
    user:
      client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJrVENDQVRlZ0F3SUJBZ0lJSzAydWNlOC9UaVl3Q2dZSUtvWkl6ajBFQXdJd0l6RWhNQjhHQTFVRUF3d1kKYXpOekxXTnNhV1Z1ZEMxallVQXhOemN5TWpjMk5EZzVNQjRYRFRJMk1ESXlPREV4TURFeU9Wb1hEVEkzTURJeQpPREV4TURFeU9Wb3dNREVYTUJVR0ExVUVDaE1PYzNsemRHVnRPbTFoYzNSbGNuTXhGVEFUQmdOVkJBTVRESE41CmMzUmxiVHBoWkcxcGJqQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJNYm9ZZDdQOGMwV1V1ekkKbjk1N2p3M2Nzb1NyMFRINC9xM3JQbTluWHRIaFh0NjhrOXB2Vit1UUJPdEJ4K3VsYWE1VTRNYThKQXVNREJTVAp1Qm5PdUs2alNEQkdNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUtCZ2dyQmdFRkJRY0RBakFmCkJnTlZIU01FR0RBV2dCUzdjQTJmQno3Qnp0SHp2UXFPeGNqbmxKYng1akFLQmdncWhrak9QUVFEQWdOSUFEQkYKQWlCNGRISkxWck1iVDZvRHZKUFlPN0xOeStUa3RuUHQvTXc4RndLbXcxamx0d0loQUxRL1dIQ0x6L1N1aHVXZgpYNmFCWEVzV2NwT0JUbWlrSFQvR0NyMU9Ld2gxCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KLS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdFkyeHAKWlc1MExXTmhRREUzTnpJeU56WTBPRGt3SGhjTk1qWXdNakk0TVRFd01USTVXaGNOTXpZd01qSTJNVEV3TVRJNQpXakFqTVNFd0h3WURWUVFEREJock0zTXRZMnhwWlc1MExXTmhRREUzTnpJeU56WTBPRGt3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFSQUNkTmFGd3hVS1BLajVZQ0pkcXRPSmJ6OHFmcDlIaDQ5Q0h6cmJCYnkKeElyN0lPWWhhZFN6ZkhuM2xvZ1VUQklOV1BIRmJ3VTBMaVFEUXNPcm84TnJvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVXUzQU5ud2Mrd2M3Ujg3MEtqc1hJCjU1U1c4ZVl3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQU53Ulh4ZjZQbUI4cm85dTNRdHFPRE5FOUd1MUVGc2EKR0ZMalJZbDB0MUQ4QWlCbzVURy9uYXVXdit1VUVRMzcwQVF2Mnlqb0QyK2ludGsyZzI4VmY2bURQUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
      client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSU5xVXN3RVUyRVRINVI1TCt2M1VYR0psTG1QdHVCcXAxZ2g2b2Zsb0V3Uk1vQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFeHVoaDNzL3h6UlpTN01pZjNudVBEZHl5aEt2Uk1maityZXMrYjJkZTBlRmUzcnlUMm05WAo2NUFFNjBISDY2VnBybFRneHJ3a0M0d01GSk80R2M2NHJnPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
```


Lets do the same thing we did before, lets decode the certificate to base64 and redirect to client.crt 

```sh
➜  /tmp echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJrVENDQVRlZ0F3SUJBZ0lJSzAydWNlOC9UaVl3Q2dZSUtvWkl6ajBFQXdJd0l6RWhNQjhHQTFVRUF3d1kKYXpOekxXTnNhV1Z1ZEMxallVQXhOemN5TWpjMk5EZzVNQjRYRFRJMk1ESXlPREV4TUR
FeU9Wb1hEVEkzTURJeQpPREV4TURFeU9Wb3dNREVYTUJVR0ExVUVDaE1PYzNsemRHVnRPbTFoYzNSbGNuTXhGVEFUQmdOVkJBTVRESE41CmMzUmxiVHBoWkcxcGJqQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJNYm9ZZDdQOGMwV1V1ekkKbjk1N2p3M2Nzb1NyMFRINC9xM3JQbTluWHRIaFh0NjhrOXB2Vit1UUJPdEJ4K3VsYWE1VTRNYThKQXVNREJTVAp1Qm5PdUs2alNEQkdNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUtCZ2dyQmdFRkJRY0RBakFmCkJnTlZIU01FR0RBV2dCUzdjQTJmQno3Qnp0SHp2UXFPeGNqbmxKYng1akFLQmdncWhrak9QUVFEQWdOSUFEQkYKQWlCNGRISkxWck1iVDZvRHZKUFlPN0xOeStUa3RuUHQvTXc4RndLbXcxamx0d0loQUxRL1dIQ0x6L1N1aHVXZgpYNmFCWEVzV2NwT0JUbWlrSFQvR0NyMU9Ld2gxCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KLS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdFkyeHAKWlc1MExXTmhRREUzTnpJeU56WTBPRGt3SGhjTk1qWXdNakk0TVRFd01USTVXaGNOTXpZd01qSTJNVEV3TVRJNQpXakFqTVNFd0h3WURWUVFEREJock0zTXRZMnhwWlc1MExXTmhRREUzTnpJeU56WTBPRGt3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFSQUNkTmFGd3hVS1BLajVZQ0pkcXRPSmJ6OHFmcDlIaDQ5Q0h6cmJCYnkKeElyN0lPWWhhZFN6ZkhuM2xvZ1VUQklOV1BIRmJ3VTBMaVFEUXNPcm84TnJvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVXUzQU5ud2Mrd2M3Ujg3MEtqc1hJCjU1U1c4ZVl3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQU53Ulh4ZjZQbUI4cm85dTNRdHFPRE5FOUd1MUVGc2EKR0ZMalJZbDB0MUQ4QWlCbzVURy9uYXVXdit1VUVRMzcwQVF2Mnlqb0QyK2ludGsyZzI4VmY2bURQUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K" | base64 -d > client.crt

➜  /tmp cat client.crt 
-----BEGIN CERTIFICATE-----
MIIBkTCCATegAwIBAgIIK02uce8/TiYwCgYIKoZIzj0EAwIwIzEhMB8GA1UEAwwY
azNzLWNsaWVudC1jYUAxNzcyMjc2NDg5MB4XDTI2MDIyODExMDEyOVoXDTI3MDIy
ODExMDEyOVowMDEXMBUGA1UEChMOc3lzdGVtOm1hc3RlcnMxFTATBgNVBAMTDHN5
c3RlbTphZG1pbjBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABMboYd7P8c0WUuzI
n957jw3csoSr0TH4/q3rPm9nXtHhXt68k9pvV+uQBOtBx+ulaa5U4Ma8JAuMDBST
uBnOuK6jSDBGMA4GA1UdDwEB/wQEAwIFoDATBgNVHSUEDDAKBggrBgEFBQcDAjAf
BgNVHSMEGDAWgBS7cA2fBz7BztHzvQqOxcjnlJbx5jAKBggqhkjOPQQDAgNIADBF
AiB4dHJLVrMbT6oDvJPYO7LNy+TktnPt/Mw8FwKmw1jltwIhALQ/WHCLz/SuhuWf
X6aBXEsWcpOBTmikHT/GCr1OKwh1
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIBdzCCAR2gAwIBAgIBADAKBggqhkjOPQQDAjAjMSEwHwYDVQQDDBhrM3MtY2xp
ZW50LWNhQDE3NzIyNzY0ODkwHhcNMjYwMjI4MTEwMTI5WhcNMzYwMjI2MTEwMTI5
WjAjMSEwHwYDVQQDDBhrM3MtY2xpZW50LWNhQDE3NzIyNzY0ODkwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAARACdNaFwxUKPKj5YCJdqtOJbz8qfp9Hh49CHzrbBby
xIr7IOYhadSzfHn3logUTBINWPHFbwU0LiQDQsOro8Nro0IwQDAOBgNVHQ8BAf8E
BAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUu3ANnwc+wc7R870KjsXI
55SW8eYwCgYIKoZIzj0EAwIDSAAwRQIhANwRXxf6PmB8ro9u3QtqODNE9Gu1EFsa
GFLjRYl0t1D8AiBo5TG/nauWv+uUEQ370AQv2yjoD2+intk2g28Vf6mDPQ==
-----END CERTIFICATE-----
```

Now lets get the client key certificate and redirect to client.key

```sh
echo "LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSU5xVXN3RVUyRVRINVI1TCt2M1VYR0psTG1QdHVCcXAxZ2g2b2Zsb0V3Uk1vQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFeHVoaDNzL3h6UlpTN01pZjNudVBEZHl5aEt2Uk1maityZXMrYjJkZTBlRmUzcnlUMm05WAo2NUFFNjBISDY2VnBybFRneHJ3a0M0d01GSk80R2M2NHJnPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=" | base64 -d > client.key

```

```sh
➜  /tmp cat client.key
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEINqUswEU2ETH5R5L+v3UXGJlLmPtuBqp1gh6ofloEwRMoAoGCCqGSM49
AwEHoUQDQgAExuhh3s/xzRZS7Mif3nuPDdyyhKvRMfj+res+b2de0eFe3ryT2m9X
65AE60HH66VprlTgxrwkC4wMFJO4Gc64rg==
-----END EC PRIVATE KEY-----
```



Now we have created 03 files:

```sh
ls -lh ca.crt client.crt client.key
-rw-r--r--@ 1 degutos  wheel   570B Mar 27 08:44 ca.crt
-rw-r--r--@ 1 degutos  wheel   1.1K Mar 27 08:49 client.crt
-rw-r--r--@ 1 degutos  wheel   227B Mar 27 08:52 client.key
```


### Checking content of certificate with openssh

```sh
openssl x509 -in ca.crt 
-----BEGIN CERTIFICATE-----
MIIBeDCCAR2gAwIBAgIBADAKBggqhkjOPQQDAjAjMSEwHwYDVQQDDBhrM3Mtc2Vy
dmVyLWNhQDE3NzIyNzY0ODkwHhcNMjYwMjI4MTEwMTI5WhcNMzYwMjI2MTEwMTI5
WjAjMSEwHwYDVQQDDBhrM3Mtc2VydmVyLWNhQDE3NzIyNzY0ODkwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAASw+Z1NMxJ1JSBpwYXNuGro6ZnpMG3oFG2Tk6mOfdUh
ct/CSHULoVJkBMs7oSQQBna69Lck1rzJ6PE/i7xDNslPo0IwQDAOBgNVHQ8BAf8E
BAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUPT4POaU39z8lf/w51uZ1
R/ChS68wCgYIKoZIzj0EAwIDSQAwRgIhAJUSIGlyun/A1/IkCOjoqR3strEWWUIu
n/zDxqZlQJ47AiEAiSp5jRLIwycPr141tmCMtgJxFsxR20V1KK1gRbUCLhc=
-----END CERTIFICATE-----
```

or 

```sh
openssl x509 -in ca.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN=k3s-server-ca@1772276489
        Validity
            Not Before: Feb 28 11:01:29 2026 GMT
            Not After : Feb 26 11:01:29 2036 GMT
        Subject: CN=k3s-server-ca@1772276489
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:b0:f9:9d:4d:33:12:75:25:20:69:c1:85:cd:b8:
                    6a:e8:e9:99:e9:30:6d:e8:14:6d:93:93:a9:8e:7d:
                    d5:21:72:df:c2:48:75:0b:a1:52:64:04:cb:3b:a1:
                    24:10:06:76:ba:f4:b7:24:d6:bc:c9:e8:f1:3f:8b:
                    bc:43:36:c9:4f
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                3D:3E:0F:39:A5:37:F7:3F:25:7F:FC:39:D6:E6:75:47:F0:A1:4B:AF
    Signature Algorithm: ecdsa-with-SHA256
    Signature Value:
        30:46:02:21:00:95:12:20:69:72:ba:7f:c0:d7:f2:24:08:e8:
        e8:a9:1d:ec:b6:b1:16:59:42:2e:9f:fc:c3:c6:a6:65:40:9e:
        3b:02:21:00:89:2a:79:8d:12:c8:c3:27:0f:af:5e:35:b6:60:
        8c:b6:02:71:16:cc:51:db:45:75:28:ad:60:45:b5:02:2e:17
-----BEGIN CERTIFICATE-----
MIIBeDCCAR2gAwIBAgIBADAKBggqhkjOPQQDAjAjMSEwHwYDVQQDDBhrM3Mtc2Vy
dmVyLWNhQDE3NzIyNzY0ODkwHhcNMjYwMjI4MTEwMTI5WhcNMzYwMjI2MTEwMTI5
WjAjMSEwHwYDVQQDDBhrM3Mtc2VydmVyLWNhQDE3NzIyNzY0ODkwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAASw+Z1NMxJ1JSBpwYXNuGro6ZnpMG3oFG2Tk6mOfdUh
ct/CSHULoVJkBMs7oSQQBna69Lck1rzJ6PE/i7xDNslPo0IwQDAOBgNVHQ8BAf8E
BAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUPT4POaU39z8lf/w51uZ1
R/ChS68wCgYIKoZIzj0EAwIDSQAwRgIhAJUSIGlyun/A1/IkCOjoqR3strEWWUIu
n/zDxqZlQJ47AiEAiSp5jRLIwycPr141tmCMtgJxFsxR20V1KK1gRbUCLhc=
-----END CERTIFICATE-----
```

We can see all certificate details above.






## Using curl command 

We can use man curl and learn parameters for:

```
--cacert
--cert
--key
```


```sh
man curl
```


Lets now use the curl to get all information regarding to nodes from my API:


```sh
curl https://127.0.0.1:26443/api/v1/nodes\?limit\=500 --cacert ca.crt --cert client.crt --key client.key
{
  "kind": "NodeList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "465231"
  },
  "items": [
    {
      "metadata": {
        "name": "orbstack",
        "uid": "b5fd7360-41a6-444f-89f6-8068c1c7fbed",
        "resourceVersion": "465191",
        "creationTimestamp": "2026-02-28T11:01:34Z",
        "labels": {
          "beta.kubernetes.io/arch": "arm64",
          "beta.kubernetes.io/instance-type": "k3s",
          "beta.kubernetes.io/os": "linux",
          "kubernetes.io/arch": "arm64",
          "kubernetes.io/hostname": "orbstack",
          "kubernetes.io/os": "linux",
          "node-role.kubernetes.io/control-plane": "true",
          "node-role.kubernetes.io/master": "true",
          "node.kubernetes.io/instance-type": "k3s"
        },
        "annotations": {
          "alpha.kubernetes.io/provided-node-ip": "192.168.139.2,fd07:b51a:cc66::2",
          "flannel.alpha.coreos.com/backend-data": "null",
          "flannel.alpha.coreos.com/backend-type": "host-gw",
          "flannel.alpha.coreos.com/backend-v6-data": "null",
          "flannel.alpha.coreos.com/kube-subnet-manager": "true",
          "flannel.alpha.coreos.com/public-ip": "192.168.139.2",
          "flannel.alpha.coreos.com/public-ipv6": "fd07:b51a:cc66::2",
          "k3s.io/hostname": "orbstack",
          "k3s.io/internal-ip": "192.168.139.2,fd07:b51a:cc66::2",
          "k3s.io/node-args": "[\"server\",\"--apiVersion\",\"kubelet.config.k8s.io/v1beta1\",\"--kind\",\"KubeletConfiguration\",\"--disable\",\"metrics-server,traefik,coredns\",\"--https-listen-port\",\"26443\",\"--lb-server-port\",\"26444\",\"--docker\",\"--protect-kernel-defaults\",\"--flannel-backend\",\"host-gw\",\"--disable-network-policy\",\"--cluster-cidr\",\"192.168.194.0/25,fd07:b51a:cc66:a::/72\",\"--service-cidr\",\"192.168.194.128/25,fd07:b51a:cc66:a:8000::/112\",\"--kube-controller-manager-arg\",\"node-cidr-mask-size-ipv4=25\",\"--kube-controller-manager-arg\",\"node-cidr-mask-size-ipv6=72\",\"--tls-san\",\"k8s.orb.local\",\"--tls-san\",\"docker.orb.local\",\"--write-kubeconfig\",\"/run/kubeconfig.yml\",\"--kubelet-arg\",\"--allowed-unsafe-sysctls\",\"net.*\",\"--kubelet-arg\",\"--config\",\"/etc/kubelet.conf\"]",
          "k3s.io/node-config-hash": "QM7CWH5JDJDY2ONSQKPC2C65L3ZDEWAKWSADSH7ZH32MIPC5WICQ====",
          "k3s.io/node-env": "{}",
          "node.alpha.kubernetes.io/ttl": "0",
          "volumes.kubernetes.io/controller-managed-attach-detach": "true"
        },
        "finalizers": [
          "wrangler.cattle.io/node"
        ],
        "managedFields": [
          {
            "manager": "k3s-supervisor@orbstack",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2026-02-28T11:01:35Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:finalizers": {
                  ".": {},
                  "v:\"wrangler.cattle.io/node\"": {}
                },
                "f:labels": {
                  "f:node-role.kubernetes.io/control-plane": {},
                  "f:node-role.kubernetes.io/master": {}
                }
              }
            }
          },
          {
            "manager": "k3s",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2026-02-28T11:01:40Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:alpha.kubernetes.io/provided-node-ip": {},
                  "f:k3s.io/hostname": {},
                  "f:k3s.io/internal-ip": {},
                  "f:k3s.io/node-args": {},
                  "f:k3s.io/node-config-hash": {},
                  "f:k3s.io/node-env": {},
                  "f:node.alpha.kubernetes.io/ttl": {},
                  "f:volumes.kubernetes.io/controller-managed-attach-detach": {}
                },
                "f:labels": {
                  ".": {},
                  "f:beta.kubernetes.io/arch": {},
                  "f:beta.kubernetes.io/instance-type": {},
                  "f:beta.kubernetes.io/os": {},
                  "f:kubernetes.io/arch": {},
                  "f:kubernetes.io/hostname": {},
                  "f:kubernetes.io/os": {},
                  "f:node.kubernetes.io/instance-type": {}
                }
              },
              "f:spec": {
                "f:podCIDR": {},
                "f:podCIDRs": {
                  ".": {},
                  "v:\"192.168.194.0/25\"": {},
                  "v:\"fd07:b51a:cc66:a::/72\"": {}
                },
                "f:providerID": {}
              }
            }
          },
          {
            "manager": "k3s",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2026-03-27T09:04:38Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  "f:flannel.alpha.coreos.com/backend-data": {},
                  "f:flannel.alpha.coreos.com/backend-type": {},
                  "f:flannel.alpha.coreos.com/backend-v6-data": {},
                  "f:flannel.alpha.coreos.com/kube-subnet-manager": {},
                  "f:flannel.alpha.coreos.com/public-ip": {},
                  "f:flannel.alpha.coreos.com/public-ipv6": {}
                }
              },
              "f:status": {
                "f:allocatable": {
                  "f:ephemeral-storage": {}
                },
                "f:capacity": {
                  "f:ephemeral-storage": {}
                },
                "f:conditions": {
                  "k:{\"type\":\"DiskPressure\"}": {
                    "f:lastHeartbeatTime": {}
                  },
                  "k:{\"type\":\"MemoryPressure\"}": {
                    "f:lastHeartbeatTime": {}
                  },
                  "k:{\"type\":\"PIDPressure\"}": {
                    "f:lastHeartbeatTime": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    "f:lastHeartbeatTime": {},
                    "f:message": {},
                    "f:reason": {},
                    "f:status": {}
                  }
                },
                "f:images": {},
                "f:nodeInfo": {
                  "f:bootID": {}
                }
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "podCIDR": "192.168.194.0/25",
        "podCIDRs": [
          "192.168.194.0/25",
          "fd07:b51a:cc66:a::/72"
        ],
        "providerID": "k3s://orbstack"
      },
      "status": {
        "capacity": {
          "cpu": "8",
          "ephemeral-storage": "200912352Ki",
          "memory": "8186988Ki",
          "pods": "110"
        },
        "allocatable": {
          "cpu": "8",
          "ephemeral-storage": "195447535873",
          "memory": "8186988Ki",
          "pods": "110"
        },
        "conditions": [
          {
            "type": "MemoryPressure",
            "status": "False",
            "lastHeartbeatTime": "2026-03-27T09:04:38Z",
            "lastTransitionTime": "2026-02-28T11:01:34Z",
            "reason": "KubeletHasSufficientMemory",
            "message": "kubelet has sufficient memory available"
          },
          {
            "type": "DiskPressure",
            "status": "False",
            "lastHeartbeatTime": "2026-03-27T09:04:38Z",
            "lastTransitionTime": "2026-02-28T11:01:34Z",
            "reason": "KubeletHasNoDiskPressure",
            "message": "kubelet has no disk pressure"
          },
          {
            "type": "PIDPressure",
            "status": "False",
            "lastHeartbeatTime": "2026-03-27T09:04:38Z",
            "lastTransitionTime": "2026-02-28T11:01:34Z",
            "reason": "KubeletHasSufficientPID",
            "message": "kubelet has sufficient PID available"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastHeartbeatTime": "2026-03-27T09:04:38Z",
            "lastTransitionTime": "2026-02-28T11:01:34Z",
            "reason": "KubeletReady",
            "message": "kubelet is posting ready status"
          }
        ],
        "addresses": [
          {
            "type": "InternalIP",
            "address": "192.168.139.2"
          },
          {
            "type": "InternalIP",
            "address": "fd07:b51a:cc66::2"
          },
          {
            "type": "Hostname",
            "address": "orbstack"
          }
        ],
        "daemonEndpoints": {
          "kubeletEndpoint": {
            "Port": 10250
          }
        },
        "nodeInfo": {
          "machineID": "82ddc538b9e95ae7c1d309201ccb5f9d",
          "systemUUID": "82ddc538b9e95ae7c1d309201ccb5f9d",
          "bootID": "e8cef0c9-8216-4648-9d2a-91101a28853b",
          "kernelVersion": "6.17.8-orbstack-00308-g8f9c941121b1",
          "osImage": "OrbStack",
          "containerRuntimeVersion": "docker://28.5.2",
          "kubeletVersion": "v1.33.5+orb1",
          "kubeProxyVersion": "",
          "operatingSystem": "linux",
          "architecture": "arm64",
          "swap": {
            "capacity": 9457209344
          }
        },
        "images": [
          {
            "names": [
              "kindest/node@sha256:c48c62eac5da28cdadcf560d1d8616cfa6783b58f0d94cf63ad1bf49600cb027"
            ],
            "sizeBytes": 1054561294
          },
          {
            "names": [
              "kindest/node@sha256:452d707d4862f52530247495d180205e029056831160e22870e37e3f6c1ac31f"
            ],
            "sizeBytes": 929926754
          },
          {
            "names": [
              "nginx@sha256:0236ee02dcbce00b9bd83e0f5fbc51069e7e1161bd59d99885b3ae1734f3392e",
              "nginx:latest"
            ],
            "sizeBytes": 180544671
          },
          {
            "names": [
              "ubuntu@sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9",
              "ubuntu:latest"
            ],
            "sizeBytes": 100717377
          },
          {
            "names": [
              "rancher/local-path-provisioner@sha256:80496fdeb307541007621959aa13aed41d31db9cd2dc4167c19833e0bfa3878c",
              "rancher/local-path-provisioner:v0.0.31"
            ],
            "sizeBytes": 59638920
          },
          {
            "names": [
              "rancher/mirrored-coredns-coredns@sha256:a11fafae1f8037cbbd66c5afa40ba2423936b72b4fd50a7034a7e8b955163594",
              "rancher/mirrored-coredns-coredns:1.10.1"
            ],
            "sizeBytes": 51383929
          },
          {
            "names": [
              "busybox@sha256:b3255e7dfbcd10cb367af0d409747d511aeb66dfac98cf30e97e87e4207dd76f",
              "busybox:latest"
            ],
            "sizeBytes": 4105254
          },
          {
            "names": [
              "rancher/mirrored-pause@sha256:74c4244427b7312c5b901fe0f67cbc53683d06f4f24c6faee65d4182bf0fa893",
              "rancher/mirrored-pause:3.6"
            ],
            "sizeBytes": 483864
          }
        ]
      }
    }
  ]
}%                                                                                                                                                                                                           
```



## Getting familiar with your cluster

### Get

We can use kubectl api-resources to display all resources available on our cluster


```sh
kubectl api-resources
NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
bindings                                         v1                                true         Binding
componentstatuses                   cs           v1                                false        ComponentStatus
configmaps                          cm           v1                                true         ConfigMap
endpoints                           ep           v1                                true         Endpoints
events                              ev           v1                                true         Event
limitranges                         limits       v1                                true         LimitRange
namespaces                          ns           v1                                false        Namespace
nodes                               no           v1                                false        Node
persistentvolumeclaims              pvc          v1                                true         PersistentVolumeClaim
persistentvolumes                   pv           v1                                false        PersistentVolume
pods                                po           v1                                true         Pod
podtemplates                                     v1                                true         PodTemplate
replicationcontrollers              rc           v1                                true         ReplicationController
resourcequotas                      quota        v1                                true         ResourceQuota
secrets                                          v1                                true         Secret
serviceaccounts                     sa           v1                                true         ServiceAccount
services                            svc          v1                                true         Service
mutatingwebhookconfigurations                    admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
validatingadmissionpolicies                      admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicyBinding
validatingwebhookconfigurations                  admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
apiservices                                      apiregistration.k8s.io/v1         false        APIService
controllerrevisions                              apps/v1                           true         ControllerRevision
daemonsets                          ds           apps/v1                           true         DaemonSet
deployments                         deploy       apps/v1                           true         Deployment
replicasets                         rs           apps/v1                           true         ReplicaSet
statefulsets                        sts          apps/v1                           true         StatefulSet
selfsubjectreviews                               authentication.k8s.io/v1          false        SelfSubjectReview
tokenreviews                                     authentication.k8s.io/v1          false        TokenReview
localsubjectaccessreviews                        authorization.k8s.io/v1           true         LocalSubjectAccessReview
selfsubjectaccessreviews                         authorization.k8s.io/v1           false        SelfSubjectAccessReview
selfsubjectrulesreviews                          authorization.k8s.io/v1           false        SelfSubjectRulesReview
subjectaccessreviews                             authorization.k8s.io/v1           false        SubjectAccessReview
horizontalpodautoscalers            hpa          autoscaling/v2                    true         HorizontalPodAutoscaler
cronjobs                            cj           batch/v1                          true         CronJob
jobs                                             batch/v1                          true         Job
certificatesigningrequests          csr          certificates.k8s.io/v1            false        CertificateSigningRequest
leases                                           coordination.k8s.io/v1            true         Lease
endpointslices                                   discovery.k8s.io/v1               true         EndpointSlice
events                              ev           events.k8s.io/v1                  true         Event
flowschemas                                      flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
prioritylevelconfigurations                      flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
helmchartconfigs                                 helm.cattle.io/v1                 true         HelmChartConfig
helmcharts                                       helm.cattle.io/v1                 true         HelmChart
addons                                           k3s.cattle.io/v1                  true         Addon
etcdsnapshotfiles                                k3s.cattle.io/v1                  false        ETCDSnapshotFile
ingressclasses                                   networking.k8s.io/v1              false        IngressClass
ingresses                           ing          networking.k8s.io/v1              true         Ingress
ipaddresses                         ip           networking.k8s.io/v1              false        IPAddress
networkpolicies                     netpol       networking.k8s.io/v1              true         NetworkPolicy
servicecidrs                                     networking.k8s.io/v1              false        ServiceCIDR
runtimeclasses                                   node.k8s.io/v1                    false        RuntimeClass
poddisruptionbudgets                pdb          policy/v1                         true         PodDisruptionBudget
clusterrolebindings                              rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                     rbac.authorization.k8s.io/v1      false        ClusterRole
rolebindings                                     rbac.authorization.k8s.io/v1      true         RoleBinding
roles                                            rbac.authorization.k8s.io/v1      true         Role
priorityclasses                     pc           scheduling.k8s.io/v1              false        PriorityClass
csidrivers                                       storage.k8s.io/v1                 false        CSIDriver
csinodes                                         storage.k8s.io/v1                 false        CSINode
csistoragecapacities                             storage.k8s.io/v1                 true         CSIStorageCapacity
storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass
volumeattachments                                storage.k8s.io/v1                 false        VolumeAttachment
```



Lets display all pods

```sh
kubectl get pods -A 
NAMESPACE     NAME                                      READY   STATUS    RESTARTS        AGE
kube-system   coredns-6cc96b5c97-n2mj5                  1/1     Running   3 (3d16h ago)   26d
kube-system   local-path-provisioner-774c6665dc-bxcds   1/1     Running   3 (3d16h ago)   26d
```


We can also show all labels

```sh
kubectl get pods -n kube-system --show-labels
NAME                                      READY   STATUS    RESTARTS        AGE   LABELS
coredns-6cc96b5c97-n2mj5                  1/1     Running   3 (3d16h ago)   26d   k8s-app=kube-dns,pod-template-hash=6cc96b5c97
local-path-provisioner-774c6665dc-bxcds   1/1     Running   3 (3d16h ago)   26d   app=local-path-provisioner,pod-template-hash=774c6665dc
```


We can select a specific label:

```sh
kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS        AGE
coredns-6cc96b5c97-n2mj5   1/1     Running   3 (3d16h ago)   26d
```



## ETCD 


Lets check our cloud cluster current state:

```sh
kubectl get pods -A
NAMESPACE      NAME                             READY   STATUS    RESTARTS      AGE
kube-flannel   kube-flannel-ds-bfplp            1/1     Running   6 (16m ago)   5d22h
kube-flannel   kube-flannel-ds-dmbc2            1/1     Running   1 (16m ago)   5d22h
kube-flannel   kube-flannel-ds-srhkt            1/1     Running   1 (16m ago)   5d22h
kube-system    coredns-7d764666f9-lcr8w         1/1     Running   1 (16m ago)   5d22h
kube-system    coredns-7d764666f9-pfgrb         1/1     Running   1 (16m ago)   5d22h
kube-system    etcd-master                      1/1     Running   1 (16m ago)   5d22h
kube-system    kube-apiserver-master            1/1     Running   1 (16m ago)   5d22h
kube-system    kube-controller-manager-master   1/1     Running   1 (16m ago)   5d22h
kube-system    kube-proxy-6rjlq                 1/1     Running   1 (16m ago)   5d22h
kube-system    kube-proxy-n4kn9                 1/1     Running   1 (16m ago)   5d22h
kube-system    kube-proxy-vzskd                 1/1     Running   1 (16m ago)   5d22h
kube-system    kube-scheduler-master            1/1     Running   1 (16m ago)   5d22h
```



As we see the etcd is running as pod in our master 

```sh
kubectl get pods -n kube-system etcd-master -o wide
NAME          READY   STATUS    RESTARTS      AGE     IP               NODE     NOMINATED NODE   READINESS GATES
etcd-master   1/1     Running   1 (19m ago)   5d22h   172.31.108.209   master   <none>           <none>
```


Lets describe 

```sh
kubectl get pod -n kube-system etcd-master -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://172.31.108.209:2379
    kubernetes.io/config.hash: e34d44f81dd9e44e6490e48aba2bf98a
    kubernetes.io/config.mirror: e34d44f81dd9e44e6490e48aba2bf98a
    kubernetes.io/config.seen: "2026-03-21T11:51:47.406270261Z"
    kubernetes.io/config.source: file
  creationTimestamp: "2026-03-21T11:51:47Z"
  generation: 1
  labels:
    component: etcd
    tier: control-plane
  name: etcd-master
  namespace: kube-system
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: master
    uid: 7984c4c5-d38f-45d3-993b-6f93b33c1a74
  resourceVersion: "14249"
  uid: 00a9cb16-7e0f-455c-92b7-5c7815c6d5b9
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://172.31.108.209:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --feature-gates=InitialCorruptCheck=true
    - --initial-advertise-peer-urls=https://172.31.108.209:2380
    - --initial-cluster=master=https://172.31.108.209:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://172.31.108.209:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://172.31.108.209:2380
    - --name=master
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --watch-progress-notify-interval=5s
    image: registry.k8s.io/etcd:3.6.6-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /livez
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    name: etcd
    ports:
    - containerPort: 2381
      hostPort: 2381
      name: probe-port
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      periodSeconds: 1
      successThreshold: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostNetwork: true
  nodeName: master
  preemptionPolicy: PreemptLowerPriority
  priority: 2000001000
  priorityClassName: system-node-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    operator: Exists
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-03-27T10:04:26Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-03-27T10:04:21Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-03-27T10:04:42Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-03-27T10:04:42Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-03-27T10:04:21Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - allocatedResources:
      cpu: 100m
      memory: 100Mi
    containerID: containerd://6b6c467f09a5337d0b40d2126c831e5b92ec0c8522e65626d8348bf336973f48
    image: registry.k8s.io/etcd:3.6.6-0
    imageID: registry.k8s.io/etcd@sha256:60a30b5d81b2217555e2cfb9537f655b7ba97220b99c39ee2e162a7127225890
    lastState:
      terminated:
        containerID: containerd://22cf39c99e2a6c248b77a22ea216190b41094df0154c0eb90cefcc6ed38d9fc1
        exitCode: 255
        finishedAt: "2026-03-27T10:04:17Z"
        reason: Unknown
        startedAt: "2026-03-21T11:51:39Z"
    name: etcd
    ready: true
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    restartCount: 1
    started: true
    state:
      running:
        startedAt: "2026-03-27T10:04:25Z"
    user:
      linux:
        gid: 0
        supplementalGroups:
        - 0
        uid: 0
  hostIP: 172.31.108.209
  hostIPs:
  - ip: 172.31.108.209
  phase: Running
  podIP: 172.31.108.209
  podIPs:
  - ip: 172.31.108.209
  qosClass: Burstable
  startTime: "2026-03-27T10:04:21Z"
```


Important information on this above output is to look the following line:

```sh
    - --advertise-client-urls=https://172.31.108.209:2379
```

See that our ETCD is running on port 2379 on master node

```sh
cloud_user@master:~$ kubectl get no master -o wide
NAME     STATUS   ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master   Ready    control-plane   5d22h   v1.35.3   172.31.108.209   <none>        Ubuntu 22.04.5 LTS   6.8.0-1050-aws   containerd://2.2.2
```

As we see we are connected to our master at the moment:

```sh
cloud_user@master:~$ ip a | grep 172.31.108.209
    inet 172.31.108.209/20 metric 100 brd 172.31.111.255 scope global dynamic ens5
```


Lets check and investigate this port 2379with ss:

```sh
sudo ss -lnp | grep 2379
tcp   LISTEN 0      4096                                                                       172.31.108.209:2379             0.0.0.0:*    users:(("etcd",pid=1501,fd=7))                                  
tcp   LISTEN 0      4096                                                                            127.0.0.1:2379             0.0.0.0:*    users:(("etcd",pid=1501,fd=6))                                  
```


Lets try to access it:

```sh
curl https://127.0.0.1:2379 -k
curl: (56) OpenSSL SSL_read: error:0A00045C:SSL routines::tlsv13 alert certificate required, errno 0
```

As we see the curl is able to reach the ETCD on port 2379 and it is getting an alert for certificate required 


We can also check with nc on port 2379:

```sh
cloud_user@master:~$ nc -v 172.31.108.209 2379
Connection to 172.31.108.209 2379 port [tcp/*] succeeded!
```



### Installing ETCDCTL


```sh
$ sudo apt install etcd-client
```

Checking the version:

```sh
etcdctl --version
etcdctl version: 3.3.25
API version: 2
```


Lets now check our certificates to access our ETCD server.
Kubernetes usually creates a dir /etc/kubernetes/pki (public key infrastructure), and everything will be stored there.

```sh
root@master:/etc/kubernetes/pki/etcd# ls -lh
total 32K
-rw-r--r-- 1 root root 1.1K Mar 21 11:51 ca.crt
-rw------- 1 root root 1.7K Mar 21 11:51 ca.key
-rw-r--r-- 1 root root 1.1K Mar 21 11:51 healthcheck-client.crt
-rw------- 1 root root 1.7K Mar 21 11:51 healthcheck-client.key
-rw-r--r-- 1 root root 1.2K Mar 21 11:51 peer.crt
-rw------- 1 root root 1.7K Mar 21 11:51 peer.key
-rw-r--r-- 1 root root 1.2K Mar 21 11:51 server.crt
-rw------- 1 root root 1.7K Mar 21 11:51 server.key

root@master:/etc/kubernetes/pki/etcd# pwd
/etc/kubernetes/pki/etcd
```


We usually need of 03 certificate files:

```
ca.crt
server.crt
server.key
```


Lets check more details for ca.crt file:

```sh
root@master:/etc/kubernetes/pki/etcd# openssl x509 -in ca.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 678919028918416121 (0x96c01427def72f9)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = etcd-ca
        Validity
            Not Before: Mar 21 11:46:10 2026 GMT
            Not After : Mar 18 11:51:10 2036 GMT
        Subject: CN = etcd-ca
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d0:64:a4:0e:f0:b2:a4:a8:88:03:98:cc:82:59:
                    2c:10:21:9e:6a:3e:f4:d1:e0:9f:dd:d3:0c:ec:b0:
                    bf:93:fc:d4:51:8a:cc:ba:52:17:1e:a7:b8:f1:5a:
                    41:2c:8f:db:f2:e7:69:3a:e1:e5:af:58:cd:ab:a0:
                    d1:ce:d4:1a:33:e7:5c:90:0d:72:db:1f:42:6e:a6:
                    93:0f:04:0b:e4:ee:09:be:57:ef:d9:be:8a:98:cc:
                    1a:c3:d9:6c:97:18:bd:ad:e0:dd:89:b8:dc:a1:72:
                    d2:97:93:f4:b0:c9:60:2f:53:fa:c6:71:98:4e:4c:
                    a5:01:e8:9c:e7:21:02:f9:42:ee:d6:61:3a:88:2b:
                    4f:5c:a4:15:67:37:99:1b:21:93:a2:8e:9b:48:f8:
                    94:82:19:a3:35:b8:e0:ac:f7:0c:22:aa:36:af:99:
                    56:8f:c5:c1:f5:3d:e2:74:74:bf:aa:e1:38:a2:5d:
                    1a:92:8c:e9:3a:64:39:55:b6:eb:d7:10:77:d4:28:
                    82:85:69:8b:f1:85:9c:2a:e5:49:25:69:b0:cf:98:
                    5d:33:20:d3:65:e6:f4:44:3c:67:b0:db:cb:77:ff:
                    c2:e5:0b:c0:0e:9a:19:7e:f9:18:57:3d:67:83:3e:
                    85:a7:a6:09:3c:2a:b3:19:7c:12:a9:f9:a7:4c:5c:
                    58:d3
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                D1:D1:F3:DC:7D:33:95:33:9C:C5:EC:DC:94:6A:82:4F:81:E3:4F:FE
            X509v3 Subject Alternative Name: 
                DNS:etcd-ca
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        33:4f:00:20:20:a8:b7:a2:59:5d:08:7a:5a:3a:a4:12:37:67:
        26:57:4d:19:cb:18:41:c7:b3:61:b3:c5:ff:4f:f0:d4:70:e4:
        11:30:4f:b2:ed:49:ea:0d:6c:20:03:98:5c:5e:4b:eb:bd:1c:
        a5:ce:17:fb:c1:ca:53:d4:a6:4f:12:be:5c:50:ec:db:85:40:
        ce:c7:17:29:88:5a:f3:2b:e7:95:d4:f1:af:d7:1c:f3:25:01:
        92:82:06:c1:3e:27:00:6a:aa:9f:91:ed:c9:36:dd:36:4a:aa:
        46:e4:03:ae:41:7a:99:41:2a:10:0d:6e:8b:85:18:fc:34:2a:
        67:97:bc:71:a3:e0:22:c8:73:52:87:a9:3a:ed:84:32:5d:9a:
        c3:77:c3:a5:c4:94:01:71:19:14:78:18:d7:98:0e:d1:25:25:
        3c:99:ad:22:32:9e:f3:10:93:f2:a8:b5:05:10:7c:92:14:72:
        28:62:cc:ce:7c:e6:e9:7c:4b:78:ab:22:75:e7:13:2e:cc:5b:
        46:64:d0:87:6a:c6:e0:6a:e5:96:7d:6c:15:02:69:74:89:65:
        f1:63:1f:ef:88:ef:a9:0d:28:ef:bc:49:0f:d3:89:05:8f:4d:
        31:93:4f:3a:29:04:3d:ce:b7:86:25:fc:e5:45:3c:f0:7b:a4:
        d2:c2:08:9c
-----BEGIN CERTIFICATE-----
MIIC/DCCAeSgAwIBAgIICWwBQn3vcvkwDQYJKoZIhvcNAQELBQAwEjEQMA4GA1UE
AxMHZXRjZC1jYTAeFw0yNjAzMjExMTQ2MTBaFw0zNjAzMTgxMTUxMTBaMBIxEDAO
BgNVBAMTB2V0Y2QtY2EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDQ
ZKQO8LKkqIgDmMyCWSwQIZ5qPvTR4J/d0wzssL+T/NRRisy6Uhcep7jxWkEsj9vy
52k64eWvWM2roNHO1Boz51yQDXLbH0JuppMPBAvk7gm+V+/ZvoqYzBrD2WyXGL2t
4N2JuNyhctKXk/SwyWAvU/rGcZhOTKUB6JznIQL5Qu7WYTqIK09cpBVnN5kbIZOi
jptI+JSCGaM1uOCs9wwiqjavmVaPxcH1PeJ0dL+q4TiiXRqSjOk6ZDlVtuvXEHfU
KIKFaYvxhZwq5UklabDPmF0zINNl5vREPGew28t3/8LlC8AOmhl++RhXPWeDPoWn
pgk8KrMZfBKp+adMXFjTAgMBAAGjVjBUMA4GA1UdDwEB/wQEAwICpDAPBgNVHRMB
Af8EBTADAQH/MB0GA1UdDgQWBBTR0fPcfTOVM5zF7NyUaoJPgeNP/jASBgNVHREE
CzAJggdldGNkLWNhMA0GCSqGSIb3DQEBCwUAA4IBAQAzTwAgIKi3olldCHpaOqQS
N2cmV00ZyxhBx7Nhs8X/T/DUcOQRME+y7UnqDWwgA5hcXkvrvRylzhf7wcpT1KZP
Er5cUOzbhUDOxxcpiFrzK+eV1PGv1xzzJQGSggbBPicAaqqfke3JNt02SqpG5AOu
QXqZQSoQDW6LhRj8NCpnl7xxo+AiyHNSh6k67YQyXZrDd8OlxJQBcRkUeBjXmA7R
JSU8ma0iMp7zEJPyqLUFEHySFHIoYszOfObpfEt4qyJ15xMuzFtGZNCHasbgauWW
fWwVAml0iWXxYx/viO+pDSjvvEkP04kFj00xk086KQQ9zreGJfzlRTzwe6TSwgic
-----END CERTIFICATE-----
```



```sh
root@master:/etc/kubernetes/pki/etcd# ETCDCTL_API=3 etcdctl --cacert $(pwd)/ca.crt --cert $(pwd)/server.crt --key=$(pwd)/server.key member list
165675e53dd94a40, started, master, https://172.31.108.209:2380, https://172.31.108.209:2379
```


We can also show in format of table:

```sh
root@master:/etc/kubernetes/pki/etcd# ETCDCTL_API=3 etcdctl --cacert $(pwd)/ca.crt --cert $(pwd)/server.crt --key=$(pwd)/server.key member list --write-out table
+------------------+---------+--------+-----------------------------+-----------------------------+
|        ID        | STATUS  |  NAME  |         PEER ADDRS          |        CLIENT ADDRS         |
+------------------+---------+--------+-----------------------------+-----------------------------+
| 165675e53dd94a40 | started | master | https://172.31.108.209:2380 | https://172.31.108.209:2379 |
+------------------+---------+--------+-----------------------------+-----------------------------+
```


### Set up ETCDCTL certificate variables


Lets create an etcdctl.env file with all variables pre-set up :

```sh
cat <<EOF> etcdctl.env
> export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
> export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
> export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
> export ETCDCeTL_API=3
> EOF
```

If you copy and paste the above command and it doesn't work and you won't see your variables set with the `env`  and your etcdctl member list may fail , perhaps you need to copy line by line avoid typing the '>' 

Recommended using this snippet as precaution:

```sh
cat <<EOF > etcdctl.env
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
export ETCDCTL_API=3
EOF
```



Checking the file:

```sh
root@master:~# cat etcdctl.env 
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
export ETCDCTL_API=3
```


Sending the env to memory:

```sh
root@master:~# source etcdctl.env 
root@master:~# env | grep ETCDCTL
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
ETCDCTL_API=3
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
```

As we see with source command we set up all variables to memory 

Now we can run etcdctl commands without having to set variables every time:

```sh
root@master:~# etcdctl member list --write-out table
+------------------+---------+--------+-----------------------------+-----------------------------+
|        ID        | STATUS  |  NAME  |         PEER ADDRS          |        CLIENT ADDRS         |
+------------------+---------+--------+-----------------------------+-----------------------------+
| 165675e53dd94a40 | started | master | https://172.31.108.209:2380 | https://172.31.108.209:2379 |
+------------------+---------+--------+-----------------------------+-----------------------------+
```


If we connect a new connection to server we are going to lose our variables configuration then we can run again $source etcdctl.env and run the etcdctl again and it will work.


#### ETCD-client

if you need to access your etcd pod we can install manually the etcd-client

```sh
docker exec -it comunidade-devops-control-plane sh
```

then we can install etcdctl to interact with our etcd server.

```sh
# apt install etcd-client
```


### ETCD Snapshot


Usually saved in 

```sh
/var/lib/etcd
```


We can check all options available when we run `etcdctl`  see that we have these 03 options:

```sh
snapshot restore
snapshot save
snapshot status
```


##### Snapshot save

```sh
root@comunidade-devops-control-plane:/var/lib/etcd# etcdctl snapshot save snapshot-andre.db
{"level":"info","ts":1774936291.4868507,"caller":"snapshot/v3_snapshot.go:119","msg":"created temporary db file","path":"snapshot-andre.db.part"}
{"level":"info","ts":"2026-03-31T05:51:31.495Z","caller":"clientv3/maintenance.go:200","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1774936291.4950914,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching snapshot","endpoint":"127.0.0.1:2379"}
{"level":"info","ts":"2026-03-31T05:51:31.553Z","caller":"clientv3/maintenance.go:208","msg":"completed snapshot read; closing"}
{"level":"info","ts":1774936291.5575054,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched snapshot","endpoint":"127.0.0.1:2379","size":"2.4 MB","took":0.070372445}
{"level":"info","ts":1774936291.5577567,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","path":"snapshot-andre.db"}
Snapshot saved at snapshot-andre.db
```


```sh
root@comunidade-devops-control-plane:/var/lib/etcd# ls -lh snapshot-andre.db
-rw------- 1 root root 2.3M Mar 31 05:51 snapshot-andre.db
```


##### Snapshot restore

```sh
root@comunidade-devops-control-plane:/var/lib/etcd# etcdctl  snapshot restore snapshot-andre.db 
{"level":"info","ts":1774936457.2102253,"caller":"snapshot/v3_snapshot.go:296","msg":"restoring snapshot","path":"snapshot-andre.db","wal-dir":"default.etcd/member/wal","data-dir":"default.etcd","snap-dir":"default.etcd/member/snap"}
{"level":"info","ts":1774936457.2317588,"caller":"mvcc/kvstore.go:388","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":1469634}
{"level":"info","ts":1774936457.2440696,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"cdf818194e3a8c32","local-member-id":"0","added-peer-id":"8e9e05c52164694d","added-peer-peer-urls":["http://localhost:2380"]}
{"level":"info","ts":1774936457.2586267,"caller":"snapshot/v3_snapshot.go:309","msg":"restored snapshot","path":"snapshot-andre.db","wal-dir":"default.etcd/member/wal","data-dir":"default.etcd","snap-dir":"default.etcd/member/snap"}
```




## YAML

This is an example of yaml file:

```yaml
# An employee record
name: Martin D'vloper
job: Developer 
skill: Elite 
employed: True 
foods:
  - ﻿﻿Apple
  - ﻿﻿Orange
  - ﻿﻿Strawberry
  - ﻿﻿Mango
languages:
  perl: Elite 
  python: Elite 
  pascal: Lame 
education: |
  4 GCSEs
  3 A-Levels
  BSc in the Internet of Things
```



