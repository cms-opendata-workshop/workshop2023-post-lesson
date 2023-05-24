---
title: "Getting started with Argo"
teaching: 10
exercises: 0
questions:
- "What is Argo?"
- "How to use Argo commands?"
- "What are Argo workflows?"
- "How do I deploy my Argo GUI?"

objectives:
- "Appreciate the necessity for the Argo workflows tool (or similar)"
- "Learn the basics of Argo commands"

keypoints:
- "Argo is a very useful tool for running workflows and parallel jobs"
---

## Argo

Argo is a collection of open-source tools that extend the functionality of Kubernetes, providing several benefits for workflow management. Some key advantages of using Argo include:

* **Cloud Agnostic:** Argo can be deployed on any Kubernetes cluster, irrespective of the underlying cloud provider.
* **Resource Monitoring:** Argo enables constant monitoring of resource states, allowing users to track the progress and status of their workflows.
* **Scalability:** Argo supports the execution of multiple jobs simultaneously across different nodes, providing high scalability.
* **Error Debugging:** With Argo, it is easier to identify and debug errors in workflows, ensuring smooth execution.

In the context of Argo, there are three important tools that facilitate working with workflows, we will be using the Argo Workflow Engine.

### Argo Workflow Engine
The Argo Workflow Engine is designed to execute complex job orchestration, including both serial and parallel execution of stages, with each stage executed as a container.

In the context of scientific analysis, such as physics analysis using datasets from the CMS Open Data portal and CMSSW, Argo's orchestration capabilities are particularly valuable. By leveraging Argo, researchers can automate and streamline complex analysis workflows, which often involve multiple processing stages and dependencies. Argo's support for parallel execution and container-based environments allows for efficient utilization of computational resources, enabling faster and more scalable data analysis.

## Install argo as a workflow engine

While jobs can be run manually, utilizing a workflow engine like Argo simplifies the process of defining and submitting jobs. In this tutorial, we will use the Argo Quick Start page to install and configure Argo in your working environment.

<div id="kubernetes-run">
    <div>
        <ul class="nav nav-tabs" role="tablist">
            <li role="presentation" class="active"><a data-os="GKE" href="#shell-gke" aria-controls="GKE" role="tab" data-toggle="tab">GKE</a></li>
            <li role="presentation"><a data-os="minikube" href="#shell-minikube" aria-controls="Minikube" role="tab" data-toggle="tab">Minikube</a></li>
        </ul>

    <div class="tab-content">
      <article role="tabpanel" class="tab-pane active" id="shell-gke">
        <p>Install it into your working environment with the following commands (all commands to be entered into the cloud shell):</p> 
        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml

# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.4.7/argo-darwin-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
sudo mv ./argo-linux-amd64 /usr/local/bin/argo
</code></pre></div></div>   
        
        <p>This will also install the argo binary, which makes managing the workflow easier.</p>
        
        <blockquote class="testimonial">
          <p>In case you leave your computer, you might have to reconnect to the CloudShell again, and also on a different computer. If the <code class="language-plaintext highlighter-rouge">argo</code> command is not found, run the command above again starting from the <code class="language-plaintext highlighter-rouge">curl</code> command.</p>
        </blockquote>
      </article><!-- gke -->
      
      <article role="tabpanel" class="tab-pane" id="shell-minikube">
        <p>Install it into your working environment with the following commands (all commands to be entered into your local shell):</p>    
        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml

# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.4.7/argo-darwin-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
sudo mv ./argo-darwin-amd64 /usr/local/bin/argo</code></pre></div></div> 
        <p>This will also install the argo binary, which makes managing the workflows easier.</p>   
      </article><!-- Minikube  -->
    </div> <!-- tab-contents  -->
  </div><!-- nav-tabs  -->
</div><!-- kubernetes-run  -->
<br/>
You can now check that argo is available with:
              
```bash
argo version
```    
         
         
### Run a simple test workflow
             
To test the setup, run a simple test workflow with:
```bash
argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
```
              
Wait till the yellow light turns green. 
<br/>
Get the logs with:
```bash
argo logs -n argo @latest
```
              
If argo was installed correctly you will have the following:
              
```output
hello-world-ml5bf: time="2022-07-25T12:33:54.295Z" level=info msg="capturing logs" argo=true
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
```
              
Please mind that it is important to delete your workflows once they have completed. If you do not do this, the pods associated with the workflow will remain scheduled in the cluster, which might lead to additional charges. You will learn how to automatically remove them later.
              
```bash
argo delete -n argo @latest
```

## Argo GUI

Argo UI is a web-based user interface that provides a graphical representation and control panel for managing and monitoring workflows in Argo. It offers a visual interface for creating, submitting, tracking, and troubleshooting workflows. With Argo UI, users can easily interact with and manage their workflows without the need for manual command-line operations.
<br/>
Let's get started!

<div id="argo-run">
    <div>
        <ul class="nav nav-tabs" role="tablist">
            <li role="presentation" class="active"><a data-os="GKE1" href="#shell-gke1" aria-controls="GKE1" role="tab" data-toggle="tab">GKE</a></li>
            <li role="presentation"><a data-os="minikube1" href="#shell-minikube1" aria-controls="Minikube1" role="tab" data-toggle="tab">Minikube</a></li>
        </ul>

    <div class="tab-content">
      
      <article role="tabpanel" class="tab-pane active" id="shell-gke1">

        <p>Check the services running and the associated IP addresses:</p>    
        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get svc -n argo
kubectl -n argo port-forward deployment/argo-server 2746:2746</code></pre></div></div>
        
        <p>Once it has started fowarding the port we will have to manually enable the port, to do this open a new cloud shell tab and run the following command:</p>
        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>lynx https://localhost:2746</code></pre></div></div>  

        <p>Access it and then quit. Return to the previous tab and you will see that the port is being accessed and handled, you can exit with ^C and finally patch the service with:</p>
        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl patch svc argo-server -n argo -p '{"spec": {"type": "LoadBalancer"}}'</code></pre></div></div>

        <p>Since it is creating an external ip, wait a couple minutes. You can check if it is ready with:</p>
        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get svc -n argo</code></pre></div></div>     

        <p>Finally, you can access this address in your localhost. Do not forget to add “https://” and “:2746”. Click on Advanced, proceed to ip(unsafe) and voilà.</p>
        
        <img src="../fig/ArgoInterface.png" width="1000">
        </article><!-- gke -->
        <article role="tabpanel" class="tab-pane" id="shell-minikube1">
              
        <p>Check the services running and the associated IP addresses:</p>    

        <div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl get svc -n argo
kubectl -n argo port-forward deployment/argo-server 2746:2746</code></pre></div></div>        
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

{% include links.md %}
