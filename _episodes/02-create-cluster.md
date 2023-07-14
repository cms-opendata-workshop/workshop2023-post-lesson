---
title: "Creating a K8s cluster"
teaching: 5
exercises: 20
questions:
- "How do I manually create a K8s cluster on the GCP"
objectives:
- "Learn how to create a K8s cluster from scratch on GKE"
keypoints:
- "It takes just a few clicks to create you own K8s cluster"
---

## Introduction
In this demonstration we will show you the very basic way in which you can create a computer cluster (a Kubernetes cluster to be exact) in the cloud so you can do some data processing and analysis using those resources.  In the process we will make sure you learn about the jargon.  During the hands-on session of the workshop (cloud computing), a cluster similar to this one will be provided to you for the exercises.  


## Creating your own cluster on GKE
For the hands-on part of this lesson you will not have to create the cluster for yourself, it will be already done for you.  For pedagogical reasons, however, we will show an example of how to do it by hand. The settings below should be good and cheap enough for CMSSW-related workflows.

> Here you will find some [instructions on how to use preemptible machines](https://cloud.google.com/kubernetes-engine/docs/how-to/preemptible-vms#create_a_cluster_or_node_pool_with_preemptible_vms) in GKE.  Preemptible VMs offer similar functionality to Spot VMs, but only last up to 24 hours after creation, this might be taken into account if reducing costs is of your concern.
{: .testimonial}

* Get to the Console
* Create a new project or select one (if you already have one). Note that you will not be able to create clusters in the GCP project that we used for the workshop.
* Click on the Kubernetes engine/clusters section on the left side menu
![](../fig/SelectCluster.png)
* Click Create
![](../fig/SelectCreate.png)
* Select Standard (not autopilot)
![](../fig/SelectGKE.png)
* Give it a name
* Change the zone to e.g. ```europe-west1-b``` (for data streaming speed, choose a [zone](https://cloud.google.com/compute/docs/regions-zones) close to CERN)
![](../fig/BasicClusterName.png)
* Many ways to configure the cluster, but let's try an efficient one with autoscaling
* Go to "Default pool"
* Choose size: 1 node
* Autoscaling 0 to 4
  ![](../fig/ClusterDetails.png)
* Go to "Nodes"
* Choose a machine e2-standard-4
* Check "Enable spot VMs" down in the menu
* Leave the rest as it is
* Hit create
![](../fig/MachineConfig.png)
* Creation will take a while
While we wait, let us inspect the Cloud shell...
## Cloud shell
GCP provides an *access machine* so you can interact with their different services, including our newly created K8s cluster.  This machine (and the terminal) is not really part of the cluster. As was said, it is an entry point. From here you could *connect* to your cluster.
## The `gcloud` command
The gcloud command-line interface is the primary CLI tool to create and manage Google Cloud resources. You can use this tool to perform many common platform tasks either from the command line or in scripts and other automations.
## Connect to your cluster
Once the cluster is ready (green check-mark should appear)
* Click on the connect button of your cluster:
![](../fig/Connection4.png)
* Execute that command in the cloud shell:
![](../fig/Commandline.png)
* You’ve connected to your shell, now press enter to link to your GKE cluster:
![](../fig/Load.png)
* Authorize Shell
![](../fig/SomeCredentials.png)

{% include links.md %}

