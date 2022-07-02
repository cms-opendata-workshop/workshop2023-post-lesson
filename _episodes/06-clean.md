---
title: "Cleaning up"
teaching: 5
exercises: 10
questions:
- "How do I clean my workspace?"
- "How do I delete my cluster?"
objectives:
- "Clean my workflows"
- "Delete my storage volume"
keypoints:
- "Cleaning your workspace in periods of time while you're not running workflows will save you money."
- "With a couple commands it is easy to get back to square one."
---


## Cleaning workspace

Remember to delete your workflow again to avoid additional charges:
Run this until you get a message indicating there is no more workflows.

```bash
argo delete -n argo @latest
```

Delete the argo namespace and all yaml files and configurations with:

```bash
kubectl delete ns argo
rm *
rm -r *
```

Delete your disk:
```bash
gcloud compute disks delete DISK_NAME [DISK_NAME …] [--region=REGION     | --zone=ZONE]
```

> ## Demo delete disk
>
> To delete the disk 'gce-nfs-disk-1' in zone 'us-central1-c' that was used as an example in this workshop , run:
>
> ```bash
> gcloud compute disks delete gce-nfs-disk-1 --zone=us-central1-c
> ```
>
{: .testimonial}

## Delete cluster

* Click on the delete button of your cluster:
![](../fig/Connection4.PNG)

* Confirm deletion:
![](../fig/DeleteCluster.PNG)

* Standby to see the complete deletion of the cluster:
![](../fig/StartOver5.PNG)
Perfect you’re ready to start over
