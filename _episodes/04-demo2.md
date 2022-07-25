---
title: "Demo: Storing a workflow output on Kubernetes"
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


## Install argo as a workflow engine

<div id="kubernetes-run">
  <div>
        <ul class="nav nav-tabs" role="tablist">
        <li role="presentation" class="active"><a data-os="GKE" href="#shell-gke" aria-controls="GKE" role="tab" data-toggle="tab">GKE</a></li>
        <li role="presentation"><a data-os="minikube" href="#shell-minikube" aria-controls="Minikube" role="tab" data-toggle="tab">Minikube</a></li>
        </ul>

        <div class="tab-content">

            <article role="tabpanel" class="tab-pane active" id="shell-gke">

<p>While jobs can also be run manually, a workflow engine makes defining and submitting jobs easier. In this tutorial, we use [argo](https://argoproj.github.io/argo/quick-start/). Install it into your working environment with the following commands (all commands to be entered into the cloud shell):</p>
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
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
</code></pre></div></div>

<p>This will also install the argo binary, which makes managing the workflows
easier.</p>
              
<blockquote class="testimonial">
  <p>In case you leave your computer, you might have to reconnect to the CloudShell again, and also on a different computer. If the <code class="language-plaintext highlighter-rouge">argo</code> command is not found, run the command above again starting from the <code class="language-plaintext highlighter-rouge">curl</code> command.</p>
</blockquote>

<p>You can now check that argo is available with:</p>
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo version
</code></pre></div></div>
              
<h3>Run a simple test workflow</h3>
             
<p>To test the setup, run a simple test workflow with</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
</code></pre></div></div>
              
<p>Wait till the yellow light turns green.
Get the logs with</p>              

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo logs -n argo @latest
</code></pre></div></div>              
              
<p>If argo was installed correctly you will have the following:</p>      
              
<div class="language-plaintext output highlighter-rouge"><div class="highlight"><pre class="highlight"><code>hello-world-ml5bf: time="2022-07-25T12:33:54.295Z" level=info msg="capturing logs" argo=true
hello-world-ml5bf:  _____________
hello-world-ml5bf: < hello world >
hello-world-ml5bf:  -------------
hello-world-ml5bf:     \
hello-world-ml5bf:      \
hello-world-ml5bf:       \
hello-world-ml5bf:                     ##        .
hello-world-ml5bf:               ## ## ##       ==
hello-world-ml5bf:            ## ## ## ##      ===
hello-world-ml5bf:        /""""""""""""""""___/ ===
hello-world-ml5bf:   ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
hello-world-ml5bf:        \______ o          __/
hello-world-ml5bf:         \    \        __/
hello-world-ml5bf:           \____\______/
</code></pre></div></div>
              
<p>Please mind that it is important to delete your workflows once they have completed. If you do not do this, the pods associated with the workflow will remain scheduled in the cluster, which might lead to additional charges. You will learn how to automatically remove them later.</p>  
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo delete -n argo @latest
</code></pre></div></div>                

<h3>Storage Volume</h3>  
              
<p>If we run some application or workflow, we usually require a disk space where to dump our results.  There is no persistent disk by default, we have to create it.</p>     
              
<p>You could create a disk clicking on the web interface above, but lets do it faster in the command line.</p>    
              
<p>Create the volume (disk) we are going to use:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>gcloud compute disks create --size=100GB --zone=europe-west1-b gce-nfs-disk-1
</code></pre></div></div> 
              
<p>Set up an nfs server for this disk:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/001-nfs-server.yaml
kubectl apply -n argo -f 001-nfs-server.yaml
</code></pre></div></div> 
              
<p>Set up a nfs service, so we can access the server:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/002-nfs-server-service.yaml
kubectl apply -n argo -f 002-nfs-server-service.yaml
</code></pre></div></div> 
              
<p>Let’s find out the IP of the nfs server:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get -n argo svc nfs-server |grep ClusterIP | awk '{ print $3; }'
</code></pre></div></div> 
              
<p>Let’s create a persistent volume out of this nfs disk. Note that persistent volumes are not namespaced they are available to the whole cluster.</p> 
              
<p>We need to write that IP number above into the appropriate place in this file:</p> 
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/003-pv.yaml
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
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>wget https://cms-opendata-workshop.github.io/workshop2022-lesson-introcloud/files/003-pvc.yaml
kubectl apply -n argo -f 003-pvc.yaml
</code></pre></div></div> 
              
<p>Check:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pvc -n argo
</code></pre></div></div> 
  
<p>Now an argo workflow coul claim and access this volume with a configuration like:</p> 
  
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
              
<p>Create a file <code class="language-plaintext highlighter-rouge">pv-pod.yaml</code> with the following contents:</p>    
              
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

<p>While jobs can also be run manually, a workflow engine makes defining and submitting jobs easier. In this tutorial, we use [argo](https://argoproj.github.io/argo/quick-start/). Install it into your working environment with the following commands (all commands to be entered into the cloud shell):</p>
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml
</code></pre></div></div>

<p>This will also install the argo binary, which makes managing the workflows
easier.</p>

<p>You can now check that argo is available with:</p>
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo version
</code></pre></div></div>
              
<h3>Run a simple test workflow</h3>
             
<p>To test the setup, run a simple test workflow with</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
</code></pre></div></div>
              
<p>Wait till the yellow light turns green.
Get the logs with</p>              

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo logs -n argo @latest
</code></pre></div></div>              
              
<p>If argo was installed correctly you will have the following:</p>
              
<div class="language-plaintext output highlighter-rouge"><div class="highlight"><pre class="highlight"><code>hello-world-ml5bf: time="2022-07-25T12:33:54.295Z" level=info msg="capturing logs" argo=true
hello-world-ml5bf:  _____________
hello-world-ml5bf: < hello world >
hello-world-ml5bf:  -------------
hello-world-ml5bf:     \
hello-world-ml5bf:      \
hello-world-ml5bf:       \
hello-world-ml5bf:                     ##        .
hello-world-ml5bf:               ## ## ##       ==
hello-world-ml5bf:            ## ## ## ##      ===
hello-world-ml5bf:        /""""""""""""""""___/ ===
hello-world-ml5bf:   ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
hello-world-ml5bf:        \______ o          __/
hello-world-ml5bf:         \    \        __/
hello-world-ml5bf:           \____\______/
</code></pre></div></div>
              
<p>Please mind that it is important to delete your workflows once they have completed. If you do not do this, the pods associated with the workflow will remain scheduled in the cluster, which might lead to additional charges. You will learn how to automatically remove them later.</p>  
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>argo delete -n argo @latest
</code></pre></div></div>                

<h3>Storage Volume</h3>                
              
<p>If we run some application or workflow, we usually require a disk space where to dump our results. Unlike GKE, our local machine is the persistent disk by default. So let's create a persistent volume out of this nfs disk. Note that persisten volumes are not namespaced they are available to the whole cluster.</p>   
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>nano pv.yaml
</code></pre></div></div>    
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code>apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
</code></pre></div></div>   
              
<p>Deploy:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pv.yaml -n argo
</code></pre></div></div>
              
<p>Check:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pv
</code></pre></div></div>
              
<p>Apps can claim persistent volumes through persistent volume claims (pvc). Let’s create a pvc:</p>   
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>nano pvc.yaml
</code></pre></div></div>    
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code>apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
</code></pre></div></div>  
   
<p>Deploy:</p> 
      
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pvc.yaml -n argo
</code></pre></div></div> 
             
<p>Check:</p> 
  
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get pvc -n argo
</code></pre></div></div> 
              
<p>Now an argo workflow coul claim and access this volume with a configuration like:</p> 
              
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
        path: /mnt/data
        type: DirectoryOrCreate
  templates:
  - name: test-hostpath
    script:
      image: alpine:latest
      command: [sh]
      source: |
        echo "This is the ouput" > /mnt/vol/test.txt
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
             
<p>The example job above produced a text file as an output. It resides in the persistent volume that the workflow job has created. To copy the file from that volume to the cloud shell, we will define a container, a “storage pod” and mount the volume there so that we can get access to it.</p>  
              
<p>Create a file <code class="language-plaintext highlighter-rouge">pv-pod.yaml</code> with the following contents:</p>    
              
<div class="language-code highlighter-rouge"><div class="highlight"><pre class="highlight"><code># pv-pod.yaml
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
        - mountPath: /mnt/data
          name: task-pv-storage
</code></pre></div></div>              
 
<p>Create the storage pod and copy the files from there</p>    
              
<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply -f pv-pod.yaml -n argo
kubectl cp  task-pv-pod:/mnt/data /tmp/poddata -n argo
</code></pre></div></div>
              
<p>and you will get the file created by the job in <code class="language-plaintext highlighter-rouge">/tmp/poddata/test.txt</code>.</p>    
              
              </article><!-- Minikube  -->
        </div> <!-- tab-contents  -->
    </div><!-- nav-tabs  -->
</div><!-- kubernetes-run  -->



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
