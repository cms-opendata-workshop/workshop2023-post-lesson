---
title: "Getting started with Argo and Kubectl"
teaching: 10
exercises: 0
questions:
- "What is Kubectl?"
- "How to use Kubectl commands?"
- "What is Argo workflows?"

objectives:
- "Lear what the kubectl command can do"
- "Appreciate the necessity for the Argo workflows tool (or similar)"
- "Lear how to set up different services/resources to get the most of your cluster"

keypoints:
- "`kubectl` is the ruler of GKE"
- "Argo is a very useful tool for running workflows and parallel jobs"
- "To be able to write, read and extract data, a few services/resources need to be set up on the GCP"
---

## K8s API
The Kubernetes API (Application Programming Interface) is a set of rules and protocols that allows users and external systems to interact with a Kubernetes cluster. It serves as the primary interface for managing and controlling various aspects of the cluster, including deploying applications, managing resources, and monitoring the cluster's state.

The Kubernetes API provides a declarative model, where users can define the desired state of the cluster and the API server handles the necessary actions to achieve that state. Users can interact with the API using various methods, such as command-line tools (e.g., kubectl), programming languages (e.g., Python, Go), or through user interfaces built on top of the API.

<img src="https://collabnix.com/wp-content/uploads/2022/01/Screen-Shot-2022-01-22-at-10.52.54-AM.png" width="400">

### Kubectl

The kubectl command-line tool is a powerful utility provided by Kubernetes that allows you to interact with and manage Kubernetes clusters. Use the following syntax to run kubectl commands from your terminal window:

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

Where:
* **command:** Specifies the operation you want to perform on one or more Kubernetes resources. Some commonly used commands include `create, get, describe, delete, apply, and scale`. Each command has its own set of options and subcommands.
* **TYPE:** Indicates the type of Kubernetes resource you want to interact with. It can be a single resource type like `Pod, Deployment, Service`, or a more general term like `all, nodes, namespaces`, etc.
* **NAME:** Specifies the name of the specific resource you want to operate on.
* **flags:** These are optional flags that modify the behavior of the command. Flags can be used to specify additional parameters, control output formats, apply labels, set resource limits, etc. Flags are specific to each command and can be listed by running `kubectl [command] --help`.

> See the kubectl [installation instructions](https://kubernetes.io/docs/tasks/tools/#kubectl).
> <br/>
> See the kubectl [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).
{: .testimonial}

## K8s - Imperative vs Declarative programming
In the context of Kubernetes, imperative and declarative are two different paradigms used to define and manage the desired state of resources within a cluster. While imperative commands are useful for ad-hoc tasks or interactive exploration, the declarative approach is more suitable for managing and maintaining resources in a Kubernetes cluster efficiently. Let's explore each approach!

<img src="https://ucarecdn.com/024d46aa-b898-4713-b3a7-2c095c94b00e/" width="400">

### Imperative Approach
In the imperative approach, you specify the exact sequence of commands or actions to be performed to create or modify Kubernetes resources. You interact with the Kubernetes API by issuing explicit instructions.

Let’s first create a node running Nginx by using the imperative way.

##### Create the pod using the Imperative way
```bash
kubectl run mynginx --image=nginx
```

##### Get a list of running pods
```bash
kubectl get pods
```

##### Get more info
```bash
kubectl get pods -o wide
kubectl describe pod mynginx
```

##### Delete the pod
```bash
kubectl delete pod mynginx
```

### Declarative Approach
In the declarative approach, you define the desired state of Kubernetes resources in a declarative configuration file (e.g., **YAML** or JSON). Rather than specifying the steps to achieve that state, you describe the desired outcome and let Kubernetes handle the internal details.

##### Create a pod using the declarative way
Download the file:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/myapp.yaml
```

> ## YAML File
> This YAML file describes a Pod with an nginx web server container that listens on port 80 and has an environment variable set. 
> The specific behavior and functionality of the nginx web server will depend on the configuration of the nginx image used. 
> ~~~
> # myapp.yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: myapp-pod
> spec:
>   containers:
>   - name: nginx-container
>     image: nginx
>     ports:
>     - containerPort: 80
>     env:
>     - name: DBCON
>       value: myconnectionstring
> ~~~
> {: .language-yaml}
{: .solution}

Now, let's create a node using the YAML file
```bash
kubectl create -f myapp.yaml
```

Get some info
```bash
kubectl get pods -o wide
kubectl describe pod myapp-pod
```

Attach our terminal
```bash
kubectl exec -it myapp-pod -- bash
```

Print the DBCON environment variable that was set in the YAML file.
```bash
echo $DBCON
```

Detach from the instance
```bash
exit
```

Cleanup
```bash
kubectl delete -f myapp.yaml
```

The declarative approach is the recommended way to manage resources in Kubernetes. It promotes consistency, reproducibility, and automation. You can easily version control the configuration files, track changes over time, and collaborate with team members more effectively.

## Let's run a few examples.

* Get the status of the nodes in your cluster:

```bash
kubectl get nodes  
```

* Get the cluster info:

```bash
kubectl cluster-info  # Display addresses of the master and services
```

Let's list some kubernetes components:

* Check pods

```bash
kubectl get pod
```

* Check the services

```bash
kubectl get services
```

We don't have much going on.  Let's create some components.

* Inspect the `create` operation

```bash
kubectl create -h
```

Note there is no `pod` on the list, so in K8s you don't create pods but *deployments*.  These will create pods, which will run under the hood.

* Let's create an application, it does not matter which.  Let's go for `nginx`:

```bash
kubectl create deployment mynginx-depl --image=nginx
```

The `nginx` image will be pulled down from the Docker Hub.
This is the most minimalist way of creating a deployment.

* Check the deployments

```bash
kubectl get deployment
```

* Check pods

```bash
kubectl get pod
```

## Argo

Argo is a collection of open source tools that let us to extend the functions in Kubernetes. We can find some benefits from use argo.
- Cloud agnostic service
- Argo can execute on absolutely in all clusters in kubernetes.
- We can review the state of resources constantly.
- There is a large capacity of executing jobs at the same time and from different nodes.
- It’s possible correct debug errors.
We are going to explain 3 of the tools most important for working with argo.

### Argo as a workflow engine

- Is used to execute complex job orchestration, including serial and parallel execution where each stage is executed like a container.
- It is the most popular workflow execution engine for kubernetes.
- You can run thousands of workflows a day, each with thousands of concurrent tasks.
- Designed from the ground up for containers without the overhead and limitations of legacy VM and server-based environments. 

While jobs can also be run manually, a workflow engine makes defining and
submitting jobs easier. In this tutorial, we use
[argo](https://argoproj.github.io/argo/quick-start/).
Install it into your working environment with the following commands
(all commands to be entered into the cloud shell):

While jobs can also be run manually, a workflow engine makes defining and submitting jobs easier. In this tutorial, we use [argo quick start](https://argoproj.github.io/argo-workflows/quick-start/) page to install it.

## Namespaces

Namespaces are a kind of reservations in your K8s cluster.  Let's create one for the Argo workflow we will user

```bash
kubectl create ns <NAMESPACE>
```

> ## Kubernetes namespaces
>
> The above commands as well as most of the following use a flag `-n argo`,
> which defines the namespace in which the resources are queried or created.
> [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
> separate resources in the cluster, effectively giving you multiple virtual
> clusters within a cluster.
>
> You can change the default namespace to `argo` as follows:
>
> ```bash
> kubectl config set-context --current --namespace=argo
> ```
>
{: .testimonial}

{% include links.md %}
