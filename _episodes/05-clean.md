---
title: "Cleaning up"
reading: 5
exercises: 10
questions:
- "How do I clean my workspace?"
- "How do I delete my cluster?"
objectives:
- "Clean my workflows"
- "Delete my storage volume"
keypoints:
- "Cleaning your workspace in periods of time while you're not running workflows will save you money."
- "With a couple of commands it is easy to get back to square one."
---


## Cleaning workspace

Remember to delete your workflow again to avoid additional charges:
Run this until you get a message indicating there are no more workflows.

```bash
argo delete -n argo @latest
```

Delete the argo namespace:

```bash
kubectl delete ns argo
```

If you are working on Cloud shell, it will keep your local files, such as yaml files that you downloaded. If you plan to use them afterwards, you can leave them there.

Delete your disk:
```bash
gcloud compute disks delete DISK_NAME [DISK_NAME …] [--region=REGION     | --zone=ZONE]
```

> ## Demo delete disk
>
> To delete the disk 'gce-nfs-disk-1' in zone 'europe-west1-b' that was used as an example in this workshop, run:
>
> ```bash
> gcloud compute disks delete gce-nfs-disk-1 --zone=europe-west1-b
> ```
>
{: .testimonial}

## Delete cluster

* Click on the delete button of your cluster:
![](../fig/Connection4.png)

* Confirm deletion:
![](../fig/DeleteCluster.png)

* Standby to see the complete deletion of the cluster:
![](../fig/StartOver5.png)
Perfect you’re ready to start over

{% include links.md %}
