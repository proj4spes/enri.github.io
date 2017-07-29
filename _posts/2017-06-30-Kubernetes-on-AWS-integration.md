
---
layout: post
title: Come installare Kubernetes su AWS
tags:
- Kubernetes
- DevOps
categories:
- Cloud
---


<div class="message">

 My first 5 month of Kubernetes 

 In this period  I 've been interested in installing and configuring Kubernetes cluster using different tools and architecture.

At the same time , of course , I 've looked around on Github to find the most complete and updated open source implementation .


These are my considerations on what is happening on the stage and what are the most trending way to install Kubernetes.
</div>
"<!-- more -->"
First of all,  it should be said that , apart same spcific implemtations on bare metal , the most part of open source tools to install Kubernetes   aims to be used on public cloud such as Amazon Web Services . Then it must be chosen the Os hosting the system , here we have to decide  if running the processes (kubelet ) hosted on server or in a container. Finally  you have to choose to use an external deployment tool or use the features integrated in the hosting Os to deploy the application. 

So you can range from a bash based  installation system such as kube.up (recently deprecated) up to highly intgrated to AWS Kube-aws by Coreos.

So you can use shell command to install kubernets everywhere or ansible playbook to do the same, but in the last times the trend is to use tools integrated with Aws such as Terraform or CloudFormation to provision and  Systemd/Cloud-config to deploy the kubernetes components already containerized. Coreos is most advanced  Os on that way.....
