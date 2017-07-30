---
layout: post
title: Kubernetes : Integrations on AWS
tags:
- Kubernetes
- DevOps
categories:
- Cloud
---



My first 5 month of Kubernetes.  

In this period  I 've been interested in installing and configuring Kubernetes cluster using different tools and architecture.

At the same time , of course , I 've looked around on Github to find the most complete and updated open source implementation .


These are my considerations on what is happening on the stage and what are the most trending way to install Kubernetes on AWS.
</div>
"<!-- more -->"
First of all,  it should be said that , apart same spcific implemtations on bare metal , the most part of open source tools to install Kubernetes   aim to be used on public cloud such as Amazon Web Services . Then it must be chosen the OS hosting the system , here we have to decide  if running the processes (kubelet ) hosted on server or in a container. Finally  you have to choose to use an external deployment tool or use the features integrated in the hosting OS to deploy the application. 

So you can range from a bash based  installation system such as kube.up (recently deprecated) up to highly intgrated to AWS Kube-aws by Coreos.

So you can use shell commands to install kubernets everywhere or ansible playbooks to do the same, but in the last times the trend is to use tools integrated with Aws such as Terraform or CloudFormation to provision and  Systemd/Cloud-config to deploy the kubernetes components already containerized.

Coreos is most advanced  OS on that way  infact its usage  implies a set of available features and tools which make easier to install Kubenertnes  but at the same time are so specific to lock-in the project. It must also be said that the OS is quit young and threfore not so mature and lacks of some legacy features.

In this blog I want to deeply analyze some possible implementations on AWS using Coreos. 

Kubernetes is composed of different components : Masters , Workers ; Master(s) components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and detecting and responding to cluster events (starting up a new pod when a replication controller’s ‘replicas’ field is unsatisfied).

Master components can be run on any node in the cluster. However, for simplicity, set up scripts typically start all master components on the same AWS instance, and do not run user containers (or better Pod) on this intance. 

### Master components are : 

* Kube-scheduler,
* Kube-controller,
* Kube-apiserver,
* Addons,
* Etcd,
* Cloud-controller,
* Kubelet.

 The last three components need just a little explication from the DevOps point of view : 

### etcd subsystem
etcd (composed of at least three clustered processes)  is not a functional kubernetes element but is required to store  all cluester's data (both cluster's and user entities' data) and up to now all the main implementations deploy etcd on the same instance of Master components. It shuld be considere that etcd and kubernetes cycle releases' are not mandatory the same so in the last times the community begins to evaluate if splitting etcd and master is worthful ( in order gain operability on both software packages). It must be said that  migrate an etcd system or only to re-run an etcd process is not an easy task so up to now   etcd subsystem (and therefore Master node ) are not involved in any devops type automatism ...

### Cloud-controller
It is an Alpha stage feature in kubernetes 1.6 (implemented pheraps from Rancher) in which will be collected all the kubernetes (kube-controller) 's  integration with cloud vendor features

### Kubelet
It is not a functional kubenetes Master's element ( instead it  is a node elements) but it is used  by many systemd based OS to start others Master components thorugh the manifests concept ( as Pod in system Namespace) . 

Kube-scheduler, Kube-controller, Kube-apiserver belong to Kubernetes's core ( see official doc)

### Addons
They (as the name says) are addon supported (as Pod in system Namespace) by kubernetes development team. the functions they caries up  spread from monitoring, Ui (dashboard for example), DNS, logging systems    ... up to ways to integrate cloud features in pod management (Autoscaler for example ... that is confusing eh?)

 

### Workers components are : 


### kubelet
It is the primary node agent. It watches for pods that have been assigned to its node (either by apiserver from commands from kubecl  for example  or via local configuration file at boot phase for example) and:

   * Mounts the pod’s required volumes.
   * Downloads the pod’s secrets.
   * Runs the pod’s containers via docker (or, experimentally, rkt).
   * Periodically executes any requested container liveness probes.
   * Reports the status of the pod back to the rest of the system, by creating a mirror pod if necessary.
   * Reports the status of the node back to the rest of the system.

### kube-proxy
It enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding.

### docker or rkt
They  are used for running containers.

### fluentd
It is a daemon which helps cluster level logging (addons agent)

### What are the enhancements offered by Coreos about the these component's installations ?

* Etcd3 is embedded in OS and and an etcd-member systemd service is provide the configurations of peers and clients/proxies configuration.
* Kubelet is a systemd service wrapped by kubelet-wrapper to hide the different releases parameter changes.

* Flannel is network software defined networking used for Pos is a Systemd Service.

* Rkt is , of course ,  included in the OS (through a wrapper also)

### What are AWS features can be integrated in Kubernaetes deployments ?

First of all it must be said tha tools can be divided in 'anywhere'  or any cloud tools and those focused on AWS , this implies a different degree of integration with  Aws features.

### VPC for networking
Almost all the open source tools support VPC ec2 instances . None of them (KubeSpray , Tack, Kops , Kubeform , kube.up, TerraKube...) supports  more than cluster in a VPC neither to parametrized the VPC name. Some of them ( tack) can start a cluster in an already existent vpc.

### AZ  for networking 
Availability Zones can be used for an HA configuration for Master/etcd and workers Most of the forementioned tools supports Master/etcd spread over multi-AZ . (some of them i.e. Tack does not)


All the  software reliability enhancement base on ASG (Autoscaling Group Aws)  are based on  internal instances deployment tools aka systemd aka Coreos, it is much more difficult to implememts with external tools like Ansible for example...)

### ASG can be used for  etcd and/or Master
Having already stated that none of tools split etcd and master on different instances , Master can be easily integrated in ASG (in  effect Api server is stateless and Controller and scheduler operates in Hot/standby mode through an Api-lease-lock (using etcd lock feature) and a watch-dog processus)  but etcd cluster and its raft protocol elements are strictly stateful so up to now very few tools use Etcd in ASG .
Recently there are two different approaches to put  etcd in ASG to achieve HA  (experimental from my point of view):
Smilodon (by UKHome Office DP dept.) based solutions  : each time a instance starts in  etcd ASG gets an ebs volume (already used by a previous etcd instances ) and  configures an ENI (already seen as belonging to the cluster) so the new instances is seen the old ones (remember that all etcd instances not belonging to the initial  cluster are seen as etcd-proxy).This solution assumes that there is only one etcd instance in a AZ (ie. only one etcd  ebs for AZ to avoid conflict). 
Monsanto Based solutions : first you run instance  in an ASG and then  collect the ip of that instances and make  the initial clusster string , each time that an etcd instance dies (or is killed) the new born instance understand if is joining a existent cluster or not and if is thecase remove (throgh Rest etcd Api ) the died  entity and than join (throgh Api) the cluster . the solution could be complemented with ASG life cycle hooks and some lambda function to store the events the died  etcd instance. (more to work...
In my opinion Smilodon based solutions are easier and more robust but at the moment suffers of Coreos weakness for advanced networking .... the additional ENI (with fixed ip belonging to  the cluster ) are in different subnet (in same AZ) of primary interface or just in the same subnet ( risks of  asymmetric routing without specific configurations).

### ASG can be used for master   to achieve Orizontal scaling
not yet implemnted but useful for average cluster ? anway only api server can scale.



### ASG can be used for workers to  achieve HA 
Easy to implments with Coreos and also with Ansible based solutions using kube-adm
### ASG can be used for workers Orizontal Scaling
Not  so useful due the fact the atomic item to monitor is the Service/deployment not the instance anyway there is kubernetes' Horizontal Autoscaler which uses ASG behind the scene. This is one of devops concept brings in  product source..


### ELB can used for Api server (Master front-end)
In effect it should be used more than elb system on the same instances set : an internal elb for api server request from node (kubelet) and an external elb for kubectl commands.. Some tools ( Terrakube) uses only one elb (external) to Api server.

### ALB  can be used for ingress
Not yet implemented

### DNS for internal services (etcd / Master)
Some tools (tack)  uses Aws R53 internal hosted Zones to identify and load balancing traffic to etcd and Api server. Other tools (Kops) use externl Hosted Zones  and registered domains.

### S3 for Store for cluster secrets
Really required for embedded  (in instance ) deployment tool (aka coreos) or old shell based tools (kube.up) not really required for ansible deployments or terraform deployments.


### NAT/Bastion for management
useful and required for local management or updating  os elements or container. NAT Gateway very expensive on Aws most of time  a NAt instance suffices..

### VPS for management
Not implemented in Open Source tools ( not enterprise oriented)


### EBS/ENI for persistency
See Smilodon or Master/etcd persistency
