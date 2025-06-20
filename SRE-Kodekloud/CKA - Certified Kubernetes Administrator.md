

## Kube components 

- Master control
	- etcd
	- kube-api
	- scheduler
	- controller manager (replica, node)
- Worker node
	- kubelet
	- kube-proxy


## docker x Containerd

- Docker is the most dominant container tool
- kube launched and use docker as their container runtime
- Kube created CRI container runtime interface to allow all vendors to adapt for kube
- anyone can build container runtime 
- docker didn't adapt yourself for CRI
- kube created dockershim to allow docker continue to be CRT
- Containerd launch as a demon 
- Docker was removed from kube
- ctr cli very limited 
- nerdctl is container cli maintained by containerd
- crictl cli maintained by kubernetes community 

```sh
crictl
crictl pull busybox
crictl images
crictl ps -a
circtl exec -i -t container-id ls
crictl logs contianer-id
crictl pods
```


## ETCD

- ETCD is a distributed reliable key-value store, simple secure and fast.
- It is opposite to Relational Databases
- ETCD server on port 2379
- ETCD client etcdctl 


```sh
 kubectl exec -it -n kube-system etcd-kind-control-plane -- sh

etcdctl put key1 value1
etcdctl get key1
```


ETCDCTL is the CLI tool used to interact with ETCD.ETCDCTL can interact with ETCD Server using 2 API versions – Version 2 and Version 3. By default it’s set to use Version 2. Each version has different sets of commands.

For example, ETCDCTL version 2 supports the following commands:

```
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

Whereas the commands are different in version 3

```
etcdctl snapshot save
etcdctl endpoint health
etcdctl get
etcdctl put
```

To set the right version of API set the environment variable ETCDCTL_API command

```
export ETCDCTL_API=3
```

When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.

Apart from that, you must also specify the path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex:

```
--cacert /etc/kubernetes/pki/etcd/ca.crt
--cert /etc/kubernetes/pki/etcd/server.crt
--key /etc/kubernetes/pki/etcd/server.key
```

So for the commands, I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

```
kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / \
  --prefix --keys-only --limit=10 / \
  --cacert /etc/kubernetes/pki/etcd/ca.crt \
  --cert /etc/kubernetes/pki/etcd/server.crt \
  --key /etc/kubernetes/pki/etcd/server.key"
```



## Kube-api


Kubeadmin configure the Kube-api-server as a pod in kubernetes kube-system namespace

```sh
➜  ~ kubectl get pods -A | grep kube-apiserver
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          12d
```

and the api-server configuration files are stored in

```sh
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

When you configure kube-api-server manually as a service in any node the configuration file is:

```sh
cat /etc/systemd/system/kube-apiserver.service
```


kubectl command actually reach out to kube-api server, first it authenticate and then validate request and retrieve data.

We don't need kubectl to actually send commands to a cluster, we can use curl POST/GET request

Example:

```sh
curl -X POST /api/v1/namespaces/default/pod ... 
Pod created!
```


Kube-api-server receives your request, authenticate and create a POD.
Scheduler identifies that there is a pod with no node assigned then it quickly schedule a node to the pod, then it talks with Kubelet on that node.

We can check the process running for kube-api
```sh
➜  ~ ps aux | grep kube-api
degutos          87114   0.0  0.0 410733616   1568 s003  S+   10:27AM   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox --exclude-dir=.venv --exclude-dir=venv kube-api
```



## kube-controller-manager


kube-controller-manager is a resource in kubernetes that manages the cluster, it manages many controlls in kube. 


```sh
➜  ~ kubectl get pods -A | grep kube-controller-
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          12d
```


kube-controller-manager monitor nodes, it watches the system status and remediate situation, it monitors many components in the system to bring the status healthy and desirable state.

- It watch status
- Remediate situation
- It monitors nodes to make sure they are all running and take necessary steps.
- It uses kube-api server to reach out to nodes and check every 5 seconds that the nodes are healthy
- If a node is not sending heartbeat to node-controller every 5 seconds for 40 sec it marks it as `UNREACHEABLE` 
- After 5 min a node has 5m to became healthy again, if not back up it Evict the node assigning pods to go to a healthy node, if a pod is part of a ReplicaSet.


Other resources that kube-controller monitors:

- Deployment controller
- Namespace controller
- Endpoint Controller
- CronJob
- ReplicaSet 
- Replication Controller
- Service acct controller
- others


## Kube-scheduler


Kube-scheduler allow pods to be assigned to nodes, it only decides where the pod will be put which node. It doesn't place the pod in the node. The `kubelet` actually put the pod in the node.

Kube-scheduler decides where to place pods, which pod is the best to be put in depending on criteria. If a pod has need of 10 CPU the scheduler will eliminate all nodes with less cores and will decide which node has at least 10 CPU available.


## Kubelet

Kubelet is a daemon that run on worker node, responsible for placing a pod in the node, but not deciding which node to place. it monitors the node and pods in the node, send current report to kube-controller. It actually talk with container runtime in the node to place a pod
It monitors pods and node and talk with kube-controller through the kube-api. 
Kubelet is usually not deployed by kubeadm, we will need install manually


## Kube-proxy

- In a Kubernetes world every pod can communicate with other pod
- Each pod has an IP address 
- Pods IPs always change when the pod dies and re-create.
- We need create a service to allow PODs to talks with other PODs
- Services IPs are different Network that Pods IPs.
- Services are accessible through NAMES (kube-dns)
- kube-proxy is a service that run on every kubernete nodes
- Its job is to look for new service it create a rule in the nodes with IPTABLES rules. Each Node in the cluster will have Iptables rules created for each service created in the cluster




## Pod


```sh
➜  k run nginx --image=nginx
pod/nginx created

➜  k get pods nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          11s
```


- Lets get the amount of pods running

```sh
kubectl get pods --no-headers | wc -l
4
```


- Lets check the pod image

```sh
➜  kubectl describe po newpods-gjnnw | grep -i image:
    Image:         busybox
```


- Lets check which node those Pods are running on

```sh
➜  kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
newpods-gjnnw   1/1     Running   0          7m16s   10.22.0.9    controlplane   <none>           <none>
newpods-hrs82   1/1     Running   0          7m16s   10.22.0.10   controlplane   <none>           <none>
newpods-xb7fj   1/1     Running   0          7m16s   10.22.0.11   controlplane   <none>           <none>
nginx           1/1     Running   0          5m14s   10.22.0.13   controlplane   <none>           <none>
```


- Lets check Pod webapp which is in Error state

```sh
➜  kubectl get pods webapp
NAME     READY   STATUS             RESTARTS   AGE
webapp   1/2     ImagePullBackOff   0          92s
```


- Lets describe Pod webapp and check the events and status


```sh
➜  kubectl describe po webapp 
Name:             webapp
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.100.149
Start Time:       Sun, 15 Jun 2025 05:58:49 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.22.0.14
IPs:
  IP:  10.22.0.14
Containers:
  nginx:
    Container ID:   containerd://5a54e7ae5ef8cbd7aeb6f73a3b95e460565a91e60f4840e141faa0ef3a1c2215
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:6784fb0834aa7dbbe12e3d7471e69c290df3e6ba810dc38b34ae33d3c1c05f7d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 15 Jun 2025 05:58:50 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ks7w2 (ro)
  agentx:
    Container ID:   
    Image:          agentx
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ks7w2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-ks7w2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  51s                default-scheduler  Successfully assigned default/webapp to controlplane
  Normal   Pulling    51s                kubelet            Pulling image "nginx"
  Normal   Pulled     51s                kubelet            Successfully pulled image "nginx" in 159ms (159ms including waiting). Image size: 72406859 bytes.
  Normal   Created    51s                kubelet            Created container: nginx
  Normal   Started    51s                kubelet            Started container nginx
  Normal   BackOff    24s (x2 over 50s)  kubelet            Back-off pulling image "agentx"
  Warning  Failed     24s (x2 over 50s)  kubelet            Error: ImagePullBackOff
  Normal   Pulling    10s (x3 over 51s)  kubelet            Pulling image "agentx"
  Warning  Failed     9s (x3 over 50s)   kubelet            Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     9s (x3 over 50s)   kubelet            Error: ErrImagePull
```



As we see the current state of a container agentx

```yaml
 agentx:
    Container ID:   
    Image:          agentx
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
```


and lets check the Events now:

```sh
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  51s                default-scheduler  Successfully assigned default/webapp to controlplane
  Normal   Pulling    51s                kubelet            Pulling image "nginx"
  Normal   Pulled     51s                kubelet            Successfully pulled image "nginx" in 159ms (159ms including waiting). Image size: 72406859 bytes.
  Normal   Created    51s                kubelet            Created container: nginx
  Normal   Started    51s                kubelet            Started container nginx
  Normal   BackOff    24s (x2 over 50s)  kubelet            Back-off pulling image "agentx"
  Warning  Failed     24s (x2 over 50s)  kubelet            Error: ImagePullBackOff
  Normal   Pulling    10s (x3 over 51s)  kubelet            Pulling image "agentx"
  Warning  Failed     9s (x3 over 50s)   kubelet            Failed to pull image "agentx": failed to pull and unpack image "docker.io/library/agentx:latest": failed to resolve reference "docker.io/library/agentx:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     9s (x3 over 50s)   kubelet            Error: ErrImagePull
```


- Lets delete the Pod webapp

```sh
➜  kubectl get pods webapp
NAME     READY   STATUS             RESTARTS   AGE
webapp   1/2     ImagePullBackOff   0          6m1s

controlplane ~ ➜  kubectl delete po webapp
pod "webapp" deleted
```


- Lets create a new Pod called redis with image redis123


```sh
kubectl run redis --image=redis123
pod/redis created
```

```sh
➜  kubectl get pods redis
NAME    READY   STATUS         RESTARTS   AGE
redis   0/1     ErrImagePull   0          35s
```


- Lets now create a new Pod redis using a YAML file

```sh
➜  kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml

➜  cat redis.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis123
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


➜  kubectl apply -f redis.yaml 
pod/redis created

➜  kubectl get po redis
NAME    READY   STATUS         RESTARTS   AGE
redis   0/1     ErrImagePull   0          13s
```


- Lets fix the image to redis and apply the changes

```sh
➜  grep image: redis.yaml 
  - image: redis

➜  kubectl apply -f redis.yaml 
pod/redis configured

 ➜  kubectl get pods redis
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          82s
```



## Replication Controller and ReplicaSet

- Replication Controller - old 
- ReplicaSet - new 

- Checking current ReplicaSet

```sh
➜  kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       33s
```


- Checking pods

```sh
➜  kubectl get pods
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-grtc7   0/1     ImagePullBackOff   0          75s
new-replica-set-hrjnh   0/1     ImagePullBackOff   0          75s
new-replica-set-mmwxn   0/1     ImagePullBackOff   0          75s
new-replica-set-sdq7s   0/1     ImagePullBackOff   0          75s
```



```sh
➜  kubectl describe po new-replica-set-grtc7 | grep -i image:
    Image:         busybox777
```


- Lets check the pods events

```sh
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m37s                default-scheduler  Successfully assigned default/new-replica-set-grtc7 to controlplane
  Normal   Pulling    60s (x4 over 2m37s)  kubelet            Pulling image "busybox777"
  Warning  Failed     59s (x4 over 2m37s)  kubelet            Failed to pull image "busybox777": failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     59s (x4 over 2m37s)  kubelet            Error: ErrImagePull
  Normal   BackOff    3s (x9 over 2m37s)   kubelet            Back-off pulling image "busybox777"
  Warning  Failed     3s (x9 over 2m37s)   kubelet            Error: ImagePullBackOff
```



- Lets check and deploy a new deployment

```sh
➜  cat replicaset-definition-1.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```



```sh
➜  kubectl apply -f replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created
```



- Deleting a ReplicaSet

```sh
➜  kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       12m
replicaset-1      2         2         2       2m34s
replicaset-2      2         2         2       19s

controlplane ~ ➜  kubectl delete rs replicaset-1 replicaset-2 
replicaset.apps "replicaset-1" deleted
replicaset.apps "replicaset-2" deleted
```


- Lets scale to 5 replicas our replicaset

```sh
kubectl scale rs new-replica-set --replicas=5
replicaset.apps/new-replica-set scaled
```




## Kube tips


Create an NGINX Pod

```
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)

```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Create a deployment

```
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```
kubectl create -f nginx-deployment.yaml
```

OR

In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.

```
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```


## Kubectl explain

- Use kubectl explain to get help and more details on some kube resource command.

```sh
➜  kubectl explain deployment 
```


## Create a deployment from yaml file


```sh
➜  kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3 --dry-run=client -o yaml 

➜  kubectl apply -f deploy.yaml 
deployment.apps/httpd-frontend created
```

```sh
 ➜  kubectl get deploy
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1          2/2     2            2           3m28s
frontend-deployment   0/4     4            0           10m
httpd-frontend        3/3     3            3           8s

controlplane ~ ➜  kubectl get pods
NAME                                 READY   STATUS             RESTARTS   AGE
deployment-1-858c547685-l4l9k        1/1     Running            0          3m36s
deployment-1-858c547685-msf9b        1/1     Running            0          3m36s
frontend-deployment-cd6b557c-49b59   0/1     ImagePullBackOff   0          10m
frontend-deployment-cd6b557c-kfhr2   0/1     ImagePullBackOff   0          10m
frontend-deployment-cd6b557c-pvxsk   0/1     ImagePullBackOff   0          10m
frontend-deployment-cd6b557c-wd8pc   0/1     ImagePullBackOff   0          10m
httpd-frontend-86b5794b6d-dgtcn      1/1     Running            0          16s
httpd-frontend-86b5794b6d-hb9c8      1/1     Running            0          16s
httpd-frontend-86b5794b6d-nmv8c      1/1     Running            0          16s

```



## Services

- NodePort: allow a service to enable a port on the node and forward connection to a pod port
- ClusterIP: allow backend communication between services to services 
- LoadBalancer: allow a service to balance workload between back end pods 


### NodePort


- TargetPort
- Port
- NodePort [3000:32767]


```yml
apiVersion: v1
kind: Service
metadata: 
   name: myapp-service
spec:
	type: NodePort
	ports:
	 - targetPort: 80
	   port: 80
	   nodePort: 30008 # mandatory field, auto provided if missing
	selector:
	app: myapp
	type: front-end
```



```sh
kubectl create -f service-definition.yaml
```



### ClusterIp

```yaml
apiVersion: v1
kind: Service
metadata: 
	name: back-end
spec: 
	type: ClusterIP
	ports: 
	 - targetPort: 80
	   port: 80
	selector: 
	  app: myapp
	  type: back-end
```



### Services Lab


```sh
➜  kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   7m33s
```


```sh
➜  kubectl describe svc kubernetes
Name:                     kubernetes
Namespace:                default
Labels:                   component=apiserver
                          provider=kubernetes
Annotations:              <none>
Selector:                 <none>
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.0.1
IPs:                      10.43.0.1
Port:                     https  443/TCP
TargetPort:               6443/TCP
Endpoints:                192.168.58.171:6443
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

Notice the targetPort is 6443 and Endpoint is 192.168.58.171:6443


- Lets check the deployment

```sh
➜  kubectl get deployments
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
simple-webapp-deployment   4/4     4            4           21s
```


```sh
➜  kubectl describe deployments simple-webapp-deployment
Name:                   simple-webapp-deployment
Namespace:              default
CreationTimestamp:      Mon, 16 Jun 2025 19:40:19 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=simple-webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=simple-webapp
  Containers:
   simple-webapp:
    Image:         kodekloud/simple-webapp:red
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   simple-webapp-deployment-8555484b96 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  73s   deployment-controller  Scaled up replica set simple-webapp-deployment-8555484b96 from 0 to 4
```



```sh
➜  cat service-definition-1.yaml 
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports:
  - nodePort: 30080
    port: 8080
    targetPort: 8080 
  selector:
    name: simple-webapp
  type: NodePort


➜  kubectl apply -f service-definition-1.yaml 
service/webapp-service created



controlplane ~ ➜  kubectl get svc webapp-service
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
webapp-service   NodePort   10.43.46.131   <none>        8080:30080/TCP   19s

controlplane ~ ➜  kubectl describe svc webapp-service
Name:                     webapp-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 name=simple-webapp
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.46.131
IPs:                      10.43.46.131
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                10.22.0.10:8080,10.22.0.11:8080,10.22.0.12:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>

```



### Namespaces


Lets check all namespaces

```sh
➜  kubectl get namespaces
NAME              STATUS   AGE
default           Active   6m51s
dev               Active   34s
finance           Active   34s
kube-node-lease   Active   6m51s
kube-public       Active   6m52s
kube-system       Active   6m52s
manufacturing     Active   34s
marketing         Active   34s
prod              Active   34s
research          Active   34s
```


Lets now count amount of namespaces

```sh
➜  kubectl get namespaces --no-headers | wc -l
10
```


Lets check pods in research namespace

```sh
➜  kubectl get pods -n research
NAME    READY   STATUS             RESTARTS        AGE
dna-1   0/1     CrashLoopBackOff   5 (2m40s ago)   5m40s
dna-2   0/1     CrashLoopBackOff   5 (2m34s ago)   5m40s
```



Create a pod in finance namespace finance

```sh
➜  kubectl run redis --image=redis -n finance 
pod/redis created

controlplane ~ ➜  kubectl get pods -n finance
NAME      READY   STATUS    RESTARTS   AGE
payroll   1/1     Running   0          9m59s
redis     1/1     Running   0          43s

```


Lets check now all namespaces for a pod blue

```sh
✖ kubectl get pods -A | grep blue
marketing       blue                                      1/1     Running            0             12m
```


Lets list now all pods and services in marketing namespace

```sh
 ➜  kubectl get pods -n marketing
NAME       READY   STATUS    RESTARTS   AGE
blue       1/1     Running   0          14m
redis-db   1/1     Running   0          14m

➜  kubectl get svc -n marketing
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
blue-service   NodePort   10.43.53.239    <none>        8080:30082/TCP   16m
db-service     NodePort   10.43.211.240   <none>        6379:31246/TCP   16m
```

Note that blue and redis-db are in the same namespace. To access the redis-db the blue pod can reach by name db-service

Lets list now all services in dev namespaces

```sh
➜  kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
db-service   ClusterIP   10.43.2.83   <none>        6379/TCP   18m
```

Notice that to blue pod in marketing ns to access db-services in dev ns it has to use full address db-service.dev.svc.cluster.local



### POD

**Create an NGINX Pod**

```
kubectl run nginx --image=nginx
```

**Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)**

```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

### Deployment

**Create a deployment**

```
kubectl create deployment --image=nginx nginx
```

**Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)**

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

**Generate Deployment with 4 Replicas**

```
kubectl create deployment nginx --image=nginx --replicas=4
```

You can also scale a deployment using the

```
kubectl scale
```

command.

```
kubectl scale deployment nginx--replicas=4
```

**Another way to do this is to save the YAML definition to a file and modify**

```
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

You can then update the YAML file with the replicas or any other field before creating the deployment.

### Service

**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**

```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

(This will automatically use the pod’s labels as selectors)

Or

```
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
```

(This will not use the pods labels as selectors, instead, it will assume selectors as **app=redis.**

[You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191)

So, it does not work very well if your pod has a different label set. So, generate the file and modify the selectors before creating the service)

**Create a Service named nginx of type NodePort to expose pod nginx’s port 80 on port 30080 on the nodes:**

```
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

(This will automatically use the pod’s labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port manually before creating the service with the pod.)

Or

```
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

(This will not use the pod labels as selectors.)

Both the above commands have their own challenges. While one of them cannot accept a selector, the other cannot accept a node port. I would recommend going with the

```
kubectl expose
```

command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

### **Reference:**

[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

[https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)

### Imperative commands Lab


```sh
➜  kubectl run nginx-pod --image=alpine
pod/nginx-pod created
```


Lets create a pod with label tier=db

```sh
 ➜  kubectl run redis --image=redis:alpine --labels=tier=db
pod/redis created
```


Lets expose a pod creating a service

```sh
➜  kubectl expose po redis --name=redis-service --port=6379
service/redis-service exposed
```

Lets create a deployment with 3 replicas

```sh
➜  kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
deployment.apps/webapp created
```


Lets create a pod on container port 8080

```sh
➜  kubectl run custom-nginx --image=nginx --port=8080
pod/custom-nginx created
```


Lets create a deployment redis-deploy in dev-ns namesapce with redis image and 2 replicas.

```sh
➜  kubectl create deploy redis-deploy -n dev-ns --image=redis --replicas=2
deployment.apps/redis-deploy created
```



Lets create a pod httpd and expose it as type clusterIP on port 80

```sh
➜  kubectl run httpd --image=httpd:alpine --port=80 --expose 
service/httpd created
pod/httpd created
```


### Scheduling 


Lets check a pod just started without a scheduler


```sh
➜  kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          5s
```



```sh
➜  kubectl describe po nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Containers:
  nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7mtts (ro)
Volumes:
  kube-api-access-7mtts:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```



Lets add nodeName 

```sh
➜  kubectl delete po nginx 
pod "nginx" deleted

# we can also use the command
$ kubectl replace --force -f nginx.yaml
```

Here is the content of the file with variable nodeName

```sh
➜  cat nginx.yaml 
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node01
  containers:
  -  image: nginx
     name: nginx
```


```sh
 ➜  kubectl apply -f nginx.yaml 
pod/nginx created
```


```sh
➜  kubectl get po -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          3m36s   172.17.1.2   node01   <none>           <none>
```


> [!Important] 
> > A pod can not migrate from a node to another node without being deleted.
> > Pod has a container which is a process in the compute node, so it cannot be transfer to another host
> > 



### Labels and Selector


Lets list all pods with theirs labels

```sh
➜  kubectl get pods --show-labels
NAME          READY   STATUS    RESTARTS   AGE     LABELS
app-1-489hj   1/1     Running   0          5m22s   bu=finance,env=dev,tier=frontend
app-1-8qt44   1/1     Running   0          5m22s   bu=finance,env=dev,tier=frontend
app-1-mfbrf   1/1     Running   0          5m22s   bu=finance,env=dev,tier=frontend
app-1-zzxdf   1/1     Running   0          5m22s   bu=finance,env=prod,tier=frontend
app-2-bxngv   1/1     Running   0          5m22s   env=prod,tier=frontend
auth          1/1     Running   0          5m22s   bu=finance,env=prod
db-1-jkhng    1/1     Running   0          5m22s   env=dev,tier=db
db-1-l97rj    1/1     Running   0          5m22s   env=dev,tier=db
db-1-njrbv    1/1     Running   0          5m22s   env=dev,tier=db
db-1-sbc5x    1/1     Running   0          5m22s   env=dev,tier=db
db-2-pl7rl    1/1     Running   0          5m22s   bu=finance,env=prod,tier=db
```


Lets show now only labels env=dev

```sh
 ➜  kubectl get pods --selector env=dev
NAME          READY   STATUS    RESTARTS   AGE
app-1-489hj   1/1     Running   0          6m31s
app-1-8qt44   1/1     Running   0          6m31s
app-1-mfbrf   1/1     Running   0          6m31s
db-1-jkhng    1/1     Running   0          6m31s
db-1-l97rj    1/1     Running   0          6m31s
db-1-njrbv    1/1     Running   0          6m31s
db-1-sbc5x    1/1     Running   0          6m31s
```


```sh
➜  kubectl get pods --no-headers --selector env=dev --show-labels
app-1-489hj   1/1   Running   0     8m26s   bu=finance,env=dev,tier=frontend
app-1-8qt44   1/1   Running   0     8m26s   bu=finance,env=dev,tier=frontend
app-1-mfbrf   1/1   Running   0     8m26s   bu=finance,env=dev,tier=frontend
db-1-jkhng    1/1   Running   0     8m26s   env=dev,tier=db
db-1-l97rj    1/1   Running   0     8m26s   env=dev,tier=db
db-1-njrbv    1/1   Running   0     8m26s   env=dev,tier=db
db-1-sbc5x    1/1   Running   0     8m26s   env=dev,tier=db
```


Lets count them

```sh
➜  kubectl get pods --no-headers --selector env=dev --show-labels | wc -l
7
```


Lets check now how many pods are in business unit (bu) finance.


```sh
➜  kubectl get pods --selector bu=finance
NAME          READY   STATUS    RESTARTS   AGE
app-1-489hj   1/1     Running   0          10m
app-1-8qt44   1/1     Running   0          10m
app-1-mfbrf   1/1     Running   0          10m
app-1-zzxdf   1/1     Running   0          10m
auth          1/1     Running   0          10m
db-2-pl7rl    1/1     Running   0          10m
```


```sh
➜  kubectl get pods --no-headers --selector bu=finance --show-labels 
app-1-489hj   1/1   Running   0     11m   bu=finance,env=dev,tier=frontend
app-1-8qt44   1/1   Running   0     11m   bu=finance,env=dev,tier=frontend
app-1-mfbrf   1/1   Running   0     11m   bu=finance,env=dev,tier=frontend
app-1-zzxdf   1/1   Running   0     11m   bu=finance,env=prod,tier=frontend
auth          1/1   Running   0     11m   bu=finance,env=prod
db-2-pl7rl    1/1   Running   0     11m   bu=finance,env=prod,tier=db
```


```sh
➜  kubectl get pods --no-headers --selector bu=finance --show-labels | wc -l
6
```


Lets now list all pods and replicasets and all other objects that belongs to environment prod.


```sh
➜  kubectl get all --selector env=prod
NAME              READY   STATUS    RESTARTS   AGE
pod/app-1-zzxdf   1/1     Running   0          12m
pod/app-2-bxngv   1/1     Running   0          12m
pod/auth          1/1     Running   0          12m
pod/db-2-pl7rl    1/1     Running   0          12m

NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.43.33.79   <none>        3306/TCP   12m

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/app-2   1         1         1       12m
replicaset.apps/db-2    1         1         1       12m
```


```sh
➜  kubectl get all --selector env=prod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
pod/app-1-zzxdf   1/1     Running   0          13m   bu=finance,env=prod,tier=frontend
pod/app-2-bxngv   1/1     Running   0          13m   env=prod,tier=frontend
pod/auth          1/1     Running   0          13m   bu=finance,env=prod
pod/db-2-pl7rl    1/1     Running   0          13m   bu=finance,env=prod,tier=db

NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE   LABELS
service/app-1   ClusterIP   10.43.33.79   <none>        3306/TCP   13m   bu=finance,env=prod

NAME                    DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/app-2   1         1         1       13m   env=prod
replicaset.apps/db-2    1         1         1       13m   env=prod
```


Lets now list a POD that belongs to environment prod, bu finance, and tier frontend.

```sh
➜  kubectl get pods --selector env=prod,bu=finance,tier=frontend --show-labels
NAME          READY   STATUS    RESTARTS   AGE   LABELS
app-1-zzxdf   1/1     Running   0          15m   bu=finance,env=prod,tier=frontend
```


Given a ReplicaSet yaml file

```yaml
➜  cat replicaset-definition-1.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        tier: nginx
     spec:
       containers:
       - name: nginx
         image: nginx
```


Notice that the selector label doesn't match with pod label. This will provide an error when applying this configuration.

```sh
➜  kubectl apply -f replicaset-definition-1.yaml 
The ReplicaSet "replicaset-1" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"tier":"nginx"}: `selector` does not match template `labels`
```


We need to use the same labels

```sh
➜  cat replicaset-definition-1.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        tier: front-end
     spec:
       containers:
       - name: nginx
         image: nginx
```


```sh
➜  kubectl apply -f replicaset-definition-1.yaml 
replicaset.apps/replicaset-1 created
```



### Taints and Tolerations



Lets check all our nodes

```sh
➜  kubectl get no
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   8m58s   v1.32.0
node01         Ready    <none>          8m22s   v1.32.0
```



If we describe the node we can see it has taint or not

```sh
➜  kubectl describe no node01 | grep -i taints
Taints:             <none>
```


Lets now taint the node node01 with key spray=mortein effect of NoSchedule

```sh
 ➜  kubectl taint node node01 spray=mortein:NoSchedule
node/node01 tainted

controlplane ~ ➜  kubectl describe no node01 | grep -i taints
Taints:             spray=mortein:NoSchedule
```

Lets now create a new pod mosquito with image nginx 

```sh
➜  kubectl run mosquito --image=nginx 
pod/mosquito created
```

Notice that the new Pod is in Pending state because it can not land on node01

```sh
➜  kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
mosquito   0/1     Pending   0          11s   <none>   <none>   <none>           <none>
```


Lets create a pod bee with nginx which has toleration set to taint mortein

```sh
➜  kubectl run bee --image=nginx --dry-run=client -o yaml > pod.yaml
```


```sh
 ➜  cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: spray
    value: mortein
    effect: NoSchedule
    operator: Equal
```


```sh
➜  kubectl create -f pod.yaml 
pod/bee created
```


```sh
➜  kubectl get pods -owide
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          93s   172.17.1.2   node01   <none>           <none>
mosquito   0/1     Pending   0          13m   <none>       <none>   <none>           <none>
```

As we see the pod bee has toleration to mortein

Describing the pod we can a session Tolerations

```sh
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
                             spray=mortein:NoSchedule
```


Lets now check the taint applied to the node controlplane

```sh
 ➜  kubectl describe no controlplane | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

Lets now remove the taint by editing the node configuration and removing the lines for the taint

```sh
 ➜  kubectl edit no controlplane
node/controlplane edited
```

```sh
➜  kubectl describe no controlplane | grep -i taints
Taints:             <none>
```

We can remove the taint also with the below command

```sh
➜  kubectl describe no controlplane | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

controlplane ~ ➜  kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted

➜  kubectl describe no controlplane | grep -i taints
Taints:             <none>

```


Notice that the pod mosquito went into running state in the node controlplane now

```sh
➜  kubectl get pods -owide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          4m21s   172.17.1.4   node01         <none>           <none>
mosquito   1/1     Running   0          26m     172.17.0.4   controlplane   <none>           <none>
```




### Node Selectors


- Label node with key and value, example: size=Large, size=Medium, size=Small, department=Marketing
- Add nodeSelector to your Pod yaml file under spec session
- Node selectors doesn't allow OR or NOT as limitation.



### Node Affinity


- Ensure pods are hosted in specific nodes. It is similar to Node Selectors but give us more flexibility and options.
- Allow us to use OR and NOT statement 


Lets check all labels for the node node01

```sh
➜  kubectl get nodes --show-labels node01
NAME     STATUS   ROLES    AGE   VERSION   LABELS
node01   Ready    <none>   82s   v1.32.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux
```


Lets now add a new label to node node01

```sh
 $ kubectl label no node01 color=blue
node/node01 labeled
```


Lets now create a new deployment named blue with the nginx image and 3 replicas.

```sh
➜  kubectl create deployment blue --image=nginx --replicas=3
deployment.apps/blue created
```


Lets check the 02 nodes we have in our cluster which one has taints

```sh
➜  kubectl describe no | grep -i taints
Taints:             <none>
Taints:             <none>
```


Because nodes has no taint Pods can run every node.

```sh
➜  kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-d7967dc55-5jqm4   1/1     Running   0          2m52s   172.17.1.2   node01         <none>           <none>
blue-d7967dc55-g26p7   1/1     Running   0          2m52s   172.17.1.3   node01         <none>           <none>
blue-d7967dc55-zk98t   1/1     Running   0          2m52s   172.17.0.4   controlplane   <none>           <none>
```


Lets Set Node Affinity to the deployment to place the pods on `node01` only.


```sh
➜  kubectl get deployment blue -o yaml > deployment.yaml
```

 
 ```yaml
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                  - blue
```


```sh
➜  kubectl apply -f deployment.yaml 
deployment.apps/blue created
```

See now that pods are running on node01

```sh
➜  kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
blue-7bd99994c-6qbvz   1/1     Running   0          55s   172.17.1.4   node01   <none>           <none>
blue-7bd99994c-p94jp   1/1     Running   0          55s   172.17.1.5   node01   <none>           <none>
blue-7bd99994c-wwqcm   1/1     Running   0          55s   172.17.1.6   node01   <none>           <none>
```


Notice that now because we have node01 with label color=blue the pods will land on it.

```sh

controlplane ~ ➜  kubectl get nodes node01 --show-labels
NAME     STATUS   ROLES    AGE   VERSION   LABELS
node01   Ready    <none>   20m   v1.32.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,color=blue,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux
```


Lets now create a deployment red with nginx image and 2 replicas

```sh
➜  kubectl create deployment red --image=nginx --replicas=2 
deployment.apps/red created


➜  kubectl get pods --selector app=red -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
red-5784d99ff8-bdph7   1/1     Running   0          4m18s   172.17.0.5   controlplane   <none>           <none>
red-5784d99ff8-m4nqn   1/1     Running   0          4m18s   172.17.1.7   node01         <none>           <none>
```


As we see the pod red went into both nodes. Lets allow them to land only on controlplane node.


```sh
 kubectl get no controlplane --show-labels
NAME           STATUS   ROLES           AGE   VERSION   LABELS
controlplane   Ready    control-plane   28m   v1.32.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=controlplane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
```

Lets use the already label add node-role.kubernetes.io/control-plane=

```sh
$ kubectl edit deployment red
deployment.apps/red edited
```


```sh
➜  kubectl get pods -o wide --selector app=red
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
red-7d56954959-47w6x   1/1     Running   0          18s   172.17.0.6   controlplane   <none>           <none>
red-7d56954959-sw9bt   1/1     Running   0          16s   172.17.0.7   controlplane   <none>           <none>
```



```yaml
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
      containers:
```
























