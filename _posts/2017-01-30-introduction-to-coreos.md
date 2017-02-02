---
author: proietti
comments: 
date: 2017-01-30 09:00:00+00:00
layout: post
title: An introduction to CoreOS
Categories: Cloud
tags:
- Docker
- Linux
- CoreOS
---

<div class="message">
In this post, I'm going to provide a  quick introduction to [CoreOS](http://www.coreos.com/). CoreOS, in case you haven't heard of it, is a highly streamlined Linux distribution designed with containers, massive server deployments, and distributed systems/applications in mind.

</div>
"<!-- more -->"

CoreOS is built around a number of key concepts/technologies:

  1. _The OS is updated as a whole, not package-by-package._ CoreOS  employs an active/passive dual root partition scheme. This dual root partition scheme allows CoreOS to run off one root partition while updating the other; the system then reboots onto the updated partition once an update is complete. If the system fails to boot from the updated partition, then reboot it again and it will revert to the known-good installation on the first partition.

  2. _All applications run  mainly in containers (i.e. Cgroups)._ CoreOS provides out-of-the-box support for [Docker containers](https://www.docker.com/). In fact, all applications on CoreOS run in containers. This enables separation of applications from the underlying OS and further streamlines the CoreOS update process (because applications are essentially self-contained).

  3. _CoreOS is based on systemd._ [systemd](http://freedesktop.org/wiki/Software/systemd/) is not unique to CoreOS; it is the new standard system and service manager for Linux. (Debian has elected to use systemd; Ubuntu has adopted systemd with 16.x; and Red Hat and related distributions already use systemd in rel7.x.) In CoreOS, systemd unit files are used not only for system services, but also for running Docker containers and others basic services.

  4. _CoreOS expands cloudinit   with specific verbs to configure etcd.services. This features make it the preferred os for cloud deployments.  

  5. _CoreOS has a distributed key-value data store called etcd._ The etcd distributed key-value data store can be used for shared configuration and service discovery in distributed applications. Etcd uses a simple REST API (HTTP+JSON) and leverages the [Raft consensus protocol](http://raftconsensus.github.io/). Docker containers on CoreOS are able to access etcd via the loopback interface, and thus can use etcd to do dynamic service registration or discovery, for example. etcd is also configurable via cloud-init, which means it's friendly to deployment on many cloud platforms including [OpenStack](https://coreos.com/docs/running-coreos/platforms/openstack/) and [AWS] (https://coreos.com/docs/running-coreos/platforms/AWS/). More information on etcd is available via [the etcd GitHub site.](https://github.com/coreos/etcd/)

  6. _CoreOS supported deploying containers across a cluster using fleet._ [Fleet](https://github.com/coreos/fleet/) is another open source project that leverages etcd to deploy Applications (written as systemd unit files) across a cluster of CoreOS systems. Fleet leverages both etcd and systemd to support the deployment of containers across a cluster of systems. See [this page](https://coreos.com/using-coreos/clustering/) for more information on clustering with CoreOS and fleet. BUT very important CoreOs is now supporting Kubernetes orchestrator as first citizen. This means that fleet implementations will be translated in Kubernetes pods and leverage kubernetes infrastructure. 

 7. _CoreOS is promoting the RKT containers format to improve security and composability within linux container.


Taken individually---the use of a minimal Linux distribution, systemd support, the distributed key-value data store, Docker support, dual root partition w/ recoverable system updates, fleet---these technologies are interesting, but not all that revolutionary. Put them all together, however, and you have a very interesting solution.


