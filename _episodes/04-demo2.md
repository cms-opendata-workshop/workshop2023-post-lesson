---
title: "Demo: Storing workflow output on Google Kubernetes Engine"
teaching: 5
exercises: 30
questions:
- "How to setup a workflow engine to submit jobs?"
- "How to run a simple job?"
- "How can I set up shared storage for my workflows?"
- "How to run a simple job and get the the ouput?"
objectives:
- "Understand how to run a simple workflows in a commercial cloud environment"
- "Understand how to set up shared storage and use it in a workflow"
keypoints:
- "With Kubernetes one can run workflows similar to a batch system"
- "Open Data workflows can be run in a commercial cloud environment using modern tools"
---

> ## Minikube
> 
> Keep in mind that [minikube](https://minikube.sigs.k8s.io/docs/start/) quickly sets up a single 
> local Kubernetes cluster on macOS, Linux, and Windows. 
> It is not possible to fetch yaml configuration files from your terminal, so omit the ```wget``` 
> command and insert the following urls in your browser of preference and create a yaml file with 
> its content to be able to ```kubectl apply -n argo -f <example.yaml>```.
> 
{: .testimonial}

## Install argo as a workflow engine

While jobs can also be run manually, a workflow engine makes defining and
submitting jobs easier. In this tutorial, we use
[argo](https://argoproj.github.io/argo/quick-start/).
Install it into your working environment with the following commands
(all commands to be entered into the cloud shell):


```bash
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml

# Download the binary
curl -sLO https://github.com/argoproj/argo/releases/download/v2.11.1/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
sudo mv ./argo-linux-amd64 /usr/local/bin/argo
```

This will also install the argo binary, which makes managing the workflows
easier.

> ## Reconnecting after longer time away
>
> In case you leave your computer, you might have to reconnect to the CloudShell
> again, and also on a different computer. If the `argo` command is not found,
> run the command above again starting from the `curl` command.
>
{: .callout}

You can now check that argo is available with

```bash
argo version
```

We need to apply a small patch to the default argo config. Create a file called
`patch-workflow-controller-configmap.yaml`:

~~~
data:
  artifactRepository: |
    archiveLogs: false
~~~
{: .language-yaml}

Apply:

```bash
kubectl patch configmap workflow-controller-configmap -n argo --patch "$(cat patch-workflow-controller-configmap.yaml)"
```

## Run a simple test workflow

To test the setup, run a simple test workflow with

```bash
argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
argo list -n argo
argo get -n argo @latest
argo logs -n argo @latest
```
* Wait till the yellow light turns green
* If argo was installed correctly you will have the following:
![](../fig/HelloWorld.png)

Please mind that it is important to delete your workflows once they have
completed. If you do not do this, the pods associated with the workflow
will remain scheduled in the cluster, which might lead to additional charges.
You will learn how to automatically remove them later.

```bash
argo delete -n argo @latest
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



## Storage volumes

If we run some application or workflow, we usually require a disk space where to dump our results.  There is no persistent disk by default, we have to create it.

You could create a disk clicking on the web interface above, but lets do it faster in the command line.

Create the volume (disk) we are going to use

```bash
gcloud compute disks create --size=100GB --zone=us-central1-c gce-nfs-disk-1
```

Set up an nfs server for this disk:

```bash
wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/001-nfs-server.yaml
kubectl apply -n argo -f 001-nfs-server.yaml
```

Set up a nfs service, so we can access the server:

```bash
wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/002-nfs-server-service.yaml
kubectl apply -n argo -f 002-nfs-server-service.yaml
```

Let's find out the IP of the nfs server:

```bash
kubectl get -n argo svc nfs-server |grep ClusterIP | awk '{ print $3; }'
```

Let's create a *persisten volume* out of this nfs disk.  Note that persisten volumes are not namespaced they are available to the whole cluster.

We need to write that IP number above into the appropriate place in this file:

```bash
wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/003-pv.yaml
```

~~~
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-1
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <Add IP here>
    path: "/"
~~~
{: .language-yaml}

Deploy:

```bash
kubectl apply -f 003-pv.yaml
```

Check:

```bash
kubectl get pv
```

Apps can claim persistent volumes through *persistent volume claims* (pvc).  Let's create a pvc:

```bash
wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/003-pvc.yaml
kubectl apply -n argo -f 003-pvc.yaml
```
Check:

```bash
kubectl get pvc -n argo
```

Now an argo workflow coul claim and access this volume with a configuration like:

~~~
# argo-wf-volume.yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: test-hostpath-
spec:
  entrypoint: test-hostpath
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: nfs-<NUMBER>
  templates:
  - name: test-hostpath
    script:
      image: alpine:latest
      command: [sh]
      source: |
        echo "This is the ouput" > /mnt/vol/test.txt
        echo ls -l /mnt/vol: `ls -l /mnt/vol`
      volumeMounts:
      - name: task-pv-storage
        mountPath: /mnt/vol
~~~
{: .language-yaml}

Submit and check this workflow with

```bash
argo submit -n argo argo-wf-volume.yaml
argo list -n argo
```

Take the name of the workflow from the output (replace XXXXX in the following command)
and check the logs:

```bash
kubectl logs pod/test-hostpath-XXXXX  -n argo main
```

Once the job is done, you will see something like:

~~~
ls -l /mnt/vol: total 20 drwx------ 2 root root 16384 Sep 22 08:36 lost+found -rw-r--r-- 1 root root 18 Sep 22 08:36 test.txt
~~~
{: .output}

## Get the output file

The example job above produced a text file as an output. It resides in the persistent volume that the workflow job has created.
To copy the file from that volume to the cloud shell, we will define a container, a "storage pod" and mount the volume there so that we can get access to it.

Create a file `pv-pod.yaml` with the following contents:

~~~
# pv-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: nfs-<NUMBER>
  containers:
    - name: pv-container
      image: busybox
      command: ["tail", "-f", "/dev/null"]
      volumeMounts:
        - mountPath: /mnt/data
          name: task-pv-storage
~~~
{: .language-yaml}

Create the storage pod and copy the files from there

```bash
kubectl apply -f pv-pod.yaml -n argo
kubectl cp  pv-pod:/mnt/data /tmp/poddata -n argo
```

and you will get the file created by the job in `/tmp/poddata/test.txt`.
