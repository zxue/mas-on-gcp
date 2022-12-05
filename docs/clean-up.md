# Shut Down the OpenShift Cluster

For dev/test environment, you may want to shut down the OpenShift
environment to reduce costs. You can run the command line below to [shut
down the cluster gracefully](https://docs.openshift.com/container-platform/4.8/backup_and_restore/graceful-cluster-shutdown.html).

```
for node in $(oc get nodes -o jsonpath='{.items[*].metadata.name}'); do oc debug node/${node} -- chroot /host shutdown -h 1; done 
```

To resume the Cluster, you can restart each master node instance first
and then each worker node instance from the cloud console. Wait until
all virtual instances are up and running before running any commands
against the cluster.

Do not shut down the virtual machine instances directly from GCP cloud
console. If you do so, the cluster may not work properly when you resume
the virtual machine instances.

# Delete OpenShift Cluster and MAS Deployment

You can delete the project in GCP if you want to delete OpenShift or
MAS. To delete all resources without deleting the project, you can do so
in the GCP cloud, which is straightforward, or run "gcloud" commands.

Under the VM Instances section:

-   delete all virtual machine instances

-   delete all disks

-   delete 7 service accounts

Under the Hybrid Connectivity section:

-   delete 1 cloud routers

Under the Network Services section:

-   delete 3 loadbalancers including healthchecks

-   delete 1 dns zone including 3 recordsets

-   delete 2 nats

Under the VPC networks section:

-   delete 1 vpc network peering (if filestore is configured)

-   delete 10+1 firewall rules (1 for ssh to nfs vm)

-   delete 2 IP addresses

-   delete 1 routes

-   delete 2 subnets

-   delete vpc


[Back to ReadMe page](../README.MD)