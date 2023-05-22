---
title: "Getting started with Argo"
teaching: 10
exercises: 0
questions:
- "What is Argo?"
- "How to use Argo commands?"
- "What are Argo workflows?"

objectives:
- "Appreciate the necessity for the Argo workflows tool (or similar)"
- "Learn the basics of Argo commands"

keypoints:
- "`kubectl` is the ruler of GKE"
- "Argo is a very useful tool for running workflows and parallel jobs"
- "To be able to write, read and extract data, a few services/resources need to be set up on the GCP"
---

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

                            
            </article><!-- gke  -->

            <article role="tabpanel" class="tab-pane" id="shell-minikube">

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

              </article><!-- Minikube  -->
        </div> <!-- tab-contents  -->
    </div><!-- nav-tabs  -->
</div><!-- kubernetes-run  -->



