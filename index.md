---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---
Physics analyses in the field of particle physics often involve processing large amounts of data, typically spanning several terabytes. Within the CMS collaboration, these analyses have been performed using high-throughput batch systems like HTCondor, which is installed at CERN and various research institutions. Additionally, the worldwide LHC computing grid (WLCG) has been instrumental in providing distributed computing resources for such analyses. However, open data users may not have access to these resources at their own institutions. To overcome this limitation, the CMS OpenData workshop in 2023 aims to enable researchers to utilize computing resources through public cloud vendors.

> ## What is this lesson about?
>
> Welcome.  In this lesson you will:
> - Learn what Kubernetes is.
> - Understand the basic way on how Kubernetes and Argo operate.
> - Learn how to create a K8s cluster prepared for physics analysis.
> - Learn to configure and run a simple Argo workflow.
{: .objectives}

> ## Prerequisites
> In order to complete this lesson you need
> - A basic understanding of [UNIX shells](https://swcarpentry.github.io/shell-novice/) (external link).
> - A basic understanding of containers (after completing the [Docker](https://cms-opendata-workshop.github.io/workshop2022-lesson-docker/) lesson).
{: .prereq}

> ## Helpline
> Remember that we are always available to help. Our [Mattermost](https://mattermost.web.cern.ch/cmsodws2023/channels/cloud-pre-exercise) channel is open.
{: .callout}

{% include links.md %}
