---
title: Setup
---
## Welcome!
The workshop will introduce participants to Kubernetes, an open-source container orchestration platform. Kubernetes provides a scalable and efficient infrastructure for running applications in the cloud. It allows users to manage and automate the deployment, scaling, and monitoring of containerized applications. In the context of the CMS OpenData workshop, Kubernetes will be used to create an environment on [Google Cloud Platform](https://cloud.google.com), enabling participants to run realistic physics analyses in the cloud.

## Setup
In this pre-exercise, we will guide you through setting up a working cloud environment for running realistic physics analysis using Google Cloud Platform (GCP). Here are the steps to get started:

### 1. GCP (Google Cloud Platform):
To participate in this workshop, you will need a [GCP](https://cloud.google.com) account. Creating a GCP account allows you to access $300.00 USD in credit, which can be used within a period of 3 months. Please note that a credit or debit card is required during the account creation process. However, it's important to mention that for the Cloud Computing lesson, we will provide temporary resources, so activating your own GCP account credit is not entirely necessary. 

### 2. GKE(Google Kubernetes Engine) or Minikube:
For this Cloud Pre-exercise, you will need to create a Kubernetes cluster to run some tests. You have two options:
* **GKE:** _Only if_ you opted to access the $300.00 USD in credit, you can create your cloud environment in [GKE](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwj_k8-twIf_AhUOmYQIHYroCxAQFnoECAwQAQ&url=https%3A%2F%2Fcloud.google.com%2Fkubernetes-engine&usg=AOvVaw0qQOKliejkrhEK8IRWdf5r). Please do not forget to [clean up](https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/07-clean/index.html) (delete everything) when you are finished, since you can run out of credit and **be charged** respectively. 

* **Minikube:** If you prefer to use minikube (which is fine), please click [here](https://minikube.sigs.k8s.io/docs/start/) to get started. 
  * **Kubectl**: When running minikube locally, you will need to [download the kubectl](https://kubernetes.io/docs/tasks/tools/) command-line tool in order to interact with your cluster.

### 3. Prep-work Episodes:
Once you have decided on your preferred approach (GCP or minikube), please refer to the prep-work episodes of this lesson. These episodes will provide you with the necessary instructions and guidance to set up your cloud environment. Follow the step-by-step process outlined in the prep-work episodes to ensure a smooth setup.

### 4. Mattermost Cloud Pre-Exercises Channel:
Don't forget to subscribe to the [Mattermost cloud pre-exercises channel](https://mattermost.web.cern.ch/cmsodws2023/channels/cloud-pre-exercise). This channel will provide you with important updates, announcements, and any additional resources or support you may need throughout the workshop. It's a valuable communication platform to stay connected with instructors and fellow participants.

We wish you good luck with the workshop! Feel free to reach out if you have any questions or need assistance along the way. Enjoy the learning experience!

{% include links.md %}
