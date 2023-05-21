---
title: "Getting started with Kubectl"
teaching: 10
exercises: 0
questions:
- "What is Kubectl?"
- "How to use Kubectl commands?"
- "What is Argo workflows?"

objectives:
- "Learn what the kubectl command can do"
- "Appreciate the necessity for the Argo workflows tool (or similar)"
- "Learn how to set up different services/resources to get the most of your cluster"

keypoints:
- "`kubectl` is the ruler of GKE"
- "Argo is a very useful tool for running workflows and parallel jobs"
- "To be able to write, read and extract data, a few services/resources need to be set up on the GCP"
---

## K8s - Imperative vs Declarative programming
In the context of Kubernetes, imperative and declarative are two different paradigms used to define and manage the desired state of resources within a cluster. While imperative commands are useful for ad-hoc tasks or interactive exploration, the declarative approach is more suitable for managing and maintaining resources in a Kubernetes cluster efficiently. Let's explore each approach! But first, we need a tool to interact with our cluster.

<img src="https://ucarecdn.com/024d46aa-b898-4713-b3a7-2c095c94b00e/" width="400">

### Kubectl
The kubectl command-line tool is a powerful utility provided by Kubernetes that allows you to interact with and manage Kubernetes clusters. Use the following syntax to run kubectl commands from your terminal window:

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

Where:
* **command:** Specifies the operation you want to perform on one or more Kubernetes resources. Some commonly used commands include `create`, `get`, `describe`, `delete`, `apply`, and `scale`. Each command has its own set of options and subcommands.
* **TYPE:** Indicates the type of Kubernetes resource you want to interact with. It can be a single resource type like `Pod, Deployment, Service`, or a more general term like `all`, `nodes`, `namespaces`, etc.
* **NAME:** Specifies the name of the specific resource you want to operate on.
* **flags:** These are optional flags that modify the behavior of the command. Flags can be used to specify additional parameters, control output formats, apply labels, set resource limits, etc. Flags are specific to each command and can be listed by running `kubectl [command] --help`.

> Check out the kubectl [cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).
{: .testimonial}

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


{% include links.md %}
