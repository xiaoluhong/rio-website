---
title: "Installation"
weight: 1
---

## Quick Start

Download: Click one of the links above

Prerequisites: If you want to run on your laptop, then Minikube or Docker for Mac/Windows is recommended.  If you
don't have those then you need to run a Linux VM (or Linux itself, come to the darkside).  We will make this easier in the future.
Otherwise you can run this easily with any modern Linux server, nothing is needed to be installed except the kernel.

Run: `rio ps` and it will tell you what to do

## Installation Types

Rio will run in two different modes:

**Rio Standalone**: In this mode Rio comes will all the container tech you need built in. All you need are modern Linux
servers.  Rio does not need Docker, Kubernetes or anything else installed on the host.  This mode is good if you want
to run containers but don't want to be a Kubernetes expert.  Rio will ensure that you have the most secure setup and
keep all the components up to date.  This is by far the easiest way to run clusters.

**Run on Kubernetes**: In this mode Rio will use an existing Kubernetes cluster.  The advantages of this approach is
that you get more flexibility in terms of networking, storage, and other components at the cost of greatly increased
complexity.  For the time being this mode is also good for your laptop as Minikube and Docker For Mac/Windows both
provide a simple way to run Kubernetes on your laptop.  In the future Rio will have a mode that is simpler and does not
require Docker For Mac/Windows or Minikube.

### Standalone

Standalone requires Linux 4.x+ that support overlay, squashfs, and containers in general.
This will be most current distributions.

Rio forms a cluster.  To create the cluster you need one or more servers and one or more agents.  Right now HA is in the works still
so only one server is supported.  To start a server run

    sudo rio server

That will start the server and register the current host as a node in the cluster.  At this point you have a full single node
cluster.  If you don't wish to use the current server as a node then run

    rio server --disable-agent

This mode does have the benefit of not requiring root privileges.  On startup the server will print something similar as below

    ```
    INFO[0005] To use CLI: rio login -s https://10.20.0.3:7443 -t R108527fc31eb165d69e4ebb048168769d97734707dc22bd197b5ae2fcab27d9e64::admin:fb5ef140c22562de2789168ac6973bda 
    INFO[0005] To join node to cluster: rio agent -s https://10.20.0.3:7443 -t R108527fc31eb165d69e4ebb048168769d97734707dc22bd197b5ae2fcab27d9e64::node:9cb35d8ae4a4621abdacfa6d8d1ea1b6 

    ```

Use those two command to either access the server from the CLI or add another node to the cluster.  If you are root
on the host that is running the Rio server, `rio login` is not required.

The state of the server will be in `/var/lib/rancher/rio/server` or `${HOME}/.rancher/rio/server` if running as non-root.
For more robust HA setups that state can be moved to MySQL or etcd (this is still in the works).  The state of the agent
will be in `/var/lib/rancher/rio/agent`.

### On Kubernetes

If you wish to run on an existing Kubernetes cluster all that is requires is that you have a working `kubectl` setup.  Then
just run

    rio login

Follow the onscreen prompts and Rio will try to install itself into the current `kubectl` cluster.  Please note `cluster-admin`
privileges are required for Rio.  This will probably changes, but for now we need the world.

