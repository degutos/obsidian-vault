

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

## kubectl api-resources

- Use kubectl api-resources to list all kubernetes resources within the API/etcd. It is good option when you don't remember a resource name you need to investigate.


## Kubectl explain

- Use kubectl explain to get help and more details on some kube resource command.

```sh
➜  kubectl explain deployment 
```

or 

```sh
$ kubectl explain pods
```


To go deeper and get more details explained use:

```sh
$ kubectl explain pods.spec
```

To get the entire list of commands use:

```sh
$ kubectl explain pods --recursive
```

To get the explanation of what is available in containers session:

```sh
➜  kubectl explain pod.spec.containers
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


Lets create a namespace

```sh
➜  kubectl create ns dev-ns
namespace/dev-ns created
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




### Resources Limit CPU and Memory



Given a pod Rabbit 

```sh
➜  kubectl describe po rabbit
Name:             rabbit
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.58.141
Start Time:       Sat, 21 Jun 2025 10:41:32 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:  10.22.0.9
Containers:
  cpu-stress:
    Container ID:  containerd://36e24b05920be99cf43d9eca9884dbd2eb9691303e5db737aadf0de8e2edd2dc
    Image:         ubuntu
    Image ID:      docker.io/library/ubuntu@sha256:b59d21599a2b151e23eea5f6602f4af4d7d31c4e236d22bf0b62b86d2e386b8f
    Port:          <none>
    Host Port:     <none>
    Args:
      sleep
      1000
    State:          Running
      Started:      Sat, 21 Jun 2025 10:41:36 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:  1
    Requests:
      cpu:        500m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5bxhg (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-5bxhg:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  33s   default-scheduler  Successfully assigned default/rabbit to controlplane
  Normal  Pulling    33s   kubelet            Pulling image "ubuntu"
  Normal  Pulled     31s   kubelet            Successfully pulled image "ubuntu" in 1.798s (1.798s including waiting). Image size: 29724744 bytes.
  Normal  Created    31s   kubelet            Created container: cpu-stress
  Normal  Started    30s   kubelet            Started container cpu-stress

```



> [!Important] 
> > Notice the current limits and Requests


```yaml
   Limits:
      cpu:  1
    Requests:
      cpu:        500m
```


As we see this pod has limit of 1 cpu and Request minimum of 500m. So 500m cpu is = to 0.5 cpu


Lets now delete the pod

```sh
➜  kubectl delete po rabbit
pod "rabbit" deleted
```


#### Crashloopbackoff



```sh
➜  kubectl get pods
NAME       READY   STATUS             RESTARTS      AGE
elephant   0/1     CrashLoopBackOff   1 (12s ago)   14s
```



```sh
➜  kubectl describe po elephant
Name:             elephant
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.58.141
Start Time:       Sat, 21 Jun 2025 10:48:47 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.10
IPs:
  IP:  10.22.0.10
Containers:
  mem-stress:
    Container ID:  containerd://2ff291109ed7f1a9e44474825f3dffb943d40edbff84ac38cc3e2c4fffa4e159
    Image:         polinux/stress
    Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    Port:          <none>
    Host Port:     <none>
    Command:
      stress
    Args:
      --vm
      1
      --vm-bytes
      15M
      --vm-hang
      1
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Sat, 21 Jun 2025 10:49:07 +0000
      Finished:     Sat, 21 Jun 2025 10:49:07 +0000
    Ready:          False
    Restart Count:  2
    Limits:
      memory:  10Mi
    Requests:
      memory:     5Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4mrht (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-4mrht:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  35s                default-scheduler  Successfully assigned default/elephant to controlplane
  Normal   Pulled     34s                kubelet            Successfully pulled image "polinux/stress" in 649ms (649ms including waiting). Image size: 4041495 bytes.
  Normal   Pulled     33s                kubelet            Successfully pulled image "polinux/stress" in 151ms (151ms including waiting). Image size: 4041495 bytes.
  Normal   Pulling    16s (x3 over 34s)  kubelet            Pulling image "polinux/stress"
  Normal   Created    15s (x3 over 34s)  kubelet            Created container: mem-stress
  Normal   Started    15s (x3 over 34s)  kubelet            Started container mem-stress
  Normal   Pulled     15s                kubelet            Successfully pulled image "polinux/stress" in 163ms (163ms including waiting). Image size: 4041495 bytes.
  Warning  BackOff    2s (x4 over 32s)   kubelet            Back-off restarting failed container mem-stress in pod elephant_default(36c3cb8f-6a72-4c52-a51c-efb3e2dbe43a)
```




Lets increase limit to 20Mi

```sh
   resources:
      limits:
        memory: 20Mi
```



with kubectl edit

```sh
➜  kubectl edit pod elephant 
error: pods "elephant" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-2914857219.yaml"
error: Edit cancelled, no valid changes were saved.
```

- Notice that we can change a limit in memory during the pod execution, so we need to save in a temp file delete the pod and run apply again 


```sh
$ kubectl delete po elephant 
pod "elephant" deleted

 ➜  kubectl apply -f /tmp/kubectl-edit-2914857219.yaml
pod/elephant created



 ➜  kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          6s

```


### DaemonSets


```sh
➜  kubectl get daemonsets --all-namespaces
NAMESPACE      NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   kube-flannel-ds   1         1         1       1            1           <none>                   5m58s
kube-system    kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   5m59s
```



```sh
➜  kubectl describe ds kube-proxy -n kube-system
Name:           kube-proxy
Selector:       k8s-app=kube-proxy
Node-Selector:  kubernetes.io/os=linux
Labels:         k8s-app=kube-proxy
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
Number of Nodes Scheduled with Up-to-date Pods: 1
Number of Nodes Scheduled with Available Pods: 1
Number of Nodes Misscheduled: 0
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           k8s-app=kube-proxy
  Service Account:  kube-proxy
  Containers:
   kube-proxy:
    Image:      registry.k8s.io/kube-proxy:v1.32.0
    Port:       <none>
    Host Port:  <none>
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
    Mounts:
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /var/lib/kube-proxy from kube-proxy (rw)
  Volumes:
   kube-proxy:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy
    Optional:  false
   xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
   lib-modules:
    Type:               HostPath (bare host directory volume)
    Path:               /lib/modules
    HostPathType:       
  Priority Class Name:  system-node-critical
  Node-Selectors:       kubernetes.io/os=linux
  Tolerations:          op=Exists
Events:
  Type    Reason            Age    From                  Message
  ----    ------            ----   ----                  -------
  Normal  SuccessfulCreate  8m23s  daemonset-controller  Created pod: kube-proxy-llcdx

```


- We can notice that this pod is schedule to run in just one node 


- Lets now describe DaemonSet kube-flannel-ds


```sh
kubectl describe ds -n kube-flannel kube-flannel-ds
Name:           kube-flannel-ds
Selector:       app=flannel,k8s-app=flannel
Node-Selector:  <none>
Labels:         app=flannel
                k8s-app=flannel
                tier=node
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
Number of Nodes Scheduled with Up-to-date Pods: 1
Number of Nodes Scheduled with Available Pods: 1
Number of Nodes Misscheduled: 0
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           app=flannel
                    k8s-app=flannel
                    tier=node
  Service Account:  flannel
  Init Containers:
   install-cni-plugin:
    Image:      docker.io/flannel/flannel-cni-plugin:v1.2.0
    Port:       <none>
    Host Port:  <none>
    Command:
      cp
    Args:
      -f
      /flannel
      /opt/cni/bin/flannel
    Environment:  <none>
    Mounts:
      /opt/cni/bin from cni-plugin (rw)
   install-cni:
    Image:      docker.io/flannel/flannel:v0.23.0
    Port:       <none>
    Host Port:  <none>
    Command:
      cp
    Args:
      -f
      /etc/kube-flannel/cni-conf.json
      /etc/cni/net.d/10-flannel.conflist
    Environment:  <none>
    Mounts:
      /etc/cni/net.d from cni (rw)
      /etc/kube-flannel/ from flannel-cfg (rw)
  Containers:
   kube-flannel:
    Image:      docker.io/flannel/flannel:v0.23.0
    Port:       <none>
    Host Port:  <none>
    Command:
      /opt/bin/flanneld
    Args:
      --ip-masq
      --kube-subnet-mgr
      --iface=eth0
    Requests:
      cpu:     100m
      memory:  50Mi
    Environment:
      POD_NAME:            (v1:metadata.name)
      POD_NAMESPACE:       (v1:metadata.namespace)
      EVENT_QUEUE_DEPTH:  5000
    Mounts:
      /etc/kube-flannel/ from flannel-cfg (rw)
      /run/flannel from run (rw)
      /run/xtables.lock from xtables-lock (rw)
  Volumes:
   run:
    Type:          HostPath (bare host directory volume)
    Path:          /run/flannel
    HostPathType:  
   cni-plugin:
    Type:          HostPath (bare host directory volume)
    Path:          /opt/cni/bin
    HostPathType:  
   cni:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/cni/net.d
    HostPathType:  
   flannel-cfg:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-flannel-cfg
    Optional:  false
   xtables-lock:
    Type:               HostPath (bare host directory volume)
    Path:               /run/xtables.lock
    HostPathType:       FileOrCreate
  Priority Class Name:  system-node-critical
  Node-Selectors:       <none>
  Tolerations:          :NoSchedule op=Exists
Events:
  Type    Reason            Age   From                  Message
  ----    ------            ----  ----                  -------
  Normal  SuccessfulCreate  10m   daemonset-controller  Created pod: kube-flannel-ds-ltvml

```



- Notice above that the Image this DameonSet uses is `docker.io/flannel/flannel:v0.23.0`

- Lets now create a new DamonSet 

```sh
➜  cat ds.yaml 
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
  labels:
    name: fluentd-elasticsearch
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: registry.k8s.io/fluentd-elasticsearch:1.20
```


```sh
➜  kubectl apply -f ds.yaml 
daemonset.apps/elasticsearch created
```


### Static Pods


```sh
➜  kubectl get pods -A
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-fn922                  1/1     Running   0          3m36s
kube-flannel   kube-flannel-ds-zxq6s                  1/1     Running   0          3m15s
kube-system    coredns-7484cd47db-qs9jp               1/1     Running   0          3m36s
kube-system    coredns-7484cd47db-th646               1/1     Running   0          3m36s
kube-system    etcd-controlplane                      1/1     Running   0          3m42s
kube-system    kube-apiserver-controlplane            1/1     Running   0          3m42s
kube-system    kube-controller-manager-controlplane   1/1     Running   0          3m42s
kube-system    kube-proxy-lwhlx                       1/1     Running   0          3m15s
kube-system    kube-proxy-m6d9p                       1/1     Running   0          3m36s
kube-system    kube-scheduler-controlplane            1/1     Running   0          3m42s
```


Those pods with -nodeName are static pods. Coredns and kube-proxy is not a static pod.


```sh
➜  ps -aux | grep kubelet
bad data in /proc/uptime
root        3470  0.0  0.4 1457364 268696 ?      Ssl  05:48   0:19 kube-apiserver --advertise-address=192.168.242.137 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=172.20.0.0/16 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        3934  0.0  0.1 2931756 97332 ?       Ssl  05:49   0:10 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.10
```


Check for  `--config=/var/lib/kubelet/config.yaml`


```sh
➜  grep staticPodPath: /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```


As we see the path is `/etc/kubernetes/manifests`


```sh
controlplane /etc/kubernetes/manifests ➜  ls -lh
total 16K
-rw------- 1 root root 2.6K Jun 22 05:48 etcd.yaml
-rw------- 1 root root 3.9K Jun 22 05:48 kube-apiserver.yaml
-rw------- 1 root root 3.4K Jun 22 05:48 kube-controller-manager.yaml
-rw------- 1 root root 1.7K Jun 22 05:48 kube-scheduler.yaml
```


These are the files within the `/etc/kubernetes/manifests`


#### Create a new static pod called static-busybox

```sh
controlplane /etc/kubernetes/manifests ➜  kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml
```


```sh
controlplane /etc/kubernetes/manifests ➜  cat static-busybox.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox
    name: static-busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```


- then notice that the Static pod is now running automatically


```sh
controlplane /etc/kubernetes/manifests ➜  kubectl get pods -n default
NAME                          READY   STATUS    RESTARTS   AGE
static-busybox-controlplane   1/1     Running   0          104s
```


- Lets now change the image running in the static pod
`
```sh
controlplane /etc/kubernetes/manifests ➜  kubectl describe po static-busybox-controlplane -n default | grep -i image:
    Image:         busybox:1.28.4

controlplane /etc/kubernetes/manifests ✖ kubectl get pods -n default 
NAME                          READY   STATUS    RESTARTS   AGE
static-busybox-controlplane   1/1     Running   0          15s
```


## Priority Classes - `New`



Sometimes the kubernetes controlplane run as a pod in kubernets clusters. The kube controlplane components need to have priority to run in the nodes. Here are sample example of Priority:


- Kubernetes components
- Databases
- Critical Applications
- Jobs


High priority run first with guaranteed resource. We can set priority between -2.147.438.648 to 1.000.000.000 being 0 no priority and the default.

There are a separated system kubernetes components that can go beyond 1.000.000.000 and in this case the System workload can go to 2.000.000.000.

```sh
$ kubectl get priorityclass
```


```sh
root@controlplane ~ ➜  kubectl get priorityclass
NAME                      VALUE        GLOBAL-DEFAULT   AGE   PREEMPTIONPOLICY
system-cluster-critical   2000000000   false            16m   PreemptLowerPriority
system-node-critical      2000001000   false            16m   PreemptLowerPriority
```



- Lets create a priority class named high-priority with value of 100.000 

```sh
➜  kubectl create priorityclass high-priority --value=100000 
priorityclass.scheduling.k8s.io/high-priority created
```


```sh
~ ➜  kubectl get pc
NAME                      VALUE        GLOBAL-DEFAULT   AGE   PREEMPTIONPOLICY
high-priority             100000       false            47s   PreemptLowerPriority
system-cluster-critical   2000000000   false            19m   PreemptLowerPriority
system-node-critical      2000001000   false            19m   PreemptLowerPriority
```



- Lets now create another priority class called low-priority with value of 1000

```sh
➜  kubectl create pc low-priority --value=1000 
priorityclass.scheduling.k8s.io/low-priority created
```


```sh
➜  kubectl get pc
NAME                      VALUE        GLOBAL-DEFAULT   AGE     PREEMPTIONPOLICY
high-priority             100000       false            2m18s   PreemptLowerPriority
low-priority              1000         false            41s     PreemptLowerPriority
system-cluster-critical   2000000000   false            20m     PreemptLowerPriority
system-node-critical      2000001000   false            20m     PreemptLowerPriority
```


#### Creating a pod with low-priority


```sh
root@controlplane ~ ➜  kubectl run low-prio-pod --image=nginx --dry-run=client -o yaml > low.yaml

root@controlplane ~ ➜  cat low.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: low-prio-pod
  name: low-prio-pod
spec:
  priorityClassName: low-priority
  containers:
  - image: nginx
    name: low-prio-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@controlplane ~ ➜  kubectl apply -f low.yaml 
pod/low-prio-pod created

root@controlplane ~ ➜  kubectl get pods 
NAME           READY   STATUS    RESTARTS   AGE
low-prio-pod   1/1     Running   0          15s
```


#### Creating a pod with high-priority


```sh
root@controlplane ~ ➜  kubectl run high-prio-pod --image=nginx --dry-run=client -o yaml > high.yaml

root@controlplane ~ ➜  vi high.yaml 

root@controlplane ~ ➜  cat high.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: high-prio-pod
  name: high-prio-pod
spec:
  priorityClassName: high-priority
  containers:
  - image: nginx
    name: high-prio-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@controlplane ~ ➜  kubectl apply -f high.yaml 
pod/high-prio-pod created

root@controlplane ~ ➜  kubectl get pods 
NAME            READY   STATUS    RESTARTS   AGE
high-prio-pod   1/1     Running   0          7s
low-prio-pod    1/1     Running   0          4m52s
```


```sh
root@controlplane ~ ➜  kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
NAME            PRIORITY
high-prio-pod   high-priority
low-prio-pod    low-priority
```


#### Pod with critical-app starving for resources


```sh
➜  kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
critical-app    0/1     Pending   0          24s
high-prio-pod   1/1     Running   0          5m29s
low-app         1/1     Running   0          24s
low-prio-pod    1/1     Running   0          10m
```


- Lets describe and see the events for critical-app


```sh
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  102s  default-scheduler  0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
```


- Lets fix adding the pod critical-app to class high-priority


```sh
➜  kubectl get po critical-app -o yaml > critical.yaml

➜  vi critical.yaml 

# lets add this line
  priorityClassName: high-priority

➜  kubectl apply -f critical.yaml 
pod/critical-app created
```



```sh
➜  kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
critical-app    1/1     Running   0          108s
high-prio-pod   1/1     Running   0          13m
low-prio-pod    1/1     Running   0          18m
```


### Scheduler


```sh
➜  kubectl get pods -n kube-system | grep scheduler
kube-scheduler-controlplane            1/1     Running   0          6m4s
```


- Lets check the image

```sh
➜  kubectl describe po kube-scheduler-controlplane -n kube-system | grep -i image:
    Image:         registry.k8s.io/kube-scheduler:v1.32.0
```


- Lets check all servicesAccount in kube-system

```sh
➜  kubectl get serviceaccount -n kube-system
NAME                                          SECRETS   AGE
attachdetach-controller                       0         9m5s
bootstrap-signer                              0         9m
certificate-controller                        0         9m5s
clusterrole-aggregation-controller            0         9m5s
coredns                                       0         9m4s
cronjob-controller                            0         9m1s
daemon-set-controller                         0         9m4s
default                                       0         8m59s
deployment-controller                         0         9m5s
disruption-controller                         0         9m1s
endpoint-controller                           0         9m3s
endpointslice-controller                      0         9m3s
endpointslicemirroring-controller             0         9m4s
ephemeral-volume-controller                   0         9m3s
expand-controller                             0         9m4s
generic-garbage-collector                     0         9m4s
horizontal-pod-autoscaler                     0         9m3s
job-controller                                0         9m5s
kube-proxy                                    0         9m4s
legacy-service-account-token-cleaner          0         9m2s
my-scheduler                                  0         80s
namespace-controller                          0         9m2s
node-controller                               0         9m1s
persistent-volume-binder                      0         9m3s
pod-garbage-collector                         0         9m5s
pv-protection-controller                      0         9m
pvc-protection-controller                     0         9m3s
replicaset-controller                         0         9m5s
replication-controller                        0         9m4s
resourcequota-controller                      0         9m1s
root-ca-cert-publisher                        0         9m4s
service-account-controller                    0         9m2s
statefulset-controller                        0         9m1s
token-cleaner                                 0         9m2s
ttl-after-finished-controller                 0         9m
ttl-controller                                0         9m5s
validatingadmissionpolicy-status-controller   0         9m2s
```


- Lets now list 

```sh
➜  kubectl get clusterrolebinding 
NAME                                                            ROLE                                                                               AGE
cluster-admin                                                   ClusterRole/cluster-admin                                                          11m
flannel                                                         ClusterRole/flannel                                                                11m
kubeadm:cluster-admins                                          ClusterRole/cluster-admin                                                          11m
kubeadm:get-nodes                                               ClusterRole/kubeadm:get-nodes                                                      11m
kubeadm:kubelet-bootstrap                                       ClusterRole/system:node-bootstrapper                                               11m
kubeadm:node-autoapprove-bootstrap                              ClusterRole/system:certificates.k8s.io:certificatesigningrequests:nodeclient       11m
kubeadm:node-autoapprove-certificate-rotation                   ClusterRole/system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   11m
kubeadm:node-proxier                                            ClusterRole/system:node-proxier                                                    11m
my-scheduler-as-kube-scheduler                                  ClusterRole/system:kube-scheduler                                                  3m29s
my-scheduler-as-volume-scheduler                                ClusterRole/system:volume-scheduler                                                3m29s
system:basic-user                                               ClusterRole/system:basic-user                                                      11m
system:controller:attachdetach-controller                       ClusterRole/system:controller:attachdetach-controller                              11m
system:controller:certificate-controller                        ClusterRole/system:controller:certificate-controller                               11m
system:controller:clusterrole-aggregation-controller            ClusterRole/system:controller:clusterrole-aggregation-controller                   11m
system:controller:cronjob-controller                            ClusterRole/system:controller:cronjob-controller                                   11m
system:controller:daemon-set-controller                         ClusterRole/system:controller:daemon-set-controller                                11m
system:controller:deployment-controller                         ClusterRole/system:controller:deployment-controller                                11m
system:controller:disruption-controller                         ClusterRole/system:controller:disruption-controller                                11m
system:controller:endpoint-controller                           ClusterRole/system:controller:endpoint-controller                                  11m
system:controller:endpointslice-controller                      ClusterRole/system:controller:endpointslice-controller                             11m
system:controller:endpointslicemirroring-controller             ClusterRole/system:controller:endpointslicemirroring-controller                    11m
system:controller:ephemeral-volume-controller                   ClusterRole/system:controller:ephemeral-volume-controller                          11m
system:controller:expand-controller                             ClusterRole/system:controller:expand-controller                                    11m
system:controller:generic-garbage-collector                     ClusterRole/system:controller:generic-garbage-collector                            11m
system:controller:horizontal-pod-autoscaler                     ClusterRole/system:controller:horizontal-pod-autoscaler                            11m
system:controller:job-controller                                ClusterRole/system:controller:job-controller                                       11m
system:controller:legacy-service-account-token-cleaner          ClusterRole/system:controller:legacy-service-account-token-cleaner                 11m
system:controller:namespace-controller                          ClusterRole/system:controller:namespace-controller                                 11m
system:controller:node-controller                               ClusterRole/system:controller:node-controller                                      11m
system:controller:persistent-volume-binder                      ClusterRole/system:controller:persistent-volume-binder                             11m
system:controller:pod-garbage-collector                         ClusterRole/system:controller:pod-garbage-collector                                11m
system:controller:pv-protection-controller                      ClusterRole/system:controller:pv-protection-controller                             11m
system:controller:pvc-protection-controller                     ClusterRole/system:controller:pvc-protection-controller                            11m
system:controller:replicaset-controller                         ClusterRole/system:controller:replicaset-controller                                11m
system:controller:replication-controller                        ClusterRole/system:controller:replication-controller                               11m
system:controller:resourcequota-controller                      ClusterRole/system:controller:resourcequota-controller                             11m
system:controller:root-ca-cert-publisher                        ClusterRole/system:controller:root-ca-cert-publisher                               11m
system:controller:route-controller                              ClusterRole/system:controller:route-controller                                     11m
system:controller:service-account-controller                    ClusterRole/system:controller:service-account-controller                           11m
system:controller:service-controller                            ClusterRole/system:controller:service-controller                                   11m
system:controller:statefulset-controller                        ClusterRole/system:controller:statefulset-controller                               11m
system:controller:ttl-after-finished-controller                 ClusterRole/system:controller:ttl-after-finished-controller                        11m
system:controller:ttl-controller                                ClusterRole/system:controller:ttl-controller                                       11m
system:controller:validatingadmissionpolicy-status-controller   ClusterRole/system:controller:validatingadmissionpolicy-status-controller          11m
system:coredns                                                  ClusterRole/system:coredns                                                         11m
system:discovery                                                ClusterRole/system:discovery                                                       11m
system:kube-controller-manager                                  ClusterRole/system:kube-controller-manager                                         11m
system:kube-dns                                                 ClusterRole/system:kube-dns                                                        11m
system:kube-scheduler                                           ClusterRole/system:kube-scheduler                                                  11m
system:monitoring                                               ClusterRole/system:monitoring                                                      11m
system:node                                                     ClusterRole/system:node                                                            11m
system:node-proxier                                             ClusterRole/system:node-proxier                                                    11m
system:public-info-viewer                                       ClusterRole/system:public-info-viewer                                              11m
system:service-account-issuer-discovery                         ClusterRole/system:service-account-issuer-discovery                                11m
system:volume-scheduler                                         ClusterRole/system:volume-scheduler                                                11m
```



## Monitoring and Logging


Lets deploy a monitoring in our cluster

```sh
➜  kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```



Lets check the top node command

```sh
➜  kubectl top node
NAME           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
controlplane   247m         1%       870Mi           1%          
node01         41m          0%       158Mi           0%   
```

- As we see controlplane is the node consuming most CPU and also Memory

Lets now check the top pods command

```sh
➜  kubectl top pods
NAME       CPU(cores)   MEMORY(bytes)   
elephant   15m          30Mi            
lion       1m           16Mi            
rabbit     110m         250Mi       
```


- As we see rabbit pod is the most consuming Memory and CPU, and the lion the pod consuming less cpu and memory.


### Checking logs

```sh
➜  kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          105s
webapp-2   2/2     Running   0          10s

controlplane ~ ➜  kubectl logs webapp-2 
```


We can also use logs with -f to display like the tail -f 

```sh
kubectl logs -f webapp-2
```


Another cool option to use with logs is the parameter or flag --previous to access logs from the last attempt to restart the pod

```sh
kubectl logs webapp-2 --previous 
```



## Rolling updates


```sh
➜  kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-6765b99794-2lxgg   1/1     Running   0          11s
frontend-6765b99794-j7bgm   1/1     Running   0          11s
frontend-6765b99794-l2qw2   1/1     Running   0          11s
frontend-6765b99794-l69fr   1/1     Running   0          11s
```


Interesting looping

```sh
 ➜  for i in {1..30} ; do echo "Hello World" ; sleep 1 ; done
```


Another interesting looping to test connectivity to the pod

```sh
➜  cat curl-test.sh 
for i in {1..35}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http://webapp-service.default.svc.cluster.local:8080/info 2>&1` && echo "$test OK" || echo "Failed"';
   echo ""
done
```



Check the deployments and deployment type

```sh
➜  kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           14m

➜  kubectl describe deployment frontend | grep -i strategy
StrategyType:           RollingUpdate
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```

During a deployment upgrade here we would have Pods restarted one by one 

### Strategy recreate

```sh
➜  kubectl describe deploy frontend 
Name:               frontend
Namespace:          default
CreationTimestamp:  Fri, 27 Jun 2025 13:22:32 +0000
Labels:             <none>
Annotations:        deployment.kubernetes.io/revision: 3
Selector:           name=webapp
Replicas:           4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:       Recreate
MinReadySeconds:    20
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:         kodekloud/webapp-color:v3
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
OldReplicaSets:  frontend-6765b99794 (0/0 replicas created), frontend-854b57fbbf (0/0 replicas created)
NewReplicaSet:   frontend-7c594bbd6d (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  25m    deployment-controller  Scaled up replica set frontend-6765b99794 from 0 to 4
  Normal  ScalingReplicaSet  7m37s  deployment-controller  Scaled up replica set frontend-854b57fbbf from 0 to 1
  Normal  ScalingReplicaSet  7m37s  deployment-controller  Scaled down replica set frontend-6765b99794 from 4 to 3
  Normal  ScalingReplicaSet  7m37s  deployment-controller  Scaled up replica set frontend-854b57fbbf from 1 to 2
  Normal  ScalingReplicaSet  7m15s  deployment-controller  Scaled down replica set frontend-6765b99794 from 3 to 1
  Normal  ScalingReplicaSet  7m15s  deployment-controller  Scaled up replica set frontend-854b57fbbf from 2 to 4
  Normal  ScalingReplicaSet  6m50s  deployment-controller  Scaled down replica set frontend-6765b99794 from 1 to 0
  Normal  ScalingReplicaSet  2m     deployment-controller  Scaled down replica set frontend-854b57fbbf from 4 to 0
  Normal  ScalingReplicaSet  88s    deployment-controller  Scaled up replica set frontend-7c594bbd6d from 0 to 4
```



Notice the last 2 events that it brought down all 4 pods and up all 4 pods at once
In the other hand the rolling update (first events) it bring down 1 by one and make sure to bring up 1 by one.



## Commands and arguments


```sh
cat ubuntu-sleeper-2.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep","5000"]
```


```sh
➜  kubectl get pods ubuntu-sleeper-2 
NAME               READY   STATUS    RESTARTS   AGE
ubuntu-sleeper-2   1/1     Running   0          38s
```



Lets now create a pod with command set in different way

```sh
➜  cat ubuntu-sleeper-3.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-3 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "1200"

controlplane ~ ➜  kubectl get pods ubuntu-sleeper-3 
NAME               READY   STATUS    RESTARTS   AGE
ubuntu-sleeper-3   1/1     Running   0          46s
```


```yaml
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```



## Secrets


```sh
➜  kubectl get secrets
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      25s
```


Lets describe the secrets

```sh
➜  kubectl describe secrets dashboard-token
Name:         dashboard-token
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: 7f047342-d25c-4bbf-8ff5-c75739f434c8

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlMyWmNmdHU1WVdWbVA1Y094RVpma01wUUtTR1h4Rm5lWDN3Y3FDWTZWVW8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI3ZjA0NzM0Mi1kMjVjLTRiYmYtOGZmNS1jNzU3MzlmNDM0YzgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.WfoxTi2_Bw2-EQywPl1Wc7dLb3u7WdNFxF1xYgWQYNSZ26cv9WjK-OjAInD5wIeruVUDabNgK6aSYSYRywKCWhM9D9Gq1f8DQ5k0ai_5m6DsWClD4OVk4wqFQR7JCCxDiqWrTdLH08yID55CAVMKSGlRZyEhOg1TYG7jbtI0DD-fsYu4O756OxJSb14YfMnsIN9RX8NlIuDjhYqiCzukqkr-1um15MxJInbjvLU-o3a7FQac0OHfewVDaUdJJwAquJqTGL_8yXhG7XhoQ755W9kjvc6aHZm1kIOu3Ts38mdp7senxoEiUaUs122G0pcpntiLRsrysp0kvgnDfWAoCw
ca.crt:     570 bytes
namespace:  7 bytes

```


Notice that above secrets is of type `kubernetes.io/service-account-token`


### Application deployed


```sh
➜  kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
mysql        1/1     Running   0          33s
webapp-pod   1/1     Running   0          33s
```


```sh
➜  kubectl get svc
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP          9m30s
sql01            ClusterIP   10.43.142.153   <none>        3306/TCP         74s
webapp-service   NodePort    10.43.118.10    <none>        8080:30080/TCP   74s
```


Error while accessing application since we have no variables/secrets deployed yet

```sh
### Failed connecting to the MySQL database.

## Environment Variables: DB_Host=Not Set; DB_Database=Not Set; DB_User=Not Set; DB_Password=Not Set; 2003: Can't connect to MySQL server on 'localhost:3306' (111 Connection refused)

From webapp-pod!
```


Create a new secret named `db-secret` with the data given below.

DB_Host = sql01

DB_User = root

DB_Password = password123

```sh
➜  echo -n  "password123" | base64
cGFzc3dvcmQxMjM=
```


```sh
➜  echo -n  "root" | base64
cm9vdA==
```

```sh
➜  kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
secret/db-secret created
```

Lets add envFrom to our pod spec:

```sh
  containers:
  - envFrom:
    - secretRef:
        name: db-secret
```



```sh
➜  kubectl replace --force -f /tmp/kubectl-edit-1613571795.yaml
pod "webapp-pod" deleted
pod/webapp-pod replaced
```

```sh
➜  kubectl get pods webapp-pod 
NAME         READY   STATUS    RESTARTS   AGE
webapp-pod   1/1     Running   0          75s
```

Now application is working

```sh
### Successfully connected to the MySQL database.

## Environment Variables: DB_Host=sql01; DB_Database=Not Set; DB_User=root; DB_Password=password123;

From webapp-pod!
```





## Multi container pods


```sh
 ➜  kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
app         0/1     ContainerCreating   0          29s
fluent-ui   1/1     Running             0          29s
red         0/3     ContainerCreating   0          15s
```


Notice that we have the pod red with 3 containers but no one is started/running yet.

```sh
➜  kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
app         1/1     Running   0          2m56s
blue        2/2     Running   0          84s
fluent-ui   1/1     Running   0          2m56s
red         3/3     Running   0          2m42s
```


### Creating a pod with 02 containers


Lets create a template with a single container and edit it:

```sh
➜  kubectl run yellow --image=busybox --dry-run=client -o yaml > pod.yaml
```

The file edited should be:

```
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: yellow
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon 
    command:
      - "sleep"
      - "1000"
  - image: redis
    name: gold
```



```sh
➜  kubectl apply -f pod.yaml 
pod/yellow created

 ➜  kubectl get pod yellow
NAME     READY   STATUS    RESTARTS   AGE
yellow   2/2     Running   0          9s
```


Lets get all containers images in the pod yellow:

```sh
 ➜  kubectl get pods yellow -o jsonpath="{.spec.containers[*].image}"
busybox redis
```


lets describe this pod with 02 containers

```sh
 ➜  kubectl describe po yellow 
Name:             yellow
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.126.139
Start Time:       Fri, 25 Jul 2025 12:46:16 +0000
Labels:           run=yellow
Annotations:      <none>
Status:           Running
IP:               172.17.0.13
IPs:
  IP:  172.17.0.13
Containers:
  lemon:
    Container ID:  containerd://4c09546d1d07bf490e76794d55933fae9d0bf58a2c58f6342438cd753fff0b55
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:f85340bf132ae937d2c2a763b8335c9bab35d6e8293f70f606b9c6178d84f42b
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      1000
    State:          Running
      Started:      Fri, 25 Jul 2025 12:46:18 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-t7bvg (ro)
  gold:
    Container ID:   containerd://8c954a18afa267b1500d9c73a2a7c7b2eacff5d83ba2c300b5f3917adcb15c24
    Image:          redis
    Image ID:       docker.io/library/redis@sha256:f957ce918b51f3ac10414244bedd0043c47db44a819f98b9902af1bd9d0afcea
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 25 Jul 2025 12:46:19 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-t7bvg (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-t7bvg:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  25s   default-scheduler  Successfully assigned default/yellow to controlplane
  Normal  Pulling    24s   kubelet            Pulling image "busybox"
  Normal  Pulled     24s   kubelet            Successfully pulled image "busybox" in 166ms (166ms including waiting). Image size: 2156518 bytes.
  Normal  Created    24s   kubelet            Created container: lemon
  Normal  Started    23s   kubelet            Started container lemon
  Normal  Pulling    23s   kubelet            Pulling image "redis"
  Normal  Pulled     23s   kubelet            Successfully pulled image "redis" in 155ms (155ms including waiting). Image size: 49527330 bytes.
  Normal  Created    23s   kubelet            Created container: gold
  Normal  Started    22s   kubelet            Started container gold
```


## kubernetes error code for troubleshooting

https://github.com/anveshmuppeda/kubernetes/tree/main/docs/012-troubleshoot

https://medium.com/@muppedaanvesh/a-hands-on-guide-to-kubernetes-exit-codes-simulate-and-fix-%EF%B8%8F-f2ad57d3cdca



## initContainer pods


In a **multi-container Pod**, each container is expected to run a process that stays alive for the **entire lifecycle of the Pod**.

For example, in a Pod with a **web application** and a **logging agent**, both containers are expected to remain active throughout the Pod’s lifecycle. The process in the logging agent container must stay alive as long as the web application is running. If any main container fails and the Pod's `restartPolicy` is `Always` or `OnFailure`, the **entire Pod is restarted**.

Using initContainer 


```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  initContainers:
    - name: init-myservice
      image: busybox:1.31
      command: ["sh", "-c", "until nslookup myservice; do echo waiting for myservice; sleep 2; done;"]
    - name: init-mydb
      image: busybox:1.31
      command: ["sh", "-c", "until nslookup mydb; do echo waiting for mydb; sleep 2; done;"]
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
```


Example of native sidecar

```
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  initContainers:
    - name: sidecar-logger
      image: busybox:1.31
      restartPolicy: Always
      command: ["sh", "-c", "while true; do echo Sidecar running; sleep 10; done"]
  containers:
    - name: main-app
      image: busybox:1.31
      command: ["sh", "-c", "echo Main app starting; sleep 60"]
```




```sh
controlplane ~ ➜  kubectl describe po blue
Name:             blue
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.219.190
Start Time:       Fri, 17 Oct 2025 12:26:30 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.11
IPs:
  IP:  10.22.0.11
Init Containers:
  init-myservice:
    Container ID:  containerd://034ee7e8d1362c3370cee21d1acee54856f3866c00edeba8d35b59a5798ea18b
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:2f590fc602ce325cbff2ccfc39499014d039546dc400ef8bbf5c6ffb860632e7
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 5
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 17 Oct 2025 12:26:31 +0000
      Finished:     Fri, 17 Oct 2025 12:26:36 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j24gg (ro)
Containers:
...
```


Notice the state is "Terminated" and the Reason is Completed


See now one pod with 02 initContainers:

```sh
✖ kubectl describe po purple
Name:             purple
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.219.190
Start Time:       Fri, 17 Oct 2025 12:31:41 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.22.0.12
IPs:
  IP:  10.22.0.12
Init Containers:
  warm-up-1:
    Container ID:  containerd://d03521d57232fd619ad4b9c2acf3b7bee78958e044bcfb51e59818fc6b918060
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 600
    State:          Running
      Started:      Fri, 17 Oct 2025 12:31:42 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mnxt8 (ro)
  warm-up-2:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 1200
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mnxt8 (ro)
Containers:
  purple-container:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mnxt8 (ro)
Conditions:
...
```



Notice that in the first initContainer called warm-up-1 it sleep for 600s and the second initContainer called warm-up-2 sleeps for 1200, which means it will wait 30 minutes before starting the application with pod purple-container.

See another sample:

```sh
spec:
  initContainers:
  - name: red-initcontainer
    image: busybox
    command: ["sleep","20"]
```


Replacing your pod with updated yaml file:

```sh
$ kubectl replace --force -f /tmp/kubectl-edit-2951710210.yaml
```



```sh
controlplane ~ ➜  kubectl get pods orange
NAME     READY   STATUS       RESTARTS      AGE
orange   0/1     Init:Error   2 (25s ago)   28s
```


```sh
controlplane ~ ➜  kubectl describe po orange
Name:             orange
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.168.219.190
Start Time:       Fri, 17 Oct 2025 12:42:09 +0000
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               10.22.0.14
IPs:
  IP:  10.22.0.14
Init Containers:
  init-myservice:
    Container ID:  containerd://68e22d9a1186792825b3556ca920166ff097711b6249baad4b16a2336216aa55
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:2f590fc602ce325cbff2ccfc39499014d039546dc400ef8bbf5c6ffb860632e7
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleeeep 2;
    State:          Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Fri, 17 Oct 2025 12:42:55 +0000
      Finished:     Fri, 17 Oct 2025 12:42:55 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Fri, 17 Oct 2025 12:42:25 +0000
      Finished:     Fri, 17 Oct 2025 12:42:25 +0000
    Ready:          False
    Restart Count:  3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-d9lq4 (ro)
Containers:
  orange-container:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-d9lq4 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 False 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-d9lq4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  53s               default-scheduler  Successfully assigned default/orange to controlplane
  Normal   Pulled     52s               kubelet            Successfully pulled image "busybox" in 122ms (122ms including waiting). Image size: 2223686 bytes.
  Normal   Pulled     50s               kubelet            Successfully pulled image "busybox" in 127ms (127ms including waiting). Image size: 2223686 bytes.
  Normal   Pulled     37s               kubelet            Successfully pulled image "busybox" in 145ms (145ms including waiting). Image size: 2223686 bytes.
  Normal   Pulling    8s (x4 over 52s)  kubelet            Pulling image "busybox"
  Normal   Pulled     8s                kubelet            Successfully pulled image "busybox" in 126ms (126ms including waiting). Image size: 2223686 bytes.
  Normal   Created    7s (x4 over 52s)  kubelet            Created container: init-myservice
  Normal   Started    7s (x4 over 52s)  kubelet            Started container init-myservice
  Warning  BackOff    6s (x4 over 49s)  kubelet            Back-off restarting failed container init-myservice in pod orange_default(543b9fab-2404-40e5-838e-aa66ecabaf3b)
```


Notice the events:


```
  Normal   Created    7s (x4 over 52s)  kubelet            Created container: init-myservice
  Normal   Started    7s (x4 over 52s)  kubelet            Started container init-myservice
  Warning  BackOff    6s (x4 over 49s)  kubelet            Back-off restarting failed container init-myservice in pod orange_default(543b9fab-2404-40e5-838e-aa66ecabaf3b)
```


something is causing back-off on init-myservice 
Notice Exit Code:    127 which means "Command not found" , see https://www.anantacloud.com/post/kubernetes-exit-codes-decoded-common-problems-and-solutions

Lets check the logs

```
controlplane ~ ➜  kubectl logs orange 
Defaulted container "orange-container" out of: orange-container, init-myservice (init)
Error from server (BadRequest): container "orange-container" in pod "orange" is waiting to start: PodInitializing
```

Logs are showed for the container "orange-container" which is waiting to start. Lets check now logs for the initContainer:

```sh
controlplane ~ ✖ kubectl logs orange -c init-myservice
sh: sleeeep: not found
```

As we see we got command not found, lets fix that

```
  - command:
    - sh
    - -c
    - sleep 2;
```


```sh
kubectl replace --force -f /tmp/kubectl-edit-2045163821.yaml
pod "orange" deleted
pod/orange replaced
```




## Self-healing applications - Autoscaling


- Vertical autoscaling - increase capacity of server like memory and cpu.
- Horizontal autoscaling - increase number of nodes and pods adding new servers to the cluster.


### Manual Scaling of a Kubernetes Deployment

Lets create a deployment with deployment.yaml using flask application.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask
        image: rakshithraka/flask-web-app
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: flask-web-app-service
spec:
  type: ClusterIP
  selector:
    app: flask-app
  ports:
   - port: 80
     targetPort: 80
```





```sh
controlplane ~ ➜  kubectl apply -f deployment.yml 
deployment.apps/flask-web-app created
service/flask-web-app-service created
```


Lets see our resources.

```sh
controlplane ~ ➜  kubectl get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
flask-web-app   2/2     2            2           23s

controlplane ~ ➜  kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
flask-web-app-584dd886c8-lz9c4   1/1     Running   0          30s
flask-web-app-584dd886c8-tnc57   1/1     Running   0          30s
```




Lets scale flask-web-app to 3 replicas.

```sh
controlplane ~ ➜  kubectl scale deploy flask-web-app --replicas=3
deployment.apps/flask-web-app scaled

controlplane ~ ➜  kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
flask-web-app-584dd886c8-lz9c4   1/1     Running   0          3m23s
flask-web-app-584dd886c8-nw9h4   1/1     Running   0          6s
flask-web-app-584dd886c8-tnc57   1/1     Running   0          3m23s
```



Lets see now all deployments, svc and endpoint.

```sh
controlplane ~ ➜  kubectl get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
flask-web-app   3/3     3            3           7m25s

controlplane ~ ➜  kubectl get svc
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
flask-web-app-service   ClusterIP   172.20.242.253   <none>        80/TCP    8m32s
kubernetes              ClusterIP   172.20.0.1       <none>        443/TCP   64m


controlplane ~ ➜  kubectl get ep
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME                    ENDPOINTS                                   AGE
flask-web-app-service   172.17.1.7:80,172.17.1.8:80,172.17.1.9:80   7m33s
kubernetes              192.168.142.205:6443                        63m

controlplane ~ ➜  kubectl get pods -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
flask-web-app-584dd886c8-lz9c4   1/1     Running   0          7m40s   172.17.1.7   node01   <none>           <none>
flask-web-app-584dd886c8-nw9h4   1/1     Running   0          4m23s   172.17.1.9   node01   <none>           <none>
flask-web-app-584dd886c8-tnc57   1/1     Running   0          7m40s   172.17.1.8   node01   <none>           <none>

```


### Lab HPA - horizontal pod autoscaling 

Lets create a deployment with deployment.yaml with nginx application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 7
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```


```sh
controlplane ~ ➜  kubectl apply -f deployment.yml 
deployment.apps/nginx-deployment created

controlplane ~ ➜  kubectl get deploy 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   7/7     7            7           24s

controlplane ~ ➜  kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-647677fc66-5579t   1/1     Running   0          94s
nginx-deployment-647677fc66-9xlfg   1/1     Running   0          94s
nginx-deployment-647677fc66-crqtq   1/1     Running   0          94s
nginx-deployment-647677fc66-jc9cz   1/1     Running   0          94s
nginx-deployment-647677fc66-m42s4   1/1     Running   0          94s
nginx-deployment-647677fc66-qgfcw   1/1     Running   0          94s
nginx-deployment-647677fc66-zvwfz   1/1     Running   0          94s
```


Lets create a autoscale.yaml

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: nginx-deployment
spec:
  maxReplicas: 3
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 80
        type: Utilization
    type: Resource
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
status:
  currentMetrics: null
  desiredReplicas: 0
  currentReplicas: 0
```


```sh
controlplane ~ ➜  kubectl apply -f autoscale.yml 
horizontalpodautoscaler.autoscaling/nginx-deployment created

controlplane ~ ➜  kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m39s

controlplane ~ ➜  kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-647677fc66-m42s4   1/1     Running   0          5m48s
nginx-deployment-647677fc66-qgfcw   1/1     Running   0          5m48s
nginx-deployment-647677fc66-zvwfz   1/1     Running   0          5m48s
```


Lets check the status of HPA

```sh
controlplane ~ ➜  kubectl get hpa
NAME               REFERENCE                     TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   cpu: <unknown>/80%   1         3         3          2m26s
```


Lets fix the failing HPA failing due the resource field missing in the deployment


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 7
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
         requests:
           cpu: 100m
         limits:
           cpu: 200m
```



```sh
controlplane ~ ➜  kubectl apply -f deployment.yml 
deployment.apps/nginx-deployment configured
```


Lets check now

```sh
controlplane ~ ➜  kubectl get hpa --watch
NAME               REFERENCE                     TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   cpu: <unknown>/80%   1         3         3          6m21s
nginx-deployment   Deployment/nginx-deployment   cpu: 0%/80%          1         3         3          6m31s
```



### In-place pod resizing


In a default kubernetes previously behavior when we change a deployment manifest to add more memory or cpu the pod automatically have to recycle, get deleted and recreated for the new configuration take in place.

The new versions of kubernetes has a feature to resize cpu and memory resources to a container and the pod won't die, this feature is currently in alpha version and has to be enabled before using

Run:
```sh
FEATURE_GATES=InPlacePodVerticalScaling=true
```



Limitations:

- only works for cpu and memory
- pod QOS class can not change
- initContainers and ephemeral containers cannot be resize
- a container memory limit can not be set below its current usage


 ### Install VPA - vertical pod autoscaling 


### **Lab Objective:**

In this lab, you will install and configure the Vertical Pod Autoscaler (VPA) in your Kubernetes cluster. The lab will walk you through the installation of VPA using predefined manifests, cloning the VPA repository for advanced control, and deploying a sample application to see how VPA interacts with it. Additionally, you'll learn how to troubleshoot issues using VPA logs.

By the end of this lab, you should be able to:

- **Install VPA** and its components (Recommender, Updater, Admission Controller) in a Kubernetes cluster
- **Understand the role of each VPA component** and how they contribute to efficient resource management
- **Deploy a sample application** to see how VPA recommends and adjusts resources for it
- **Troubleshoot resource-related issues** in your application using logs generated by the VPA components, particularly the VPA Updater

This hands-on experience will give you the skills to manage pod resources dynamically in a production-grade Kubernetes environment, ensuring that applications run efficiently with the appropriate resource requests.


### Step 1: Install VPA Custom Resource Definitions (CRDs)

These CRDs allow Kubernetes to recognize the custom resources that VPA uses to function properly. To install them, run this command:

```bash
kubectl apply -f /root/vpa-crds.yml
```


### Step 2: Install VPA Role-Based Access Control (RBAC)

RBAC ensures that VPA has the appropriate permissions to operate within your Kubernetes cluster. To install the RBAC settings, run:

```bash
kubectl apply -f /root/vpa-rbac.yml
```

By running these commands, the VPA will be successfully deployed to your cluster, ready to manage and adjust your pod resources dynamically.

```sh
controlplane ~ ➜  kubectl apply -f /root/vpa-crds.yml
customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalercheckpoints.autoscaling.k8s.io created
customresourcedefinition.apiextensions.k8s.io/verticalpodautoscalers.autoscaling.k8s.io created

controlplane ~ ➜  kubectl apply -f /root/vpa-rbac.yml
clusterrole.rbac.authorization.k8s.io/system:metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:vpa-actor created
clusterrole.rbac.authorization.k8s.io/system:vpa-status-actor created
clusterrole.rbac.authorization.k8s.io/system:vpa-checkpoint-actor created
clusterrole.rbac.authorization.k8s.io/system:evictioner created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-actor created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-actor created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-checkpoint-actor created
clusterrole.rbac.authorization.k8s.io/system:vpa-target-reader created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-target-reader-binding created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-evictioner-binding created
serviceaccount/vpa-admission-controller created
serviceaccount/vpa-recommender created
serviceaccount/vpa-updater created
clusterrole.rbac.authorization.k8s.io/system:vpa-admission-controller created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-admission-controller created
clusterrole.rbac.authorization.k8s.io/system:vpa-status-reader created
clusterrolebinding.rbac.authorization.k8s.io/system:vpa-status-reader-binding created
```


### **Clone the VPA Repository and Set Up the Vertical Pod Autoscaler**

You are required to clone the Kubernetes Autoscaler repository into the `/root` directory and set up the Vertical Pod Autoscaler (VPA) by running the provided script.


1. First, navigate to the `/root` directory and clone the repository:
    

```bash
  git clone https://github.com/kubernetes/autoscaler.git
```


2. After cloning, move into the `vertical-pod-autoscaler` directory:
    

```bash
   cd autoscaler/vertical-pod-autoscaler
```

3. **Run the setup script:**
    
    Execute the provided script to deploy the Vertical Pod Autoscaler:
    

```bash
   ./hack/vpa-up.sh
```

By following these steps, the Vertical Pod Autoscaler will be installed and ready to manage pod resources in your Kubernetes cluster.


```sh
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (a8ca316) via 🐹 ➜  kubectl get crds | grep verticalpodauto
verticalpodautoscalercheckpoints.autoscaling.k8s.io   2025-10-18T09:55:43Z
verticalpodautoscalers.autoscaling.k8s.io             2025-10-18T09:55:43Z
```


```sh
controlplane autoscaler/vertical-pod-autoscaler on  HEAD (a8ca316) via 🐹 ➜  kubectl get pods -n kube-system | grep vpa- 
vpa-admission-controller-76f55f79cc-bcxtl   1/1     Running   0          4m20s
vpa-recommender-588485c64b-w9d5p            1/1     Running   0          4m20s
vpa-updater-75d58448cf-kgp9f                1/1     Running   0          4m20s
```



```sh
controlplane ~ ➜  kubectl apply -f flask-app.yml 
deployment.apps/flask-app created
service/flask-app-service created
verticalpodautoscaler.autoscaling.k8s.io/flask-app created
```


```sh
➜  kubectl logs -f vpa-updater-75d58448cf-kgp9f -n kube-system
...
I1018 10:09:49.396629       1 pods_restriction_factory.go:212] "Too few replicas" kind="ReplicaSet" object="default/flask-app-67b666c5fc" livePods=1 requiredPods=2 globalMinReplicas=2
```

```sh
controlplane ~ ➜  kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
flask-app-67b666c5fc-27qwv   1/1     Running   0          7m34s
```


```sh
 ➜  kubectl scale deployment flask-app --replicas=2
deployment.apps/flask-app scaled

➜  kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
flask-app-67b666c5fc-27qwv   1/1     Running   0          8m37s
flask-app-67b666c5fc-68z2r   1/1     Running   0          2s
```

```sh
controlplane ~ ✖ kubectl get deployment flask-app -o wide
NAME        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                          SELECTOR
flask-app   2/2     2            2           10m   flask-app    kodekloud/flask-session-app:1   app=flask-app
```

```sh
controlplane ~ ➜  kubectl get pods -l app=flask-app
NAME                         READY   STATUS    RESTARTS   AGE
flask-app-67b666c5fc-68z2r   1/1     Running   0          2m48s
flask-app-67b666c5fc-sh98k   1/1     Running   0          114s
```

```sh
Events:
  Type    Reason      Age    From         Message
  ----    ------      ----   ----         -------
  Normal  EvictedPod  2m17s  vpa-updater  VPA Updater evicted Pod flask-app-67b666c5fc-27qwv to apply resource recommendation.
```


```sh
controlplane ~ ➜  kubectl get pods -l app=flask-app
NAME                         READY   STATUS    RESTARTS   AGE
flask-app-67b666c5fc-68z2r   1/1     Running   0          3m18s
flask-app-67b666c5fc-sh98k   1/1     Running   0          2m24s
```






```sh
controlplane ~ ✖ kubectl top pod
NAME                           CPU(cores)   MEMORY(bytes)   
flask-app-4-7dcd9549fc-59kjc   23m          19Mi            
flask-app-4-7dcd9549fc-g4h49   26m          19Mi            

controlplane ~ ➜  kubectl top pod
NAME                           CPU(cores)   MEMORY(bytes)   
flask-app-4-7dcd9549fc-59kjc   1m           19Mi            
flask-app-4-7dcd9549fc-g4h49   1m           19Mi      
```


---



Example to mount volume in yaml pod.

```sh
        volumeMounts:       
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap: 
          name: app-config
```




---




## Cluster Maintenance

### OS upgrade


Lets check all nodes and see they are ready

```sh
➜  kubectl get no
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   12m   v1.33.0
node01         Ready    <none>          12m   v1.33.0
```


Lets see that we have only one deployment 

```sh
➜  kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   3/3     3            3           10s
```


Notice that the pods are deployed in 02 different nodes

```sh
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bvqb8   1/1     Running   0          50s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-kqbrp   1/1     Running   0          50s   172.17.1.2   node01         <none>           <none>
blue-69968556cc-tjknm   1/1     Running   0          50s   172.17.1.3   node01         <none>           <none>
```


Lets take out the node01 out for maintenance. For this we drain the node01, when we drain the node we set it unschedule and evict all pods from that node

```sh
✖ kubectl drain node01 --ignore-daemonsets 
node/node01 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-kdzc4, kube-system/kube-proxy-rpwfq
evicting pod default/blue-69968556cc-tjknm
evicting pod default/blue-69968556cc-kqbrp
pod/blue-69968556cc-kqbrp evicted
pod/blue-69968556cc-tjknm evicted
node/node01 drained
```


```sh
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bvqb8   1/1     Running   0          3m20s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-dwhxr   1/1     Running   0          83s     172.17.0.5   controlplane   <none>           <none>
blue-69968556cc-llmpx   1/1     Running   0          83s     172.17.0.6   controlplane   <none>           <none>
```


```sh
➜  kubectl get no
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready                      control-plane   17m   v1.33.0
node01         Ready,SchedulingDisabled   <none>          16m   v1.33.0
```



Once the maintenance is over we can enable node01 back up by uncordon it

```sh
➜  kubectl uncordon node01
node/node01 uncordoned
```


```sh
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bvqb8   1/1     Running   0          5m49s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-dwhxr   1/1     Running   0          3m52s   172.17.0.5   controlplane   <none>           <none>
blue-69968556cc-llmpx   1/1     Running   0          3m52s   172.17.0.6   controlplane   <none>           <none>
```



```sh
➜  kubectl get no
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   19m   v1.33.0
node01         Ready    <none>          19m   v1.33.0
```

```sh
➜  kubectl describe no controlplane | grep -i taints
Taints:             <none>
```



lets try to drain a node01 with one pod within that is not part of any replicaset

```sh
➜  kubectl drain node01 --ignore-daemonsets 
node/node01 cordoned
error: unable to drain node "node01" due to error: cannot delete cannot delete Pods that declare no controller (use --force to override): default/hr-app, continuing command...
There are pending nodes to be drained:
 node01
cannot delete cannot delete Pods that declare no controller (use --force to override): default/hr-app
```


```sh
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bvqb8   1/1     Running   0          9m35s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-dwhxr   1/1     Running   0          7m38s   172.17.0.5   controlplane   <none>           <none>
blue-69968556cc-llmpx   1/1     Running   0          7m38s   172.17.0.6   controlplane   <none>           <none>
hr-app                  1/1     Running   0          80s     172.17.1.4   node01         <none>           <none>
```



```sh
➜  kubectl drain node01 --ignore-daemonsets --force
node/node01 already cordoned
Warning: deleting Pods that declare no controller: default/hr-app; ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-kdzc4, kube-system/kube-proxy-rpwfq
evicting pod default/hr-app
pod/hr-app evicted
node/node01 drained
```


Let now just cordon the node, when we cordon the node the pod running continuing running although no new pods are deployed in node01


```sh
➜  kubectl cordon node01
node/node01 cordoned
```


```sh
➜  kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-bvqb8   1/1     Running   0          12m   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-dwhxr   1/1     Running   0          10m   172.17.0.5   controlplane   <none>           <none>
blue-69968556cc-llmpx   1/1     Running   0          10m   172.17.0.6   controlplane   <none>           <none>
hr-app-79c459cc-g6qbt   1/1     Running   0          95s   172.17.1.5   node01         <none>           <none>
```


```sh
➜  kubectl get no
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready                      control-plane   25m   v1.33.0
node01         Ready,SchedulingDisabled   <none>          25m   v1.33.0
```



## Kubernetes versions


Ideally all kubernetes architecture resource should have the same version that kube-api.
No resources should be a version in front of kube-api

- kubectl control and scheduler should be up 1 level blelow 
- kubelet and proxy could have 2 version below



## Kubeadm upgrade


- kubeadm upgrade plan ## to plan the cluster upgrade
- kubeadm update to new version ## to update the kubeadm client first to new version
- kubeadm upgrade apply ## to upgrade the entire cluster
- upgrade kubelet on all nodes required usually all worker nodes ## to updgrade the kubelet in all worker nodes.


### How to upgrade kube cluster from v1.11 to v1.12


```sh
apt-get upgrade -y kubeadm=1.12.0-00 ## to update the kubeadm itself
kubeadm upgrade apply v1.12.0 ## then we update the cluster to new version v1.12.0. This update your control plane. Next: kubelet
kubectl get nodes ## this show us the current version v1.11.3, since worker nodes were not update yet only contol plane

# On master node
apt-get upgrade -y kubelet=1.12.0-00 ## to update your kubelet on master node, if you have installed previously your kubelet in master
systemctl restart kubelet
kubectl get nodes ## show us master in version v1.12.0 and workers at v1.11.3

# Update worker nodes
kubectl drain node01 # to unschedule and drain / evict all pods from node01
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
kubeadm upgrade node config --kubelet-version v1.12.0
systemctl restart kubelet
kubectl uncordon node01 ## set node01 to schedule pods again and allow it to be in READY state and updated version
kubectl get nodes ## you will see the node01 upgraded version
```


### Demo upgrade kubernetes cluster from v1.32 to v1.33


```sh
➜  kubectl get no
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   19m   v1.32.0
node01         Ready    <none>          18m   v1.32.0
```


```sh
➜  kubectl get po -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-69968556cc-4jr99   1/1     Running   0          2m32s   172.17.0.4   controlplane   <none>           <none>
blue-69968556cc-4qmwx   1/1     Running   0          2m32s   172.17.1.3   node01         <none>           <none>
blue-69968556cc-7hmjc   1/1     Running   0          2m32s   172.17.1.4   node01         <none>           <none>
blue-69968556cc-9kb9c   1/1     Running   0          2m32s   172.17.1.2   node01         <none>           <none>
blue-69968556cc-fv8pw   1/1     Running   0          2m32s   172.17.0.5   controlplane   <none>           <none>
```



```sh
➜  kubectl describe no controlplane | grep -i taints
Taints:             <none>
```



```sh
➜  kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   5/5     5            5           4m18s
```



#### How to know the next kube version


Lets see the current kubeadm version and what is the latest version supported by this kubeadm version:

```sh
✖ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"32", GitVersion:"v1.32.0", GitCommit:"70d3cc986aa8221cd1dfb1121852688902d3bf53", GitTreeState:"clean", BuildDate:"2024-12-11T18:04:20Z", GoVersion:"go1.23.3", Compiler:"gc", Platform:"linux/amd64"}
```



```sh
➜  kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config --config your-config.yaml' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.32.0
[upgrade/versions] kubeadm version: v1.32.0
I1026 13:31:40.310135   19870 version.go:261] remote version is much newer: v1.34.1; falling back to: stable-1.32
[upgrade/versions] Target version: v1.32.9
[upgrade/versions] Latest version in the v1.32 series: v1.32.9

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.32.0   v1.32.9
kubelet     node01         v1.32.0   v1.32.9

Upgrade to the latest version in the v1.32 series:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.32.0    v1.32.9
kube-controller-manager   controlplane   v1.32.0    v1.32.9
kube-scheduler            controlplane   v1.32.0    v1.32.9
kube-proxy                               1.32.0     v1.32.9
CoreDNS                                  v1.10.1    v1.11.3
etcd                      controlplane   3.5.16-0   3.5.16-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.32.9

Note: Before you can perform this upgrade, you have to update kubeadm to v1.32.9.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

```


#### Controlplane upgrade


Lets drain the controlplane first

```sh
➜  kubectl drain controlplane --ignore-daemonsets 
node/controlplane cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-9fvg7, kube-system/kube-proxy-lv2dh
evicting pod kube-system/coredns-7484cd47db-zz4zc
evicting pod default/blue-69968556cc-5pdrv
evicting pod kube-system/coredns-7484cd47db-qjx45
evicting pod default/blue-69968556cc-2rhd6
pod/blue-69968556cc-2rhd6 evicted
pod/blue-69968556cc-5pdrv evicted
pod/coredns-7484cd47db-qjx45 evicted
pod/coredns-7484cd47db-zz4zc evicted
node/controlplane drained
```


```sh
➜  kubectl get no
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready,SchedulingDisabled   control-plane   25m   v1.32.0
node01         Ready                      <none>          24m   v1.32.0
```


Lets follow up the official documentation:
https://v1-33.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


Changing the package repository.
https://v1-33.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/#changing-the-package-repository



1. ```shell
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

```shell
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```



Lets run the above 02 commands:

```sh
➜  echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /
```

```sh
✖ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
File '/etc/apt/keyrings/kubernetes-apt-keyring.gpg' exists. Overwrite? (y/N) y
```




##### Determine which version to upgrade to

```sh
➜  sudo apt update
sudo apt-cache madison kubeadm
Hit:2 https://download.docker.com/linux/ubuntu jammy InRelease                                                                            
Hit:3 http://archive.ubuntu.com/ubuntu jammy InRelease                                                                                    
Hit:4 http://archive.ubuntu.com/ubuntu jammy-updates InRelease                                                        
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  InRelease        
Hit:5 http://archive.ubuntu.com/ubuntu jammy-backports InRelease    
Hit:6 http://security.ubuntu.com/ubuntu jammy-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
86 packages can be upgraded. Run 'apt list --upgradable' to see them.
   kubeadm | 1.33.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
   kubeadm | 1.33.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages


Next version: 1.33.0-1.1  



```shell
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.0-1.1' && \
sudo apt-mark hold kubeadm
```


```sh
➜  sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.0-1.1' && \
sudo apt-mark hold kubeadm
kubeadm was already not on hold.
Hit:2 https://download.docker.com/linux/ubuntu jammy InRelease                                                                            
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  InRelease                                   
Hit:3 http://security.ubuntu.com/ubuntu jammy-security InRelease                                                                   
Hit:4 http://archive.ubuntu.com/ubuntu jammy InRelease                      
Hit:5 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 85 not upgraded.
Need to get 12.7 MB of archives.
After this operation, 3,592 kB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  kubeadm 1.33.0-1.1 [12.7 MB]
Fetched 12.7 MB in 0s (32.9 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 20567 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.33.0-1.1_amd64.deb ...
Unpacking kubeadm (1.33.0-1.1) over (1.32.0-1.1) ...
Setting up kubeadm (1.33.0-1.1) ...
kubeadm set on hold.
```


```sh
➜  kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"33", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.33.0", GitCommit:"60a317eadfcb839692a68eab88b2096f4d708f4f", GitTreeState:"clean", BuildDate:"2025-04-23T13:05:48Z", GoVersion:"go1.24.2", Compiler:"gc", Platform:"linux/amd64"}
```


```sh
➜  sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config --config your-config-file' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.32.0
[upgrade/versions] kubeadm version: v1.33.5
I1026 11:47:11.241910   29241 version.go:261] remote version is much newer: v1.34.1; falling back to: stable-1.33
[upgrade/versions] Target version: v1.33.5
[upgrade/versions] Latest version in the v1.32 series: v1.32.9

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.32.0   v1.32.9
kubelet     node01         v1.32.0   v1.32.9

Upgrade to the latest version in the v1.32 series:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.32.0    v1.32.9
kube-controller-manager   controlplane   v1.32.0    v1.32.9
kube-scheduler            controlplane   v1.32.0    v1.32.9
kube-proxy                               1.32.0     v1.32.9
CoreDNS                                  v1.10.1    v1.12.0
etcd                      controlplane   3.5.16-0   3.5.21-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.32.9

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.32.0   v1.33.5
kubelet     node01         v1.32.0   v1.33.5

Upgrade to the latest stable version:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.32.0    v1.33.5
kube-controller-manager   controlplane   v1.32.0    v1.33.5
kube-scheduler            controlplane   v1.32.0    v1.33.5
kube-proxy                               1.32.0     v1.33.5
CoreDNS                                  v1.10.1    v1.12.0
etcd                      controlplane   3.5.16-0   3.5.21-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.33.5

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```



```sh
✖ sudo kubeadm upgrade plan 1.33.0-1.1
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade/config] Use 'kubeadm init phase upload-config --config your-config-file' to re-upload it.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.32.0
[upgrade/versions] kubeadm version: v1.33.0
[upgrade/versions] Target version: 1.33.0-1.1
[upgrade/versions] Latest version in the v1.32 series: 1.33.0-1.1

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.32.0   1.33.0-1.1
kubelet     node01         v1.32.0   1.33.0-1.1

Upgrade to the latest version in the v1.32 series:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.32.0    1.33.0-1.1
kube-controller-manager   controlplane   v1.32.0    1.33.0-1.1
kube-scheduler            controlplane   v1.32.0    1.33.0-1.1
kube-proxy                               1.32.0     1.33.0-1.1
CoreDNS                                  v1.10.1    v1.12.0
etcd                      controlplane   3.5.16-0   3.5.21-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply 1.33.0-1.1 --allow-experimental-upgrades

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```







```sh
✖ sudo kubeadm upgrade apply v1.33.0
[upgrade] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade] Use 'kubeadm init phase upload-config --config your-config-file' to re-upload it.
[upgrade/preflight] Running preflight checks
        [WARNING SystemVerification]: cgroups v1 support is in maintenance mode, please migrate to cgroups v2
[upgrade] Running cluster health checks
[upgrade/preflight] You have chosen to upgrade the cluster version to "v1.33.0"
[upgrade/versions] Cluster version: v1.32.0
[upgrade/versions] kubeadm version: v1.33.0
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/preflight] Pulling images required for setting up a Kubernetes cluster
[upgrade/preflight] This might take a minute or two, depending on the speed of your internet connection
[upgrade/preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W1026 13:46:51.085745   26566 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[upgrade/control-plane] Upgrading your static Pod-hosted control plane to version "v1.33.0" (timeout: 5m0s)...
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests1825839740"
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-10-26-13-47-02/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-10-26-13-47-02/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-10-26-13-47-02/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moving new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backing up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2025-10-26-13-47-02/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This can take up to 5m0s
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upgrade/control-plane] The control plane instance for this node was successfully upgraded!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrade/kubeconfig] The kubeconfig files for this node were successfully upgraded!
W1026 13:49:31.482539   26566 postupgrade.go:117] Using temporary directory /etc/kubernetes/tmp/kubeadm-kubelet-config1151700154 for kubelet config. To override it set the environment variable KUBEADM_UPGRADE_DRYRUN_DIR
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config1151700154/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade/kubelet-config] The kubelet configuration for this node was successfully upgraded!
[upgrade/bootstrap-token] Configuring bootstrap token and cluster-info RBAC rules
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade] SUCCESS! A control plane node of your cluster was upgraded to "v1.33.0".

[upgrade] Now please proceed with upgrading the rest of the nodes by following the right order.
```



Notice kubectl get nodes still shows v1.32.0

```sh
➜  kubectl get no
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready,SchedulingDisabled   control-plane   45m   v1.32.0
node01         Ready                      <none>          44m   v1.32.0
```


Lets update our kubelet to the same version we updated kubeadm earlier:

Next version: 1.33.0


```shell
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.0-1.1' kubectl='1.33.0-1.1' && \
sudo apt-mark hold kubelet kubectl
```

```sh
✖ sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.0-1.1' kubectl='1.33.0-1.1' && \
sudo apt-mark hold kubelet kubectl
kubelet was already not on hold.
kubectl was already not on hold.
Hit:2 https://download.docker.com/linux/ubuntu jammy InRelease                                          
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  InRelease 
Hit:3 http://security.ubuntu.com/ubuntu jammy-security InRelease                 
Hit:4 http://archive.ubuntu.com/ubuntu jammy InRelease     
Hit:5 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 84 not upgraded.
Need to get 27.5 MB of archives.
After this operation, 7,090 kB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  kubectl 1.33.0-1.1 [11.7 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  kubelet 1.33.0-1.1 [15.9 MB]
Fetched 27.5 MB in 0s (65.2 MB/s) 
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 20567 files and directories currently installed.)
Preparing to unpack .../kubectl_1.33.0-1.1_amd64.deb ...
Unpacking kubectl (1.33.0-1.1) over (1.32.0-1.1) ...
Preparing to unpack .../kubelet_1.33.0-1.1_amd64.deb ...
Unpacking kubelet (1.33.0-1.1) over (1.32.0-1.1) ...
Setting up kubectl (1.33.0-1.1) ...
Setting up kubelet (1.33.0-1.1) ...
kubelet set on hold.
kubectl set on hold.
```


```sh
➜  kubectl uncordon controlplane
node/controlplane uncordoned

➜  kubectl get no
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   51m   v1.32.0
node01         Ready    <none>          50m   v1.32.0
```



Lets restart kubelet service:

```sh
➜  systemctl daemon-reload
➜  systemctl restart kubelet
```


Notice the controlplane version is now 1.33

```sh
➜  kubectl get no
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   51m   v1.33.0
node01         Ready    <none>          50m   v1.32.0
```


#### Node01 upgrade


```sh
✖ kubectl drain node01 --ignore-daemonsets 
node/node01 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-pdcjt, kube-system/kube-proxy-5s5n5
evicting pod kube-system/coredns-674b8bbfcf-8zx22
evicting pod default/blue-69968556cc-vtwgc
evicting pod default/blue-69968556cc-4qmwx
evicting pod default/blue-69968556cc-52hj8
evicting pod default/blue-69968556cc-7hmjc
evicting pod kube-system/coredns-674b8bbfcf-2lst4
evicting pod default/blue-69968556cc-9kb9c
pod/blue-69968556cc-52hj8 evicted
pod/blue-69968556cc-9kb9c evicted
pod/blue-69968556cc-vtwgc evicted
pod/blue-69968556cc-4qmwx evicted
pod/blue-69968556cc-7hmjc evicted
pod/coredns-674b8bbfcf-8zx22 evicted
pod/coredns-674b8bbfcf-2lst4 evicted
node/node01 drained
```


```shell
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.0-1.1' kubectl='1.33.0-1.1' && \
sudo apt-mark hold kubelet kubectl
```


```sh
✖ sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm='1.33.0-1.1' && sudo apt-mark hold kubead
m
kubeadm was already not on hold.
Hit:2 https://download.docker.com/linux/ubuntu jammy InRelease                                                                            
Hit:3 http://archive.ubuntu.com/ubuntu jammy InRelease                                                                                    
Hit:4 http://archive.ubuntu.com/ubuntu jammy-updates InRelease                                                                       
Hit:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  InRelease                              
Hit:5 http://archive.ubuntu.com/ubuntu jammy-backports InRelease    
Hit:6 http://security.ubuntu.com/ubuntu jammy-security InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 102 not upgraded.
Need to get 12.7 MB of archives.
After this operation, 3,592 kB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.33/deb  kubeadm 1.33.0-1.1 [12.7 MB]
Fetched 12.7 MB in 0s (43.4 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 17482 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.33.0-1.1_amd64.deb ...
Unpacking kubeadm (1.33.0-1.1) over (1.32.0-1.1) ...
Setting up kubeadm (1.33.0-1.1) ...
kubeadm set on hold.
```

```sh
➜  sudo kubeadm upgrade node
[upgrade] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[upgrade] Use 'kubeadm init phase upload-config --config your-config-file' to re-upload it.
[upgrade/preflight] Running pre-flight checks
        [WARNING SystemVerification]: cgroups v1 support is in maintenance mode, please migrate to cgroups v2
[upgrade/preflight] Skipping prepull. Not a control plane node.
[upgrade/control-plane] Skipping phase. Not a control plane node.
[upgrade/kubeconfig] Skipping phase. Not a control plane node.
W1026 14:07:07.614489   65397 postupgrade.go:117] Using temporary directory /etc/kubernetes/tmp/kubeadm-kubelet-config2950372490 for kubelet config. To override it set the environment variable KUBEADM_UPGRADE_DRYRUN_DIR
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config2950372490/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade/kubelet-config] The kubelet configuration for this node was successfully upgraded!
[upgrade/addon] Skipping the addon/coredns phase. Not a control plane node.
[upgrade/addon] Skipping the addon/kube-proxy phase. Not a control plane node.
```





```sh
➜  sudo systemctl daemon-reload
sudo systemctl restart kubelet
```



```sh
controlplane ~ ➜  kubectl get no
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready                      control-plane   59m   v1.33.0
node01         Ready,SchedulingDisabled   <none>          58m   v1.33.0

controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned

controlplane ~ ➜  kubectl get no
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   59m   v1.33.0
node01         Ready    <none>          59m   v1.33.0
```





## kubernetes backup


I good strategy to backup all kubernetes resources is:

```sh
kubectl get all --all-namespaces -o yaml > all-deploy-resources-backup.yaml
```



#### ETCD

```sh
etcdctl snapshot save snapshot-etcd.db
etcdctl snapshot status snapshot-etcd.db # to see the status of the snapshot
# to restore the backup snapshot
systemctl stop kube-apiserver # to restore the backup we need to stop kube-apiserver 
etcdctl restore snapshot.db --data-dir /dir-where-your-backup-file-is-stored
# then edit your etcd.service config file and set --data-dir=/dir-where-your-backup-file-is-stored, new users won't have access to the new cluster by default.
systemctl daemon-reload
systemctl restart etcd 
systemctl start kube-apiserver
```

For each etcdctl command we need specify the endpoints, cacert, cert and key.

```sh
ectcdctl snapshot save snapshot.db \
--endpoints=htts://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server.crt \
--key=/etc/etcd/etcd-server.key
```


# **WORKING WITH ETCDCTL & ETCDUTL**

`etcdctl` is a command line client for [etcd](https://github.com/coreos/etcd).

In all our Kubernetes hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

To make use of `etcdctl` for tasks such as backup, verify it is running on API version 3.x:

etcdctl version

Example:

controlplane ~ ➜ etcdctl version etcdctl version: 3.5.16 API version: 3.5

## **Backing Up ETCD**

### Using `etcdctl` (Snapshot-based Backup)

To take a snapshot from a running etcd server, use:

ETCDCTL_API=3 etcdctl \ --endpoints=[https://127.0.0.1:2379](https://127.0.0.1:2379/) \ --cacert=/etc/kubernetes/pki/etcd/ca.crt \ --cert=/etc/kubernetes/pki/etcd/server.crt \ --key=/etc/kubernetes/pki/etcd/server.key \ snapshot save /backup/etcd-snapshot.db

### Required Options

- `--endpoints` points to the etcd server (default: localhost:2379)
- `--cacert` path to the CA cert
- `--cert` path to the client cert
- `--key` path to the client key

### Using `etcdutl` (File-based Backup)

For offline file-level backup of the data directory:

```sh
etcdutl backup \ --data-dir /var/lib/etcd \ --backup-dir /backup/etcd-backup
```

This copies the etcd backend database and WAL files to the target location.


### Checking Snapshot Status

You can inspect the metadata of a snapshot file using:

etcdctl snapshot status /backup/etcd-snapshot.db \ --write-out=table

This shows details like size, revision, hash, total keys, etc. It is helpful to verify snapshot integrity before restore.

## **Restoring ETCD**

### Using `etcdutl`

To restore a snapshot to a new data directory:

etcdutl snapshot restore /backup/etcd-snapshot.db \ --data-dir /var/lib/etcd-restored

To use a backup made with `etcdutl backup`, simply copy the backup contents back into `/var/lib/etcd` and restart etcd.

## **Notes**

- `etcdctl snapshot save` is used for creating `.db` snapshots from live etcd clusters.
- `etcdctl snapshot status` provides metadata information about the snapshot file.
- `etcdutl snapshot restore` is used to restore a `.db` snapshot file.
- `etcdutl backup` performs a raw file-level copy of etcd’s data and WAL files without needing etcd to be running.



### checking etcd version

```sh
➜  kubectl describe po -n kube-system etcd-controlplane | grep -i image:
    Image:         registry.k8s.io/etcd:3.6.4-0
```




# Kubernetes Security


