---
title: "Deploy a Webserver"
teaching: 5
exercises: 20
questions:
- "How can I visualize my workflows?"
objectives:
- "Prepare to deploy the fileserver that mounts the storage volume."
- "Submit your workflow and get the results."
keypoints:
- "With a simple but a tight yaml structure, a full-blown analysis can be performed and accessed from a K8s cluster."
---

> ## Minikube
>
> This episode is relevant when working on the Google Kubernetes Engine, as will be done during the hands-on session of the workshop. 
> If you are going through these pre-exercises on minikube, just read this as part of your information, but do not work through it.
>
{: .testimonial}

## Accessing files via http

The storage volume in your Kubernetes cluster provides a convenient way to store and manage files. While you can copy files between the storage volume and the Cloud Console, a practical use case involves running "big data" workloads in the cloud and then downloading the output to your local desktop or laptop for further processing. To make your files easily accessible to the outside world, you can deploy a web server that mounts the storage volume.

### Benefits of a Web Server for File Access

Deploying a web server that mounts the storage volume offers several advantages:

* Remote Access: A web server allows you to access and download files from the storage volume remotely. You can retrieve your data from anywhere with an internet connection, enabling seamless collaboration and remote file management.
* Simplified Sharing: By hosting your files on a web server, you can easily share them with others. You can provide access to specific files or directories, control permissions, and share download links with colleagues or clients.
* Additional Processing: With a web server, you can perform further processing of the files directly on your local machine. This flexibility allows you to leverage local computing resources and tools for data analysis, visualization, or other tasks.

### Deploying a Web Server

We first patch the config of the webserver to be created as follows:

```bash
mkdir conf.d
cd conf.d
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/nginx-basic.conf
cd ..
kubectl create configmap basic-config --from-file=conf.d -n argo
```

Then prepare to deploy the fileserver by downloading the manifest:

```bash
wget https://cms-opendata-workshop.github.io/workshop2023-lesson-introcloud/files/GKE/deployment-http-fileserver.yaml
```

> ## YAML File
> The provided YAML configuration represents a Kubernetes Deployment for an HTTP file server. It deploys an instance of the NGINX web server container and mounts a persistent volume claim for file storage, as well as a ConfigMap for basic configuration.
> ~~~
> # deployment-http-fileserver.yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   labels:
>     service: http-fileserver
>   name: http-fileserver
> spec:
>   replicas: 1
>   strategy: {}
>   selector:
>     matchLabels:
>       service: http-fileserver
>   template:
>     metadata:
>       labels:
>         service: http-fileserver
>     spec:
>       volumes:
>       - name: volume-output
>         persistentVolumeClaim:
>           claimName: nfs-1
>       - name: basic-config
>         configMap:
>           name: basic-config
>       containers:
>       - name: file-storage-container
>         image: nginx
>         ports:
>         - containerPort: 80
>         volumeMounts:
>           - mountPath: "/usr/share/nginx/html"
>             name: volume-output
>           - name: basic-config
>             mountPath: /etc/nginx/conf.d
> ~~~
> {: .language-yaml}
{: .solution}

Apply and expose the port as a `LoadBalancer`:

```bash
kubectl create -n argo -f deployment-http-fileserver.yaml
kubectl expose deployment http-fileserver -n argo --type LoadBalancer --port 80 --target-port 80
```

Exposing the deployment will take a few minutes. Run the following command to
follow its status:

```bash
kubectl get svc -n argo
```

You will initially see a line like this:

~~~
NAME                          TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
http-fileserver               LoadBalancer   10.8.7.24    <pending>     80:30539/TCP   5s
~~~
{: .output}

Wait a couple minutes recheck the previous command, copy your EXTERNAL-IP and paste it on a new tab, it should look something like this:

![](../fig/HelloNFS.png)

The `<pending>` `EXTERNAL-IP` will update after a few minutes (run the command
again to check). Once it's there, copy the IP and paste it into a new browser
tab. This should welcome you with a "Hello from NFS" message. In order to
enable file browsing, we need to delete the `index.html` file in the pod.

![](../fig/Index.png)

Determine the pod name using the first command listed below and adjust the
second command accordingly.
  
```bash
kubectl get pods -n argo
kubectl exec http-fileserver-XXXXXXXX-YYYYY -n argo -- rm /usr/share/nginx/html/index.html
```
<b\>
By deploying this HTTP file server configuration, you can serve and share files from the persistent volume, leveraging NGINX's capabilities for hosting and delivering content over HTTP.
  
### Security Considerations
  
When deploying a web server that exposes your storage volume to the outside world, consider the following security considerations:
* Implement authentication and authorization mechanisms to control access to the files hosted by the web server.
* Enable secure connections (HTTPS) to protect data in transit, especially if sensitive information is being served.
* Regularly update the web server and associated software to mitigate security vulnerabilities.
* Follow security best practices and ensure that proper access controls are in place to prevent unauthorized access to your files.
  
By deploying a web server that mounts the storage volume, you can easily access and share files stored in your Kubernetes cluster. This approach allows you to perform further processing on your local machine while maintaining flexibility and security in file management and collaboration.
<b\>
> ## Warning: anyone can now access these files
>
> This IP is now accessible from anywhere in the world, and therefore also
> your files (mind: there are charges for outgoing bandwidth). Please delete
> the service again once you have finished downloading your files.
>
> ```bash
> kubectl delete svc/http-fileserver -n argo
> ```
>
> Run the `kubectl expose deployment` command to expose it again.
>
{: .testimonial}
