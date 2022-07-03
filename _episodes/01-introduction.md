--- 
title: "Introduction" 
teaching: 10 
exercises: 0 
questions:
- "What is Kubernetes?" 
- "What is a Kubernetes cluster and why do I need one?" 
- "What is Argo?" 
objectives:
- "Learn the very basics of Kubernetes" 
- "Learn a bit about the architecture of a Kubernetes cluster" 
- "Learn the very basics of Argo"
 keypoints: 
- "Kubernetes is an orchestrator of containers. It is most useful when it is run in a cluster of computers." 
- "Commercial K8s clusters are a good option for large computing needs." 
- "We can run our containerized CMSSW jobs and subsequent analysis workflows in a K8s cluster." 
--- 
## Introduction 
Kubernetes is a container orchestration system open sourced by Google. Its main purpose is to schedule services to run on a cluster of computers while abstracting away the existence of the cluster from the services. Kubernetes is now maintained by the Cloud Native Computing Foundation, which is a part of the Linux Foundation. Kubernetes can flexibly handle replication, impose resource limits, and recover quickly from failures. Most of you have been working with Docker containers throughout this workshop. As you know now, one could see these Docker containers like isolated machines that can run on a host machine in a very efficient way. Your desktop or laptop could actually run several of these containers if enough resources are available. For instance, you could think that one can maximize the hardware resources by running multiple CMSSW open data containers in one desktop. Imagine, if you could run 10 CMSSW containers (each over a single root file) at the same time in a single physical machine, then it will take you 10 times less time to skim a full dataset. Now, if you had more machines available, let's say 3 or 4, and in each one of them you could run 10 containers, that will certainly speed up the data processing (specially the first stage in our analysis flow example, which we will see later) Now, if you could have access to many more machines, then a few problems may appear. For instance, how would you install the software required in all those machines? Would there be enough personpower to take care of that? How would you take care of, and babysit all those containers?
## Kubernetes (K8s)
 A Kubernetes cluster consists of "master" nodes and "worker" nodes. In short, master nodes share state to manage the cluster and schedule jobs to run on workers. It is considered best practice to run an odd number of masters. 
### Kubernetes Components
When you deploy Kubernetes, you get a cluster. A Kubernetes cluster consists of a set of worker machines, called nodes. The worker nodes host the Pods that are the components of the application workload. 
* Masters Kubernetes masters share state via etcd, a distributed key-value store (KVS) implementing the Raft protocol. Do note that the state stored in etcd is scheduling state, service locations, and other cluster metadata; it does not keep state for the services running on the cluster. 
* Workers While master nodes are constantly sharing data, managing the control plane (routing inside the Kubernetes cluster), and scheduling services, workers primarily run pods. ![](https://1.bp.blogspot.com/-kCijQkEkmA8/X9ctU83lcJI/AAAAAAAAF5U/GayBI9yQ-PsUuGI9L4Mf8dJwsByp6g8WQCLcBGAsYHQ/s1192/k8%2Barchitecture.PNG)
### Nodes Components 
* Pods In the Kubernetes world, pods are the smallest computing unit. A pod is made up of one or more containers. While pods are essential for understanding Kubernetes, when writing services we don't actually deal in pods but one further abstraction, deployments, which create pods for us| 
* Kubelet The kubelet is the primary "node agent" that runs on each node.The kubelet takes a set of PodSpecs and ensures that the containers described in those PodSpecs are running and healthy. 
* Kube-Proxy It reflects the services defined in the cluster and manages the rules to load-balance requests to a service’s backend pods. ![](https://res.cloudinary.com/escalante-rep/image/upload/v1589159144/i14yfj2jn5nm70bzekxu.jpg)
### Autoscaling
Kubernetes supports autoscaling to optimise your nodes’ resources as wll as adjust CPU and memory to meet your application’s real usage. When you need to save some money, you can scale down. Probably you want to pay for what you use, keep only with the resources when you need them If you want to learn about pricing, ckeck the next link: [Google-cloud](https://cloud.google.com/compute/all-pricing)
