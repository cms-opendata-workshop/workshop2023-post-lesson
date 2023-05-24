---
title: "Storing a workflow output on Kubernetes"
teaching: 5
exercises: 30
questions:
- "How to setup a workflow engine to submit jobs?"
- "How to run a simple job?"
- "How can I set up shared storage for my workflows?"
- "How to run a simple job and get the the ouput?"
objectives:
- "Understand how to run a simple workflows in a commercial cloud environment or local machine"
- "Understand how to set up shared storage and use it in a workflow"
keypoints:
- "With Kubernetes one can run workflows similar to a batch system"
- "Open Data workflows can be run in a commercial cloud environment using modern tools"
---


## Kubernetes Cluster - Storage Volume

When running applications or workflows, it is often necessary to allocate disk space to store the resulting data. Kubernetes provides various options for managing storage volumes, allowing you to efficiently store and access data within your cluster.

<div id="kubernetes-run">
  <div>
        <ul class="nav nav-tabs" role="tablist">
        <li role="presentation" class="active"><a data-os="GKE" href="#shell-gke" aria-controls="GKE" role="tab" data-toggle="tab">GKE</a></li>
        <li role="presentation"><a data-os="minikube" href="#shell-minikube" aria-controls="Minikube" role="tab" data-toggle="tab">Minikube</a></li>
        </ul>
        <div class="tab-content">
            <article role="tabpanel" class="tab-pane active" id="shell-gke">
              
<p>In many applications or workflows, there is a need for disk space to store and persist results. While Google Kubernetes Engine (GKE) provides persistent disks as default storage for applications running in the cluster, local machines do not have such built-in persistent storage capabilities. To address this, we can create a persistent volume using an NFS disk, allowing us to store and access data from our local machines. It's important to note that persistent volumes are not namespaced and are available to the entire Kubernetes cluster.</p>     
                            
<p>Create the volume (disk) we are going to use:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>gcloud compute disks create --size=100GB --zone=europe-west1-b gce-nfs-disk-1
</code></pre></div></div> 
              
<p>Set up an nfs server for this disk:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/001-nfs-server.yaml
kubectl apply -n argo -f 001-nfs-server.yaml
</code></pre></div></div> 
              
<p>Set up a nfs service, so we can access the server:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/002-nfs-server-service.yaml
kubectl apply -n argo -f 002-nfs-server-service.yaml
</code></pre></div></div> 
              
<p>Let’s find out the IP of the nfs server:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get -n argo svc nfs-server | grep ClusterIP | awk '{ print $3; }'
</code></pre></div></div> 
              
<p>Let’s create a persistent volume out of this nfs disk. Note that persistent volumes are not namespaced they are available to the whole cluster.</p> 
              
<p>We need to write that IP number above into the appropriate place in this file:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/003-pv.yaml
</code></pre></div></div>
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>vim 003-pv.yaml
</code></pre></div></div>                   
              
<p>Deploy:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f 003-pv.yaml
</code></pre></div></div> 
  
<p>Check:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pv
</code></pre></div></div>
  
<p>Apps can claim persistent volumes through persistent volume claims (pvc). Let’s create a pvc:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/003-pvc.yaml
kubectl apply -n argo -f 003-pvc.yaml
</code></pre></div></div> 
              
<p>Check:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pvc -n argo
</code></pre></div></div> 
  
<p>Now an argo workflow coul claim and access this volume with an argo Workflow, let's retrieve the configuration file with:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/argo-wf-volume.yaml
</code></pre></div></div> 
              
<p>The file has the following content:</p> 
  
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># argo-wf-volume.yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: test-hostpath-
spec:
  entrypoint: test-hostpath
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: nfs-1
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
</code></pre></div></div> 
  
<p>Submit and check this workflow with:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo submit -n argo argo-wf-volume.yaml
argo list -n argo
</code></pre></div></div> 
  
<p>Take the name of the workflow from the output (replace XXXXX in the following command) and check the logs:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl logs pod/test-hostpath-XXXXX  -n argo main
</code></pre></div></div> 
  
<p>Once the job is done, you will see something like:</p> 
  
<div class="language-plaintext output highlighter-rouge"><div class="highlight"><pre class="highlight"><code>ls -l /mnt/vol: total 20 drwx------ 2 root root 16384 Sep 22 08:36 lost+found -rw-r--r-- 1 root root 18 Sep 22 08:36 test.txt
</code></pre></div></div>
              
<h3>Get the output file</h3> 
             
<p>The example job above produced a text file as an output. It resides in the persistent volume that the workflow job has created. To copy the file from that volume to the cloud shell, we will define a container, a “storage pod” and mount the volume there so that we can get access to it.</p>  
              
<p>Get the <code class="language-plaintext highlighter-rouge">pv-pod.yaml</code> file with:</p>    
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/pv-pod.yaml
</code></pre></div></div> 
              
<p>This file must have the following content:</p>    

              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># pv-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: nfs-1
  containers:
    - name: pv-container
      image: busybox
      command: ["tail", "-f", "/dev/null"]
      volumeMounts:
        - mountPath: /mnt/data
          name: task-pv-storage
</code></pre></div></div>              
 
<p>Create the storage pod and copy the files from there</p>    
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pv-pod.yaml -n argo
kubectl cp  pv-pod:/mnt/data /tmp/poddata -n argo
</code></pre></div></div>
              
<p>and you will get the file created by the job in <code class="language-plaintext highlighter-rouge">/tmp/poddata/test.txt</code>.</p>    
              
            </article><!-- gke  -->

            <article role="tabpanel" class="tab-pane" id="shell-minikube">                       
              
<p>Local storage volumes are tied to the Minikube host machine and are not accessible outside of it. They are meant for local development and testing purposes. By leveraging local storage volumes with Minikube, you can conveniently create and utilize storage resources within your local Kubernetes cluster, enabling data persistence and local development capabilities.</p>   
              
              
<p>Let's create a persistent volume, retrieve the persistent volume configuration file with:</p>   
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/minikube/pv.yaml
</code></pre></div></div>  
              
<p>It has the following content, you can alter the storage capacity if you'd like to whatever value.</p>
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/vol"
---
</code></pre></div></div>   
              
<p>Deploy:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pv.yaml -n argo
</code></pre></div></div>
              
<p>Check:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pv
</code></pre></div></div>
              
<p>Apps can claim persistent volumes through persistent volume claims (pvc). Let’s create a pvc, retrieve the pvc with:</p>   
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/minikube/pvc.yaml
</code></pre></div></div>  
              
<p>It has the following content, you can alter the storage request if you'd like, but it mas less or equal than the storage capacity defined in our persistent volume (previous step).</p>             
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
</code></pre></div></div>  
   
<p>Deploy:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pvc.yaml -n argo
</code></pre></div></div> 
             
<p>Check:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pvc -n argo
</code></pre></div></div> 
              
<p>Now an argo workflow can claim and access this volume, retrieve the configuration file with:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/minikube/argo-wf-volume.yaml
</code></pre></div></div>  
              
<p>It has the following content:</p>                 
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># argo-wf-volume.yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: test-hostpath-
spec:
  entrypoint: test-hostpath
  volumes:
    - name: workdir
      hostPath:
        path: /mnt/vol
        type: DirectoryOrCreate
  templates:
  - name: test-hostpath
    script:
      image: alpine:latest
      command: [sh]
      source: |
        echo "This is the new ouput" > /mnt/vol/test1.txt
        echo ls -l /mnt/vol: `ls -l /mnt/vol`
      volumeMounts:
      - name: workdir
        mountPath: /mnt/vol
</code></pre></div></div>  
              
<p>Submit and check this workflow with:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo submit -n argo argo-wf-volume.yaml
argo list -n argo
</code></pre></div></div> 
  
<p>Take the name of the workflow from the output (replace XXXXX in the following command) and check the logs:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl logs pod/test-hostpath-XXXXX  -n argo main
</code></pre></div></div> 
  
<p>Once the job is done, you will see something like:</p> 
  
<div class="language-plaintext output highlighter-rouge"><div class="highlight"><pre class="highlight"><code>time="2022-07-25T05:51:14.221Z" level=info msg="capturing logs" argo=true
ls -l /mnt/vol: total 4 -rw-rw-rw- 1 root root 18 Jul 25 05:51 test.txt
</code></pre></div></div>
              
<h3>Get the output file</h3> 
             
<p>The example job above produced a text file as an output. It resides in the persistent volume that the workflow job has created. To copy the file from that volume to the shell, we will define a container, a “storage pod”, and mount the volume there so that we can get access to it.</p>  
              
<p>Retrieve the file <code class="language-plaintext highlighter-rouge">pv-pod.yaml</code> with:</p>    
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/minikube/pv-pod.yaml
</code></pre></div></div>  
              
<p>It has the following content:</p>                
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># pv-pod.yaml
# pv-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: busybox
      command: ["tail", "-f", "/dev/null"]
      volumeMounts:
        - mountPath: /mnt/vol
          name: task-pv-storage
      resources:
        limits:
          cpu: "2"
          memory: "3Gi"
        requests:
          cpu: "1"
          memory: "512Mi"
</code></pre></div></div>              
 
<p>Create the storage pod and copy the files from there with:</p>    
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pv-pod.yaml -n argo
kubectl cp task-pv-pod:/mnt/data /tmp/poddata -n argo
</code></pre></div></div>
                          
<p>and you will get the file created by the job in <code class="language-plaintext highlighter-rouge">/tmp/poddata/test.txt</code>.</p>    
    
<blockquote class="testimonial">
          <p>Every time you want the files to get copied from your the pv-pod to your local computer you must run <code class="language-plaintext highlighter-rouge">kubectl cp task-pv-pod:/mnt/vol /tmp/poddata -n argo</code>.</p></blockquote>
              
              </article><!-- Minikube  -->
        </div> <!-- tab-contents  -->
    </div><!-- nav-tabs  -->
</div><!-- kubernetes-run  -->
