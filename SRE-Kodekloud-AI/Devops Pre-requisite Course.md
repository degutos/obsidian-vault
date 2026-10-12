

> ## Documentation Index
> Fetch the complete documentation index at: https://notes.kodekloud.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Devops Tools

> This guide explores essential DevOps tools and practices for developing, deploying, and monitoring applications from initial idea to production.

When diving into DevOps, you encounter a diverse set of tools, including Docker, Kubernetes, Ansible, Terraform, Git, GitHub, Jenkins, Prometheus, and Grafana. Although this array of technologies might initially seem overwhelming, this guide walks you through a real-world scenario that demonstrates how each tool plays a crucial role as your application and infrastructure evolve.

## From Idea to First Release

Every great project starts with an idea—imagine building a website that books tickets to Mars, helping users avoid long queues and high prices. As with any innovative project, you start writing code. Hours later, the first version is ready. However, this version is running on your local machine, accessible via HTTP on localhost (port 8080):

```python theme={null}
def book_my_ticket_to_mars():
    # world changing code here

# starts here
if __name__ == '__main__':
    book_my_ticket_to_mars()
```

Running your application locally is ideal for early testing, but it won’t suffice when you need to share it with the world. To make your application accessible 24/7, you must deploy it on a server—whether it’s a physical machine, a virtual machine, or a cloud instance. Simply copying the code isn’t enough; the server also requires the correct runtime environment (like Python or Java) and all necessary libraries with the proper versions. Here, your laptop serves as the development environment, while the server becomes the production environment.

Once deployed, users can access your application using the server’s IP address. However, an IP address isn’t always user friendly. That’s why you purchase a domain name and map it to your server.

<Frame>
  ![The image contrasts development and production environments, showing local development on a laptop and production on servers, with different URLs and programming languages.](https://kodekloud.com/kk-media/image/upload/v1752873436/notes-assets/images/DevOps-Pre-Requisite-Course-Devops-Tools/frame_170.jpg)
</Frame>

## The Basic Workflow: Building and Deploying

Initially, your workflow consists of a few critical steps:

1. Develop the code on your local machine using an editor like VS Code or PyCharm.
2. Build the code by converting it into an executable format or binary (using tools such as Python setuptools, Maven, or Gradle).
3. Deploy the built executable to your production server.

A build script is often used to automate the build process:

```bash theme={null}
$ ./build.sh
$ ./app
```

## Scaling Collaboration with Git and GitHub

As your website gains popularity and your user base expands, you invite other developers to contribute. Each team member works on their local environment, which can lead to conflicts when multiple developers modify the same files. This is where Git, a version control system, becomes indispensable. It allows developers to pull the latest changes, merge their contributions, and push updates back to a shared repository.

Platforms like GitHub, GitLab, and Bitbucket act as cloud-based hubs that streamline code collaboration, version control, and project management. For example, common Git commands include:

```bash theme={null}
$ git pull
$ git push
```

At this stage, while Git manages version control, GitHub provides a centralized repository along with a user-friendly interface for project management.

## Introducing CI/CD for Automated Deployments

With multiple developers continuously pushing new features and bug fixes, manually transferring code between environments is inefficient and prone to errors. A dedicated build server is implemented to fetch the latest code, build it, and deploy it first to a testing environment. After successful tests, the new build is moved to production.

<Callout icon="lightbulb" color="#1CB2FE">
  Automating your deployment process reduces errors and accelerates feature delivery. Tools like Jenkins, GitHub Actions, and GitLab CI/CD enable continuous integration and deployment, facilitating multiple daily deployments and quicker feedback loops.
</Callout>

The automated pipeline typically works as follows:

* Code is pulled from the GitHub repository.
* The build server compiles the code.
* Automated tests run on the build.
* If tests pass, the executable is deployed to production.

<Frame>
  ![The image illustrates a CI/CD pipeline process using Jenkins, GitHub Actions, and GitLab CI/CD, showing stages of development, build, test, and deployment.](https://kodekloud.com/kk-media/image/upload/v1752873437/notes-assets/images/DevOps-Pre-Requisite-Course-Devops-Tools/frame_530.jpg)
</Frame>

## Packaging with Containers

Even with CI/CD automation, ensuring that applications run consistently across different environments requires precise dependency management. Manually installing every dependency on each server is inefficient and error-prone. Containers solve this problem by encapsulating your application and its dependencies in an image that can run uniformly across any system.

Docker is one of the most popular containerization technologies. Developers create a Dockerfile that describes the application environment. During the build process, Docker generates an image that can be deployed on any server:

```bash theme={null}
$ vi Dockerfile
$ docker build -t mars_ticket_app .
$ docker run -d -p 80:80 mars_ticket_app
```

Containers also provide process isolation, allowing multiple containers to run on the same server without interference.

## Scaling with Container Orchestration

As user demand grows, you may need to run containers on several servers, a process that becomes challenging to manage manually. Container orchestration platforms like Kubernetes help automate tasks such as starting new containers when demand increases, shutting them down when demand decreases, and restarting failed containers. Kubernetes automatically scales and manages containerized applications, ensuring consistency and efficiency across your infrastructure.

A typical workflow in this scenario involves developers pushing code to GitHub, CI/CD pipelines building Docker images, testing the application, and finally Kubernetes deploying the containers to production.

## Infrastructure as Code with Terraform

Manually provisioning servers to ensure they all have the same configuration is a complex and error-prone task. To address this, tools like Terraform allow you to define your infrastructure as code. Using manifest files, Terraform provisions new servers with consistent configurations, such as operating system versions, storage, kernel settings, and container runtimes.

Below is an excerpt from a Terraform manifest that provisions three servers with identical configurations:

```hcl theme={null}
resource "aws_instance" "node01" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  tags = {
    Name = "controlplane"
  }
}

resource "aws_instance" "node02" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  tags = {
    Name = "worker01"
  }
}

resource "aws_instance" "node03" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  tags = {
    Name = "worker02"
  }
}
```

Modifications to your infrastructure are made in the Terraform code and applied with:

```bash theme={null}
$ terraform apply
```

## Post-Provisioning Configuration with Ansible

Provisioning servers is only part of the equation. After your servers are up and running, they require further configuration—installing necessary software, configuring services, and adjusting system settings. Ansible automates these post-provisioning tasks through playbooks, which are version-controlled like any other code. While Terraform handles the provisioning, Ansible excels at configuring and managing software on your servers.

An example Ansible playbook for deploying a Kubernetes cluster might look like this:

```yaml theme={null}
---
# This playbook deploys the Kubernetes cluster.
- name: Apply common configuration to all nodes
  hosts: all
  remote_user: root
  roles:
    - common

- name: Configure and deploy the controlplane
  hosts: controlplane
  remote_user: root
  roles:
    - kubernetes_controlplane

- name: Configure and deploy worker nodes
  hosts: workers
  remote_user: root
  roles:
    - kubernetes_workers
```

## Monitoring Infrastructure with Prometheus and Grafana

Deploying and configuring your infrastructure is only half the battle; ongoing monitoring is essential to ensure everything runs smoothly. Prometheus gathers metrics such as CPU usage, memory consumption, and process statuses from your servers. Grafana then visualizes this data through dashboards, helping you quickly spot trends and potential issues.

<Frame>
  ![The image illustrates a CI/CD pipeline using Infrastructure as Code (IaC) with tools like Ansible, Terraform, Prometheus, and Grafana, showing development, build, test, and production stages.](https://kodekloud.com/kk-media/image/upload/v1752873438/notes-assets/images/DevOps-Pre-Requisite-Course-Devops-Tools/frame_940.jpg)
</Frame>

## Conclusion

This guide has taken you on a journey from your initial idea to a fully automated, scalable, and monitored production environment. By integrating development tools, CI/CD pipelines, containerization, orchestration, infrastructure provisioning, configuration management, and monitoring solutions, DevOps practices enable seamless transitions from concept to high-quality software delivery.

For further reading, explore these resources:

* [Kubernetes Documentation](https://kubernetes.io/docs/)
* [Docker Hub](https://hub.docker.com/)
* [Terraform Registry](https://registry.terraform.io/)

DevOps is the convergence of people, processes, and tools working in harmony to ensure that your software is deployed reliably and efficiently.

<CardGroup>
  <Card title="Watch Video" icon="video" cta="Learn more" href="https://learn.kodekloud.com/user/courses/devops-pre-requisite-course/module/10cc998e-4162-40ec-be18-d26a77d7a474/lesson/dbafa3a2-d069-4c84-8977-d405f017bd04" />
</CardGroup>


This documentation is built and hosted on [Mintlify](https://mintlify.com), a developer documentation platform.



## Package Management


```sh
$ rpm -i telnet.rpm # install
$ rpm -e telnet.rpm # remove
$ rpm -q telnet.rpm # query
```


### YUM

```sh
$ yum install ansible
```


Remote repository and information of configuration 

```sh
ls /etc/yum.repos.d
```


```sh
$ yum repolist
```


```sh
cat /etc/yum.repos.d/Centos-Base.repo
```


```sh
yum list ansible # search for a package 
yum remove ansible # remove package
yum --showduplicates list ansible # check duplicate package
yum install ansible-2.4.2.0 # install package
```


How to know which package is installed using RPM

```sh
thor@host01 ~$ rpm -q python3 ansible telnet openssh-server
python3-3.9.25-5.el9.x86_64
package ansible is not installed
telnet-0.17-85.el9.x86_64
openssh-server-9.9p1-7.el9.x86_64
```


### Installing a package with RPM

```sh
thor@host01 /opt$ sudo rpm -ivh ftp-0.17-89.el9.x86_64.rpm 
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:ftp-0.17-89.el9                  ################################# [100%]
```


### Removing or uninstalling a package with RPM

```sh
thor@host01 /opt$ sudo rpm -qa | grep ftp
ftp-0.17-89.el9.x86_64

thor@host01 /opt$ sudo rpm -e ftp-0.17-89.el9.x86_64
thor@host01 /opt$
```


### Installing package with DNF 

```sh
thor@host01 ~$ sudo dnf install maven
Node.js Packages for Linux RPM based distros - x86_64                                         3.1 MB/s | 833 kB     00:00    
N|Solid Packages for Linux RPM based distros - x86_64                                         2.1 MB/s | 503 kB     00:00    
CentOS Stream 9 - BaseOS                                                                       12 MB/s | 8.9 MB     00:00    
CentOS Stream 9 - AppStream                                                                    26 MB/s |  29 MB     00:01    
CentOS Stream 9 - Extras packages                                                              47 kB/s |  22 kB     00:00    
Extra Packages for Enterprise Linux 9 - x86_64                                                 13 MB/s |  20 MB     00:01    
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                          3.5 kB/s | 2.5 kB     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64                                         398 kB/s | 411 kB     00:01    
Dependencies resolved.
==============================================================================================================================
 Package                                           Architecture     Version                         Repository           Size
==============================================================================================================================
Installing:
 maven                                             noarch           1:3.6.3-23.el9                  appstream            16 k
Installing dependencies:
 apache-commons-cli                                noarch           1.4-20.el9                      appstream            70 k
 apache-commons-codec                              noarch           1.15-10.el9                     appstream           301 k
 apache-commons-io                                 noarch           1:2.8.0-12.el9                  appstream           283 k
 apache-commons-lang3                              noarch           3.12.0-10.el9                   appstream           558 k
 atinject                                          noarch           1.0.3-9.el9                     appstream            23 k
 cdi-api                                           noarch           2.0.2-9.el9                     appstream            53 k
 google-guice                                      noarch           4.2.3-12.el9                    appstream           473 k
 guava                                             noarch           30.1-10.el9                     appstream           2.3 M
 httpcomponents-client                             noarch           4.5.13-7.el9                    appstream           656 k
 httpcomponents-core                               noarch           4.4.13-11.el9                   appstream           632 k
 jakarta-annotations                               noarch           1.3.5-16.el9                    appstream            45 k
 jansi                                             x86_64           2.3.3-9.el9                     appstream           109 k
 java-17-openjdk                                   x86_64           1:17.0.20.0.8-1.2.el9           appstream           428 k
 java-17-openjdk-headless                          x86_64           1:17.0.20.0.8-1.2.el9           appstream            44 M
 javapackages-tools                                noarch           6.4.0-1.el9                     appstream            34 k
 jcl-over-slf4j                                    noarch           1.7.30-16.el9                   appstream            24 k
 jsoup                                             noarch           1.13.1-13.el9                   appstream           376 k
 jsr-305                                           noarch           3.0.2-9.el9                     appstream            35 k
 maven-lib                                         noarch           1:3.6.3-23.el9                  appstream           1.5 M
 maven-openjdk17                                   noarch           1:3.6.3-23.el9                  appstream           7.8 k
 maven-resolver                                    noarch           1:1.6.1-14.el9                  appstream           516 k
 maven-shared-utils                                noarch           3.3.4-7.el9                     appstream           151 k
 maven-wagon                                       noarch           3.4.2-10.el9                    appstream           112 k
 plexus-cipher                                     noarch           1.7-30.el9                      appstream            26 k
 plexus-classworlds                                noarch           2.6.0-14.el9                    appstream            60 k
 plexus-containers-component-annotations           noarch           2.1.0-13.el9                    appstream            18 k
 plexus-interpolation                              noarch           1.26-15.el9                     appstream            79 k
 plexus-sec-dispatcher                             noarch           1.4-41.el9                      appstream            33 k
 plexus-utils                                      noarch           3.3.0-14.el9                    appstream           253 k
 publicsuffix-list                                 noarch           20210518-3.el9                  appstream            83 k
 sisu                                              noarch           1:0.3.4-14.el9                  appstream           510 k
 slf4j                                             noarch           1.7.30-16.el9                   appstream            69 k
Installing weak dependencies:
 java-17-openjdk-devel                             x86_64           1:17.0.20.0.8-1.2.el9           appstream           4.7 M

Transaction Summary
==============================================================================================================================
Install  34 Packages

Total download size: 59 M
Installed size: 208 M
Is this ok [y/N]: y
Downloading Packages:
(1/34): apache-commons-cli-1.4-20.el9.noarch.rpm                                              541 kB/s |  70 kB     00:00    
(2/34): apache-commons-codec-1.15-10.el9.noarch.rpm                                           1.8 MB/s | 301 kB     00:00    
(3/34): apache-commons-io-2.8.0-12.el9.noarch.rpm                                             1.6 MB/s | 283 kB     00:00    
(4/34): apache-commons-lang3-3.12.0-10.el9.noarch.rpm                                         7.7 MB/s | 558 kB     00:00    
(5/34): atinject-1.0.3-9.el9.noarch.rpm                                                       610 kB/s |  23 kB     00:00    
(6/34): cdi-api-2.0.2-9.el9.noarch.rpm                                                        1.7 MB/s |  53 kB     00:00    
(7/34): google-guice-4.2.3-12.el9.noarch.rpm                                                  9.7 MB/s | 473 kB     00:00    
(8/34): httpcomponents-client-4.5.13-7.el9.noarch.rpm                                          11 MB/s | 656 kB     00:00    
(9/34): guava-30.1-10.el9.noarch.rpm                                                           32 MB/s | 2.3 MB     00:00    
(10/34): httpcomponents-core-4.4.13-11.el9.noarch.rpm                                          13 MB/s | 632 kB     00:00    
(11/34): jakarta-annotations-1.3.5-16.el9.noarch.rpm                                          1.2 MB/s |  45 kB     00:00    
(12/34): jansi-2.3.3-9.el9.x86_64.rpm                                                         3.0 MB/s | 109 kB     00:00    
(13/34): java-17-openjdk-17.0.20.0.8-1.2.el9.x86_64.rpm                                       5.0 MB/s | 428 kB     00:00    
(14/34): java-17-openjdk-devel-17.0.20.0.8-1.2.el9.x86_64.rpm                                  33 MB/s | 4.7 MB     00:00    
(15/34): javapackages-tools-6.4.0-1.el9.noarch.rpm                                            516 kB/s |  34 kB     00:00    
(16/34): jcl-over-slf4j-1.7.30-16.el9.noarch.rpm                                              658 kB/s |  24 kB     00:00    
(17/34): jsoup-1.13.1-13.el9.noarch.rpm                                                        10 MB/s | 376 kB     00:00    
(18/34): jsr-305-3.0.2-9.el9.noarch.rpm                                                       776 kB/s |  35 kB     00:00    
(19/34): maven-3.6.3-23.el9.noarch.rpm                                                        386 kB/s |  16 kB     00:00    
(20/34): maven-openjdk17-3.6.3-23.el9.noarch.rpm                                              231 kB/s | 7.8 kB     00:00    
(21/34): maven-lib-3.6.3-23.el9.noarch.rpm                                                     19 MB/s | 1.5 MB     00:00    
(22/34): maven-resolver-1.6.1-14.el9.noarch.rpm                                               6.9 MB/s | 516 kB     00:00    
(23/34): maven-shared-utils-3.3.4-7.el9.noarch.rpm                                            3.4 MB/s | 151 kB     00:00    
(24/34): maven-wagon-3.4.2-10.el9.noarch.rpm                                                  3.0 MB/s | 112 kB     00:00    
(25/34): plexus-cipher-1.7-30.el9.noarch.rpm                                                  864 kB/s |  26 kB     00:00    
(26/34): java-17-openjdk-headless-17.0.20.0.8-1.2.el9.x86_64.rpm                               61 MB/s |  44 MB     00:00    
(27/34): plexus-classworlds-2.6.0-14.el9.noarch.rpm                                           167 kB/s |  60 kB     00:00    
(28/34): plexus-containers-component-annotations-2.1.0-13.el9.noarch.rpm                       51 kB/s |  18 kB     00:00    
(29/34): plexus-utils-3.3.0-14.el9.noarch.rpm                                                 6.8 MB/s | 253 kB     00:00    
(30/34): plexus-sec-dispatcher-1.4-41.el9.noarch.rpm                                          883 kB/s |  33 kB     00:00    
(31/34): plexus-interpolation-1.26-15.el9.noarch.rpm                                          1.3 MB/s |  79 kB     00:00    
(32/34): publicsuffix-list-20210518-3.el9.noarch.rpm                                          2.6 MB/s |  83 kB     00:00    
(33/34): sisu-0.3.4-14.el9.noarch.rpm                                                          14 MB/s | 510 kB     00:00    
(34/34): slf4j-1.7.30-16.el9.noarch.rpm                                                       2.3 MB/s |  69 kB     00:00    
------------------------------------------------------------------------------------------------------------------------------
Total                                                                                          46 MB/s |  59 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: java-17-openjdk-headless-1:17.0.20.0.8-1.2.el9.x86_64                                                1/1 
  Preparing        :                                                                                                      1/1 
  Installing       : slf4j-1.7.30-16.el9.noarch                                                                          1/34 
  Installing       : plexus-utils-3.3.0-14.el9.noarch                                                                    2/34 
  Installing       : java-17-openjdk-headless-1:17.0.20.0.8-1.2.el9.x86_64                                               3/34 
  Running scriptlet: java-17-openjdk-headless-1:17.0.20.0.8-1.2.el9.x86_64                                               3/34 
  Installing       : atinject-1.0.3-9.el9.noarch                                                                         4/34 
  Installing       : plexus-containers-component-annotations-2.1.0-13.el9.noarch                                         5/34 
  Installing       : plexus-classworlds-2.6.0-14.el9.noarch                                                              6/34 
  Installing       : plexus-cipher-1.7-30.el9.noarch                                                                     7/34 
  Installing       : httpcomponents-core-4.4.13-11.el9.noarch                                                            8/34 
  Installing       : apache-commons-lang3-3.12.0-10.el9.noarch                                                           9/34 
  Installing       : apache-commons-io-1:2.8.0-12.el9.noarch                                                            10/34 
  Installing       : maven-shared-utils-3.3.4-7.el9.noarch                                                              11/34 
  Installing       : plexus-sec-dispatcher-1.4-41.el9.noarch                                                            12/34 
  Installing       : cdi-api-2.0.2-9.el9.noarch                                                                         13/34 
  Installing       : sisu-1:0.3.4-14.el9.noarch                                                                         14/34 
  Installing       : java-17-openjdk-1:17.0.20.0.8-1.2.el9.x86_64                                                       15/34 
  Running scriptlet: java-17-openjdk-1:17.0.20.0.8-1.2.el9.x86_64                                                       15/34 
  Installing       : java-17-openjdk-devel-1:17.0.20.0.8-1.2.el9.x86_64                                                 16/34 
  Running scriptlet: java-17-openjdk-devel-1:17.0.20.0.8-1.2.el9.x86_64                                                 16/34 
  Installing       : jcl-over-slf4j-1.7.30-16.el9.noarch                                                                17/34 
  Installing       : publicsuffix-list-20210518-3.el9.noarch                                                            18/34 
  Installing       : plexus-interpolation-1.26-15.el9.noarch                                                            19/34 
  Installing       : jsr-305-3.0.2-9.el9.noarch                                                                         20/34 
  Installing       : guava-30.1-10.el9.noarch                                                                           21/34 
  Installing       : google-guice-4.2.3-12.el9.noarch                                                                   22/34 
  Installing       : jsoup-1.13.1-13.el9.noarch                                                                         23/34 
  Installing       : javapackages-tools-6.4.0-1.el9.noarch                                                              24/34 
  Installing       : maven-openjdk17-1:3.6.3-23.el9.noarch                                                              25/34 
  Installing       : jansi-2.3.3-9.el9.x86_64                                                                           26/34 
  Installing       : jakarta-annotations-1.3.5-16.el9.noarch                                                            27/34 
  Installing       : apache-commons-codec-1.15-10.el9.noarch                                                            28/34 
  Installing       : httpcomponents-client-4.5.13-7.el9.noarch                                                          29/34 
  Installing       : maven-wagon-3.4.2-10.el9.noarch                                                                    30/34 
  Installing       : maven-resolver-1:1.6.1-14.el9.noarch                                                               31/34 
  Installing       : apache-commons-cli-1.4-20.el9.noarch                                                               32/34 
  Installing       : maven-lib-1:3.6.3-23.el9.noarch                                                                    33/34 
  Installing       : maven-1:3.6.3-23.el9.noarch                                                                        34/34 
  Running scriptlet: maven-1:3.6.3-23.el9.noarch                                                                        34/34 
  Running scriptlet: java-17-openjdk-headless-1:17.0.20.0.8-1.2.el9.x86_64                                              34/34 
  Running scriptlet: java-17-openjdk-1:17.0.20.0.8-1.2.el9.x86_64                                                       34/34 
  Running scriptlet: java-17-openjdk-devel-1:17.0.20.0.8-1.2.el9.x86_64                                                 34/34 
  Running scriptlet: maven-1:3.6.3-23.el9.noarch                                                                        34/34 
  Verifying        : apache-commons-cli-1.4-20.el9.noarch                                                                1/34 
  Verifying        : apache-commons-codec-1.15-10.el9.noarch                                                             2/34 
  Verifying        : apache-commons-io-1:2.8.0-12.el9.noarch                                                             3/34 
  Verifying        : apache-commons-lang3-3.12.0-10.el9.noarch                                                           4/34 
  Verifying        : atinject-1.0.3-9.el9.noarch                                                                         5/34 
  Verifying        : cdi-api-2.0.2-9.el9.noarch                                                                          6/34 
  Verifying        : google-guice-4.2.3-12.el9.noarch                                                                    7/34 
  Verifying        : guava-30.1-10.el9.noarch                                                                            8/34 
  Verifying        : httpcomponents-client-4.5.13-7.el9.noarch                                                           9/34 
  Verifying        : httpcomponents-core-4.4.13-11.el9.noarch                                                           10/34 
  Verifying        : jakarta-annotations-1.3.5-16.el9.noarch                                                            11/34 
  Verifying        : jansi-2.3.3-9.el9.x86_64                                                                           12/34 
  Verifying        : java-17-openjdk-1:17.0.20.0.8-1.2.el9.x86_64                                                       13/34 
  Verifying        : java-17-openjdk-devel-1:17.0.20.0.8-1.2.el9.x86_64                                                 14/34 
  Verifying        : java-17-openjdk-headless-1:17.0.20.0.8-1.2.el9.x86_64                                              15/34 
  Verifying        : javapackages-tools-6.4.0-1.el9.noarch                                                              16/34 
  Verifying        : jcl-over-slf4j-1.7.30-16.el9.noarch                                                                17/34 
  Verifying        : jsoup-1.13.1-13.el9.noarch                                                                         18/34 
  Verifying        : jsr-305-3.0.2-9.el9.noarch                                                                         19/34 
  Verifying        : maven-1:3.6.3-23.el9.noarch                                                                        20/34 
  Verifying        : maven-lib-1:3.6.3-23.el9.noarch                                                                    21/34 
  Verifying        : maven-openjdk17-1:3.6.3-23.el9.noarch                                                              22/34 
  Verifying        : maven-resolver-1:1.6.1-14.el9.noarch                                                               23/34 
  Verifying        : maven-shared-utils-3.3.4-7.el9.noarch                                                              24/34 
  Verifying        : maven-wagon-3.4.2-10.el9.noarch                                                                    25/34 
  Verifying        : plexus-cipher-1.7-30.el9.noarch                                                                    26/34 
  Verifying        : plexus-classworlds-2.6.0-14.el9.noarch                                                             27/34 
  Verifying        : plexus-containers-component-annotations-2.1.0-13.el9.noarch                                        28/34 
  Verifying        : plexus-interpolation-1.26-15.el9.noarch                                                            29/34 
  Verifying        : plexus-sec-dispatcher-1.4-41.el9.noarch                                                            30/34 
  Verifying        : plexus-utils-3.3.0-14.el9.noarch                                                                   31/34 
  Verifying        : publicsuffix-list-20210518-3.el9.noarch                                                            32/34 
  Verifying        : sisu-1:0.3.4-14.el9.noarch                                                                         33/34 
  Verifying        : slf4j-1.7.30-16.el9.noarch                                                                         34/34 

Installed:
  apache-commons-cli-1.4-20.el9.noarch                       apache-commons-codec-1.15-10.el9.noarch                         
  apache-commons-io-1:2.8.0-12.el9.noarch                    apache-commons-lang3-3.12.0-10.el9.noarch                       
  atinject-1.0.3-9.el9.noarch                                cdi-api-2.0.2-9.el9.noarch                                      
  google-guice-4.2.3-12.el9.noarch                           guava-30.1-10.el9.noarch                                        
  httpcomponents-client-4.5.13-7.el9.noarch                  httpcomponents-core-4.4.13-11.el9.noarch                        
  jakarta-annotations-1.3.5-16.el9.noarch                    jansi-2.3.3-9.el9.x86_64                                        
  java-17-openjdk-1:17.0.20.0.8-1.2.el9.x86_64               java-17-openjdk-devel-1:17.0.20.0.8-1.2.el9.x86_64              
  java-17-openjdk-headless-1:17.0.20.0.8-1.2.el9.x86_64      javapackages-tools-6.4.0-1.el9.noarch                           
  jcl-over-slf4j-1.7.30-16.el9.noarch                        jsoup-1.13.1-13.el9.noarch                                      
  jsr-305-3.0.2-9.el9.noarch                                 maven-1:3.6.3-23.el9.noarch                                     
  maven-lib-1:3.6.3-23.el9.noarch                            maven-openjdk17-1:3.6.3-23.el9.noarch                           
  maven-resolver-1:1.6.1-14.el9.noarch                       maven-shared-utils-3.3.4-7.el9.noarch                           
  maven-wagon-3.4.2-10.el9.noarch                            plexus-cipher-1.7-30.el9.noarch                                 
  plexus-classworlds-2.6.0-14.el9.noarch                     plexus-containers-component-annotations-2.1.0-13.el9.noarch     
  plexus-interpolation-1.26-15.el9.noarch                    plexus-sec-dispatcher-1.4-41.el9.noarch                         
  plexus-utils-3.3.0-14.el9.noarch                           publicsuffix-list-20210518-3.el9.noarch                         
  sisu-1:0.3.4-14.el9.noarch                                 slf4j-1.7.30-16.el9.noarch                                      

Complete!
```


Installing a specific version with DNF

```sh
thor@host01 ~$ sudo dnf install maven-3.6.3
Last metadata expiration check: 0:06:49 ago on Mon Oct 12 06:12:39 2026.
Dependencies resolved.
==============================================================================================================================
 Package                                           Architecture     Version                         Repository           Size
==============================================================================================================================
Installing:
 maven                                             noarch           1:3.6.3-23.el9                  appstream            16 k
Installing dependencies:
 apache-commons-cli                                noarch           1.4-20.el9                      appstream            70 k
```

### Checking package version installed

```sh
thor@host01 ~$ yum list maven
Installed Packages
maven.noarch                                             1:3.6.3-23.el9                                             @appstream


thor@host01 ~$ rpm -qa maven
maven-3.6.3-23.el9.noarch

thor@host01 ~$ rpm -q --qf '%{EPOCH}:%{VERSION}\n' maven
1:3.6.3
```


### Remove package 

```sh
thor@host01 ~$ sudo dnf remove maven
Dependencies resolved.
==============================================================================================================================
 Package                                          Architecture    Version                           Repository           Size
==============================================================================================================================
Removing:
 maven                                            noarch          1:3.6.3-23.el9                    @appstream           21 k
...                                                                33/33 

Removed:
  apache-commons-cli-1.4-20.el9.noarch                               apache-commons-codec-1.15-10.el9.noarch                  
  apache-commons-io-1:2.8.0-12.el9.noarch                            apache-commons-lang3-3.12.0-10.el9.noarch                
  atinject-1.0.3-9.el9.noarch                                        cdi-api-2.0.2-9.el9.noarch                               
  google-guice-4.2.3-12.el9.noarch                                   guava-30.1-10.el9.noarch                                 
  httpcomponents-client-4.5.13-7.el9.noarch                          httpcomponents-core-4.4.13-11.el9.noarch                 
  jakarta-annotations-1.3.5-16.el9.noarch                            jansi-2.3.3-9.el9.x86_64                                 
  java-17-openjdk-1:17.0.20.0.8-1.2.el9.x86_64                       java-17-openjdk-devel-1:17.0.20.0.8-1.2.el9.x86_64       
  javapackages-tools-6.4.0-1.el9.noarch                              jcl-over-slf4j-1.7.30-16.el9.noarch                      
  jsoup-1.13.1-13.el9.noarch                                         jsr-305-3.0.2-9.el9.noarch                               
  maven-1:3.6.3-23.el9.noarch                                        maven-lib-1:3.6.3-23.el9.noarch                          
  maven-openjdk17-1:3.6.3-23.el9.noarch                              maven-resolver-1:1.6.1-14.el9.noarch                     
  maven-shared-utils-3.3.4-7.el9.noarch                              maven-wagon-3.4.2-10.el9.noarch                          
  plexus-cipher-1.7-30.el9.noarch                                    plexus-classworlds-2.6.0-14.el9.noarch                   
  plexus-containers-component-annotations-2.1.0-13.el9.noarch        plexus-interpolation-1.26-15.el9.noarch                  
  plexus-sec-dispatcher-1.4-41.el9.noarch                            plexus-utils-3.3.0-14.el9.noarch                         
  publicsuffix-list-20210518-3.el9.noarch                            sisu-1:0.3.4-14.el9.noarch                               
  slf4j-1.7.30-16.el9.noarch                                        

Complete!
```