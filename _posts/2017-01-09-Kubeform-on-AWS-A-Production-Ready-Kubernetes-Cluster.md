---
author: proietti
comments:
date: 2017-01-30 09:00:00+00:00
layout: post
slug: a-introduction-to-coreos
title: An introduction to CoreOS
wordpress_id:
categories: Cloud
tags:
- Docker
- Linux
- CoreOS
---
<div class="message">

With this document I want  to present a tool for provisioning an "almost  production ready"  <a href="https://kubernetes.io">Kubernetes</a> clusters to any cloud with security, scalability and maintainability in mind. The "Kubeform"    <a href="https://github.com/Capgemini/kubeform">Kubeform</a> tool has been outsourced by Cap Gemini Cloud Group and recently I forked it to  study it.
</div>
"<!-- more -->"

<p>As of today, Kubeform comes with support for <a href="https://aws.amazon.com/">AWS</a>, <a href="https://www.digitalocean.com/">DigitalOcean</a> and local clusters via <a href="https://www.docker.com/products/docker-compose">Docker Compose</a> but in this document I focused on AWS support part and I 've been upgraded the public cloud solution (i.e. managed by an external client)  to  Terraform 0.7 release. 

<p>Kubeform leverages <a href="https://terraform.io">Terraform</a>, <a href="https://ansible.com">Ansible</a> and <a href="https://coreos.com">CoreOS</a> as the basic building blocks for your Kubernetes clusters.

<h2 id="features">Features</h2>

<p>Out of the box the tool configure Kubernetes in HA mode with 3 master API servers by default and a configurable number of worker nodes (which can be configured via a terraform variable). We also provide “edge router nodes” (again configurable) used for ingress load balancing.</p>

<p>The AWS setup closely follows the <a href="https://coreos.com/kubernetes/docs/latest/">CoreOS guide for Kubernetes on AWS</a> with all elements secured with <a href="https://github.com/Capgemini/tf_tls">TLS certificates using tf_tls</a>.</p>

<p>The tool set up the edge router nodes with <a href="https://traefik.github.io/">Traefik</a> as an ingress controller by default. </p>

<p>SkyDNS is enabled by default and the <a href="https://github.com/kubernetes/dashboard">Kubernetes Dashboard</a> project is turned on as well, allowing an operator to view the state of the cluster through a nice web UI.</p>

<p>there is also an additional support for <a href="https://github.com/helm/helm">Helm</a> which can be enabled to provide the <a href="https://github.com/deis/workflow">Deis Workflow</a> by default.</p>

<p>As previously said the Cluster is "almost ready for production" due to the Worker mangement whiich does not neither allow them to scale using the terraform features (TO BE VERIFIED) nor to  belong to an AWS Autoscaling Group.<p>

<p> Also the ingress controller is not in HA TBV<p>
