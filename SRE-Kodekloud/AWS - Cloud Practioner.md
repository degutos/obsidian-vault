
## What is Cloud computing

Cloud computing is the on-demand delivery of IT resources, particularly compute power, application hosting, database application, networking and more.
It works on a client-server model of computing. Website / CLI / API calls.
Pay only for what you request or use.
Access your requested resources in seconds or minutes.


### Summary:

- Shared responsibility between Cloud provider and you

- Three models of deployment: Cloud, On-premises and Hybrid

- Works on a client-server model 

- Pay-as-you-go


## Amazon Web Services

- Console: great to learn and confirm.
- AWS CLI: Great for engineers and operations.
- AWS SDK: For developers and different types of coders. It uses the APIs. 


Amazon first cloud provider in the market.
AWS was launched in 2006 with S3 as the first service.
AWS grown to 300+ services.
Sign-up for free
Designed for failure
Decouple components
Implement elasticity 
Think parallel and run concurrent.


### Billing 

- Some services are free tier, some free for a year.
- On-demand, you pay only for what you use.
- Reservations: 1 to 3 years AWS will give discount for this.
- Volume discounts (S3 standard). More you use less it will cost.
- Prices can lower from time to time.



## IAM - Identity Access Management

- root: super user
- email: unike
- credit card can hold several different accts
- User
- Groups
- Role
- Policy

### Policy

Policy can determine what resources a user, group or role has access to and how can they manage each resources.
IAM handles who is authenticated and what are they authorized to do.
- Grant rule
- Deny rule
- Refined like user can access but not delete 
- Least privileges 


### IAM User


- Each user has to be their own acct
- User is not only for users, it can be only application user
- User need to have specific permission to do the work



## Global Infrastructure

- Regions are locations to which certain services can be deployed
- Not all services are available in all regions
- Availability zones (AZ) are isolated and independent datacenters inside Regions
- Edge locations are smaller points of presence where services are run closer to customers, mostly content deliver
- Local zones are extensions of AWS regions located near users in select metropolitan areas, provide services like ECS and EBS.



## AWS Networking


- VPC isolates computing resources from other computing resources available in the cloud
- You can have more than one VPC per region like Prod, Staging, Dev, Test, etc.
- VPCs are isolated to a region
- VPC CIDR block defines the IP addresses a VPC can use
- Subnets are a range of IP addresses with VPC
- Subnets reside with a single Availability Zone
- Subnets can be made public or private using Internet Gateways and Nat Gateways
- Internet Gateways allow subnets to communicate with internet and vice versa
- NAT Gateways allow subnets to talk to to the internet but connections must be initiated from within the VPC
- Virtual Private Gateway enable secure access to private resources over the internet
- Direct Connect (DX) is a direct connection into an AWS regions that provide low latency and high speeds



## Default VPC


- Every region has a default VPC with default subnets, SG and NACLs
- The CIDR block for the default is 172.31.0.0/16 (65.536 addresses)
- 
- The default VPC and its subnets have outbound access to the intenret by default
- One default subnet in each Availability zone 172.31.0.0/20, 172.31.16.0/20, 172.31.32.0/20 (4.096 addresses each subnet)
- The security Groups allow outbound and the NACLs are open in both inbound and outbound directions.


### Firewall

Firewall monitor traffic and only allow traffic permitted by a set of predefined rules.
Firewall rules are broken down into INBOUND and OUTBOUND rules.

#### Stateless firewall

Stateless firewall must be configured to allow both INBOUND and OUTBOUND traffic.
Stateless firewall are not intelligent, we need allow the request IN and the response OUT.
If we allow INBOUND connection to port 443 and also need allow the OUTBOUND response to port 1024 to 65.545.


#### Stateful firewall

Stateful firewall are intelligent enough to understand which request and response are part of the same connection. Stateful firewall we allow only connection into INBOUND and it knows if the connection is allowed to request (IN) it should be automatically allowed to respond (OUT).
If we allow INBOUND port 443 we don't need allow OUTBOUND, the firewall will automatically allow the response from port 1024 to 65.545
If a request is permitted, the response automatically permitted as well in a stateful firewll.


#### NACLs - Network access control list

NACLs filter traffic entering and leaving a subnet
NACLs do not filter traffic within a subnet, If your subnet has two or more instances those instances will communicate to each other since they are in the same network, and NACL will not filter traffic between them. NACL works on traffic entering and leaving your subnet not within the subnet.
NACLs are stateless firewalls, so rules MUST be set for both inbound and outbound.

#### SG - Security Group

SG act as a firewall for individual resources like EC2 instances, LB, RDS, etc, usually attached to Network Interface.
Security Groups are stateful, so only the request needs to be allowed. SG is intelligent enough.
You can attach multiples SG to a resource and you can attach one SG to multiple resources. If you have many web servers that allows traffic on port 443 you can create a SG with that rule and attach to all WebServers. 


## Storage - CORE Services


### Types of Storage


- Block Storage
- File Storage
- Object Storage



### Block Storage

#### Amazon Elastic Block Storage - EBS


Block Storage breaks up data into blocks and then stores those blocks as separate pieces, each with a unique identifier.
A collection of blocks can be presented to the OS as a volume. The operating system then creates a filesystem on top of it.
A collection of blocks can be presented as a hard drive. Block storage is bootable, which means operating systems can be installed on it.
To an Instance to connect to the a Block Storage they need to exist in the same availability zone. You can not use attach a VSI to a block storage that belongs to another availability zone.
Instances are detached and attached to the block storage when stop and start the instances.


### File Storage

#### Amazon Elastic File system - EFS


File Storage stores data in a hierarchical structure of files and folders
Similar to how your computer stores files and folders.
File system that is accessible remotely. Creates subdirectories and stores and access files within it.
File storage can be mounted
Not bootable, so you cannot install an operating system on it.
File storage is accessible over the network
Multiple client can access the same data.


### S3 - Object Storage


Object Storage stores objects. Objects are nothing else more than files
Can store any type of file. 
It is similar to DropBox.
Does not have a folder structure. Flat file structure, everything is in the same folder
Since there is no folder structure, object storage cannot be MOUNTED or BOOTED.
It can be only used to store and retrieve files
Great for storing logs and media files


#### Storage Classes


##### S3 standard (default)


- AWS replicate your data across different availability zones, if 02 Datacenter go down you still can access your data
- 99,999999999% durability (11 nines)
- Costs are changed per giga per mount
- Also pay per data out egress as per data out.


##### S3 Standard-IA

- IA - Infrequent access
- Cheaper than standard default
- Can handle also 02 AZ down
- 99,999999999% durability (11 nines)
- Also pay for outbound fee for egress
- Has a retrieval fee 
- Minimum duration charge of 90 days
- Minimum size charge of 128 KB per object, files with 1KB will be charged for 128KB
- Make sure you won't access this file frequent and the files are not too small



##### S3 One Zone-IA

- IA Infrequent access
- No redudance with AZ and you pay cheaper 
- Has retrieval fee
- Minimum duration charge of 90 days
- Minimum size object is 128 KB


##### S3 Glacier - Instant 


- Low-cost option for rarely accessed data
- Charged per GB outbound 
- Has retrieval free
- Minimum duration charge of 90 days
- Minimum size charge of 129 KB per object
- Very cheap storage 
- Higher retrieval cost
- Longer minimum duration
- Cheaper than S3 standard and S3 IA





##### S3 Glacier - Flexible 


- Has retrieval fee
- Minimum duration charge of 90 days
- Minimum size charge of 40 KB per object
- Charged per GB outbound egress
- Bulk: 5-12h to wait retrieval
- Expedited: 1-5m
- Standard: 3-5h



##### S3 Glacier deep archive

- Data almost never retrieved
- Cheapest 
- Charged per GB outbound
- Has retrieval fee
- Minimum duration 90 days
- Minimum size 40 KB
- Standard: 12h to retrieve your data
- Bulk: 48h to retrieve back


##### S3 Intelligent tiering


- Automatically reduces storage costs by intelligently moving data to the most cost-effective access tier
- Objects will also incur a monitoring/automation cost per 1000 objects.


## EBS - Elastic Block Storage  Demo


- We can create a EBS in the same AZ an attach to any Instance
- Usually EBS will have name of xvdf in /dev/xvdf 
- Once Instance is booted up and EBS is attached we can run:


```sh
lsblk
```


Check file system in the EBS

```sh
sudo file -s /dev/xvdf
```

Note that if you see /dev/xvdf: data the EBS has no file system yet

Lets create a file system in the EBS

```sh
sudo mkfs.ext4 /dev/xvdf
```

Check file system again

```sh
sudo file -s /dev/xvdf
```

You should see something like ext4 filesystem and UUID

Now we need mount 

```sh
sudo mkdir /ebsdemo
sudo mount /dev/xvdf /ebsdemo
lsblk # see our xvdf is mounted under /ebsdemo
df -h # see /dev/xvd f mounted under /ebsdemo
```


Lets now add to FSTAB


```sh
sudo blkid # to get the UUID, copy the xvdf UUID
sudo vi /etc/fstab
```

Add
```
UUID=d00_UIDxxxx  /ebsdemo  ext4  defaults
```



## EFS - Elastic File System


We can create an Amazon EFS and attach to a VSI.
Create your EFS and choose Endpoints acordingly to your AZ
Install the package need for EFS mount 


```sh
sudo yum install -y amazon-efs-utils
```

Lets now mount 

```sh
sudo mkdir /efs
sudo mount.efs instance_id /efs
df -h # check volume mounted
```

PS: to use EFS in other distros: https://github.com/aws/efs-utils?tab=readme-ov-file#on-other-linux-distributions 
§

Lets now edit our fstab and make this persistent

```sh
sudo vi /etc/fstab
fs-volume-id:/ /efs efs  _netdev,noresvport,tls,iam 0 0 
```





