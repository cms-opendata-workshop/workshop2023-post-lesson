---
title: "Storing a workflow output on Kubernetes"
teaching: 5
exercises: 30
questions:
- "How to set up a common storage on a cloud cluster"
objectives:
- "Understand how to set up shared storage and use it in a workflow"
keypoints:
- "Persistant volume is used to store the workflow output"
---


## Kubernetes Cluster - Storage Volume

When running applications or workflows, it is often necessary to allocate disk space to store the resulting data. Kubernetes provides various options for managing storage volumes, allowing you to efficiently store and access data within your cluster.

In many applications or workflows, there is a need for disk space to store and persist results. While Google Kubernetes Engine (GKE) provides persistent disks as default storage for applications running in the cluster, local machines do not have such built-in persistent storage capabilities. To address this, we can create a persistent volume using an NFS disk, allowing us to store and access data from our local machines. It's important to note that persistent volumes are not namespaced and are available to the entire Kubernetes cluster.

Create the volume (disk) we are going to use:
```bash
gcloud compute disks create --size=100GB --zone=europe-west1-b gce-nfs-disk-1
```

Set up an nfs server for this disk:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/001-nfs-server.yaml
kubectl apply -n argo -f 001-nfs-server.yaml
```
Set up a nfs service, so we can access the server:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/002-nfs-server-service.yaml
kubectl apply -n argo -f 002-nfs-server-service.yaml
```
Let’s find out the IP of the nfs server:
```bash
kubectl get -n argo svc nfs-server | grep ClusterIP | awk '{ print $3; }'
```
Let’s create a persistent volume out of this nfs disk. Note that persistent volumes are not namespaced they are available to the whole cluster.

We need to write that IP number above into the appropriate place in this file:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/003-pv.yaml
vim 003-pv.yaml
```
Deploy:
```bash
kubectl apply -f 003-pv.yaml
```
Check:
```bash
kubectl get pv
```

Apps can claim persistent volumes through persistent volume claims (pvc). Let’s create a pvc:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/003-pvc.yaml
kubectl apply -n argo -f 003-pvc.yaml
```
Check:
```bash
kubectl get pvc -n argo
```
Now an argo workflow coul claim and access this volume with an argo Workflow, let's retrieve the configuration file with:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/argo-wf-volume.yaml
```
The file has the following content:

> ## YAML File
> ~~~
> # argo-wf-volume.yaml
> apiVersion: argoproj.io/v1alpha1
> kind: Workflow
> metadata:
>   generateName: test-hostpath-
> spec:
>   entrypoint: test-hostpath
>   volumes:
>     - name: task-pv-storage
>       persistentVolumeClaim:
>         claimName: nfs-1
>   templates:
>   - name: test-hostpath
>    script:
>       image: alpine:latest
>       command: [sh]
>       source: |
>         echo "This is the ouput" > /mnt/vol/test.txt
>         echo ls -l /mnt/vol: `ls -l /mnt/vol`
>       volumeMounts:
>       - name: task-pv-storage
>         mountPath: /mnt/vol
> ~~~
> {: .language-yaml}
{: .solution}

Submit and check this workflow with:
```bash
argo submit -n argo argo-wf-volume.yaml
argo list -n argo
```

Take the name of the workflow from the output (replace XXXXX in the following command) and check the logs:
```bash
kubectl logs pod/test-hostpath-XXXXX  -n argo main
```

Once the job is done, you will see something like:
```output
ls -l /mnt/vol: total 20 drwx------ 2 root root 16384 Sep 22 08:36 lost+found -rw-r--r-- 1 root root 18 Sep 22 08:36 test.txt
```

## Get the output file
The example job above produced a text file as an output. It resides in the persistent volume that the workflow job has created. To copy the file from that volume to the cloud shell, we will define a container, a “storage pod” and mount the volume there so that we can get access to it.

Get the pv-pod.yaml file with:
```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/pv-pod.yaml
```
This file must have the following content:

> ## YAML File
> ~~~
> # pv-pod.yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: pv-pod
> spec:
>   volumes:
>     - name: task-pv-storage
>       persistentVolumeClaim:
>         claimName: nfs-1
>   containers:
>   - name: pv-container
>     image: busybox
>     command: ["tail", "-f", "/dev/null"]
>     volumeMounts:
>     - mountPath: /mnt/data
>       name: task-pv-storage
> ~~~
> {: .language-yaml}
{: .solution}

Create the storage pod and copy the files from there
```bash
kubectl apply -f pv-pod.yaml -n argo
kubectl cp  pv-pod:/mnt/data /tmp/poddata -n argo
```
and you will get the file created by the job in `/tmp/poddata/test.txt`.

{% include links.md %}