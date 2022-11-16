Benjamin Xue, Principal Solution Architect at IBM

Jenny Wang, Software Architect at IBM

# Table of Contents {#table-of-contents .TOC-Heading}

[Prerequisites [2](#prerequisites)](#prerequisites)

[GCP Project, Service Account and Role Assignments
[2](#gcp-project-service-account-and-role-assignments)](#gcp-project-service-account-and-role-assignments)

[Register a custom domain
[3](#register-a-custom-domain)](#register-a-custom-domain)

[Create a DNS zone in GCP
[3](#create-a-dns-zone-in-gcp)](#create-a-dns-zone-in-gcp)

[Add DNS Name Servers in Domain Registration Account
[4](#add-dns-name-servers-in-domain-registration-account)](#add-dns-name-servers-in-domain-registration-account)

[Install Red Hat OpenShift
[5](#install-red-hat-openshift)](#install-red-hat-openshift)

[Install OpenShift with the installer
[5](#install-openshift-with-the-installer)](#install-openshift-with-the-installer)

[Install OpenShift with Daffy
[6](#install-openshift-with-daffy)](#install-openshift-with-daffy)

[Use NFS Server backed Storage for OpenShift
[8](#use-nfs-server-backed-storage-for-openshift)](#use-nfs-server-backed-storage-for-openshift)

[Create a Filestore Instance
[8](#create-a-filestore-instance)](#create-a-filestore-instance)

[Deploy a Dynamic Provisioning Storage Class
[9](#deploy-a-dynamic-provisioning-storage-class)](#deploy-a-dynamic-provisioning-storage-class)

[The Infrastructure of OpenShift Deployment on GCP
[11](#the-infrastructure-of-openshift-deployment-on-gcp)](#the-infrastructure-of-openshift-deployment-on-gcp)

[Network Topology [12](#network-topology)](#network-topology)

[Cloud Services in OpenShift Deployment
[12](#cloud-services-in-openshift-deployment)](#cloud-services-in-openshift-deployment)

[Infrastructure Architecture
[12](#infrastructure-architecture)](#infrastructure-architecture)

[Install Maximo Application Suite
[13](#install-maximo-application-suite)](#install-maximo-application-suite)

[Define Environment Variables
[13](#define-environment-variables)](#define-environment-variables)

[Install MAS Core [15](#install-mas-core)](#install-mas-core)

[Activate MAS Manage [16](#activate-mas-manage)](#activate-mas-manage)

[Deploy IBM Cloud Pak for Data
[16](#deploy-ibm-cloud-pak-for-data)](#deploy-ibm-cloud-pak-for-data)

[Create DB2 Warehouse Instance
[17](#create-db2-warehouse-instance)](#create-db2-warehouse-instance)

[Create Database Schema for Manage
[17](#create-database-schema-for-manage)](#create-database-schema-for-manage)

[Configure Database Connection [17](#_Toc118371200)](#_Toc118371200)

[Activate MAS Manage
[18](#activate-mas-manage-1)](#activate-mas-manage-1)

[Launch MAS Manage [19](#launch-mas-manage)](#launch-mas-manage)

[Storages for OpenShift and Maximo
[20](#storages-for-openshift-and-maximo)](#storages-for-openshift-and-maximo)

[OpenShift Storage Classes
[20](#openshift-storage-classes)](#openshift-storage-classes)

[Maximo DB2WH Storage
[21](#maximo-db2wh-storage)](#maximo-db2wh-storage)

[Troubleshoot MAS Manage Activation
[22](#troubleshoot-mas-manage-activation)](#troubleshoot-mas-manage-activation)

[Check the database connection string in Pods
[22](#check-the-database-connection-string-in-pods)](#check-the-database-connection-string-in-pods)

[Check "ManageWorkspace" custom resource
[22](#check-manageworkspace-custom-resource)](#check-manageworkspace-custom-resource)

[Check "jdbc" custom resource
[23](#check-jdbc-custom-resource)](#check-jdbc-custom-resource)

[Shut Down the OpenShift Cluster
[24](#shut-down-the-openshift-cluster)](#shut-down-the-openshift-cluster)

[Delete OpenShift Cluster and MAS Deployment
[24](#delete-openshift-cluster-and-mas-deployment)](#delete-openshift-cluster-and-mas-deployment)

[Next Steps [25](#next-steps)](#next-steps)

This document provides detailed instructions on how to deploy Red Hat
OpenShift, NFS backed storage and Maximo Application Suite Google Cloud
and some troubleshooting tips.

# Prerequisites

Before installing Red Hat OpenShift, you must complete and meet the
prerequisites.

## GCP Project, Service Account and Role Assignments

Create a project in your Google Cloud account and a service account with
within the project. Grant proper permissions to the service account and
enable a set of Google Cloud API services.

The following APIs must be enabled in your GCP project. You can enable
the API service using gcloud commands or from Google cloud console.

-   iam.googleapis.com

-   deploymentmanager.googleapis.com

-   compute.googleapis.com

-   cloudapis.googleapis.com

-   cloudresourcemanager.googleapis.com

-   dns.googleapis.com

-   iamcredentials.googleapis.com

-   servicemanagement.googleapis.com

-   serviceusage.googleapis.com

-   storage-api.googleapis.com

-   storage-component.googleapis.com

-   networksecurity.googleapis.com

The following roles must be assigned to the service account based on
OpenShift recommended security practices. If the service account is
assigned to the "owner" role instead of individual roles, which is not
recommended, you can install OpenShift.

-   compute.admin

-   iam.securityAdmin

-   iam.serviceAccountAdmin

-   iam.serviceAccountUser

-   iam.serviceAccountKeyAdmin

-   storage.admin

-   dns.admin

## Register a custom domain

One OpenShift requirement is to have a domain name. The domain name
along with its subdomains for APIs and apps are used to access the
OpenShift cluster console and applications including Maximo Application
Suite. You can purchase a domain through Google or other domain service
providers or use an existing domain name. Note that the domain name
registration is done in your Google account, which is separated from
your GCP account.

## Create a DNS zone in GCP

You can create a Cloud DNS zone in your GCP project. While not required,
you may use a subdomain for OpenShift clusters, for example,
gcp.mydomainxxx.com, leaving submains available for other uses.

![Graphical user interface, application Description automatically
generated](media/image1.png){width="4.663905293088364in"
height="5.482082239720035in"}

After a DNS zone is created, one SOA record and one NS (name server)
record are added automatically. Make a note of the NS names, for example
"ns-cloud-a1.googledomains.com."

## Add DNS Name Servers in Domain Registration Account

In your Google domain account, select DNS from the navigation menu and
the "Default name servers" tab. Add a DNS record with the NS names you
obtained from your GCP DNS zone. Save the record.

![Graphical user interface, text, application Description automatically
generated](media/image2.png){width="6.5in" height="2.863888888888889in"}

# Install Red Hat OpenShift

There are various ways of installing Red Hat OpenShift. We tested two of
them on GCP: OpenShift installer and Daffy, an opensource tool managed
by an IBM team. The Maximo team provides the [Ansible
playbooks](https://ibm-mas.github.io/ansible-devops/playbooks/ocp/)
option that you can try.

## Install OpenShift with the installer

You can download the OpenShift installer from your Red Hat account and
install it. For installing OpenShift version 4.8 or higher, please refer
to the document, "[Installing a cluster quickly on
GCP](https://docs.openshift.com/container-platform/4.8/installing/installing_gcp/installing-gcp-default.html)".

By default, the installer creates a new cluster with 3 master nodes and
3 worker nodes. Note that the installer creates the local installation
directory automatically if it does not exist already.

*./openshift-install create cluster \--dir \<installation_directory\>*

This method is also referred to "Installer Provisioned Infrastructure"
or IPI. Behind the scenes the installer creates virtual machines and
disks, load balancers, a virtual network including two sub networks,
firewall rules, IP addresses, routes, DNS records in the selected zone,
cloud NAT records, cloud routers and additional service accounts.

To customize the default configuration, you can create the
"install-config.yaml" file, modify the number of worker nodes, and then
create the cluster using the configuration file. Note that you may want
to save a copy of the revised install-config.yaml file because it is
deleted automatically when the cluster is created.

*./openshift-install create install-config \--dir
\<installation_directory\>*

*./openshift-install create cluster \--dir \<installation_directory\>*

Sample install-config.yaml file

*apiVersion: v1\
baseDomain: gcp.xxx.com\
compute:\
- architecture: amd64\
hyperthreading: Enabled\
name: worker\
platform: {}\
replicas: 3\
controlPlane:\
architecture: amd64\
hyperthreading: Enabled\
name: master\
platform: {}\
replicas: 3\
metadata:\
creationTimestamp: null\
name: \<replace cluster name here\>\
networking:\
clusterNetwork:\
- cidr: 10.128.0.0/14\
hostPrefix: 23\
machineNetwork:\
- cidr: 10.0.0.0/16\
networkType: OpenShiftSDN\
serviceNetwork:\
- 172.30.0.0/16\
platform:\
gcp:\
projectID: \<replace gcp project name here\>\
region: \<e.g. us-east1\>\
publish: External\
pullSecret: \'replace redhat pull secrets here\'\
sshKey: \<replace ssh key here\>*

When the installation is completed, you are provided with the following
cluster console login info.

INFO To access the cluster as the system:admin user when using \'oc\',
run \'export
KUBECONFIG=/home/\<username\>/Downloads/xxx/auth/kubeconfig\'\
INFO Access the OpenShift web-console here:
[https://console-openshift-console.apps.\<clustername\>.gcp.mydomainxxx.com\
](https://console-openshift-console.apps.bx48.gcp.xueopenlabs.com/)INFO
Login to the console with user: \"kubeadmin\", and password:
\"nVwnr-ma84g-kUob3-766B8\"

To install OpenShift in an existing environment on GCP, which is
referred to as "User provisioned infrastructure" or UPI, please refer to
"[Installing a cluster on GCP into an existing
VPC](https://docs.openshift.com/container-platform/4.8/installing/installing_gcp/installing-gcp-vpc.html)".

## Install OpenShift with Daffy

Another easy way to install OpenShift is to use an opensource tool named
Daffy, which stands for "Deployment Automation Framework For You", and
has been created and maintained by one IBM team. You can find specific
instructions on how to install OpenShift on GCP at the
[Daffy](https://ibm.github.io/daffy/Deploying-OCP/GCP/) website.

The Daffy tool uses values stored in an environment variable to install
OpenShift and other IBM products such as Cloud Pak for Data and Cloud
Pak for Business Automation. To create a cluster, download and install
Daffy first on a Linux machine, either Ubuntu 20.04 or REHL 8.x. The
Daffy files are stored in the /data/daffy folder, not your home
directory.

The sample environment file below can be used to create a cluster with 3
master nodes and 6 worker nodes.

*DAFFY_UNIQUE_ID=\"replace with email address here\"*

*DAFFY_DEPLOYMENT_TYPE=Enablement*

*BASE_DOMAIN=\"gcp.mydomainxxx.com\"*

*CLUSTER_NAME=\"replace with cluster name here for example mycluster\"*

*OCP_INSTALL_TYPE=\"gcp-ipi\"*

*OCP_RELEASE=\"4.8.49\"*

*VM_TSHIRT_SIZE=\"Large\"*

*GCP_PROJECT_ID=\"replace with GCP project name here\"*

*GCP_REGION=\"us-east1\"*

*OCP_CREATE_OPENSHIFT_CONTAINER_STORAGE=\"true\"*

*CP4D_VERSION=\"4.5.3\"*

*CP4D_ENABLE_SERVICE_DB2_WAREHOUSE=true*

To create an OpenShift, run the command line in the "/data/daffy/env"
folder. The installation takes approximately 45 minutes. Note that the
ocp folder is used.

/data/daffy/ocp/build.sh \<replace with cluster name here without
"-env.sh"\>

During installation, Daffy checks if all required APIs have been enabled
in your GCP project. Also, Daffy checks if required roles have been
assigned to the service account.

While not recommended, you could disable the role assignment validation
by commenting out the *gcpValidateRolesRequired* line in the
"/data/daffy/ocp/functions.sh" file if your service account has been
assigned to the "owner" role instead of individual roles.

*gcpInstallGCloud*

*gcpValidateAPIServicesRequired*

*#gcpValidateRolesRequired*

*gcpValidateConstraintsRequired*

*gcpValidateDNSZone*

When installation is completed, which takes about 35 minutes or longer,
you are provided with the cluster console login info. Note that Daffy
creates an additional user account, "ocpadmin".

*Here is the login info you can use for all services and **console**:\
\#####################################################################################\
Super User : kubeadmin\
Password : xxx\
Admin User : ocpadmin\
Password : xx\
OpenShift Web Console :
[https://console-openshift-console.apps.clustername.gcp.mydomainxxx.com\
](https://console-openshift-console.apps.mas16.gcp.xueopenlabs.com/)OC
Commandline : export KUBECONFIG=/var/ibm-ocp/mas16/kubeconfig\
OC Login command : oc login
[https://api.\<clustername\>.gcp.mydomainxxx.com:6443](https://api.mas16.gcp.xueopenlabs.com:6443/)
-u ocpadmin -p xxx --insecure-skip-tls-verify\
OC Client Download :
[https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.49\
](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.49)Install
Temp Files : /data/daffy/tmp/mas16/ocp\
openshift-install Dir : /data/daffy/tmp/mas16/ocp/ocp-install*

# Use NFS Server backed Storage for OpenShift

Red Hat has re-branded OpenShift Container Storage (OCS) as Red Hat Open
Data Foundation (ODF) since OpenShift 4.9 was released in 2021. While
use of ODF, currently in technical preview on GCP, is the storage path
going forward, a network file server (NFS) is a viable option for
applications requiring support for "Read Write Many" (RWX) storage mode,
for example, attached documents in Maximo. One benefit with NFS backed
storage is that applications and APIs off the OpenShift cluster can
access the shared storage when adequate permissions are granted. Refer
to Red Hat [OpenShift Data
Foundation](https://www.redhat.com/en/technologies/cloud-computing/openshift-data-foundation)
for more details.

## Create a Filestore Instance

You can create an NFS server in your GCP project from the cloud console
or using the gcloud command line. From the cloud console, search for
"Filestore" and then create an instance.

![Graphical user interface, text, application, email Description
automatically generated](media/image3.png){width="4.240088582677165in"
height="2.666363735783027in"}

Select the same VPC network for the cluster. Choose the instance type
(Basic, Enterprise, High Scale), storage type (HDD, SSD), and storage
capacity (ranging from 1TB to 64 TiB). For region and zone, select ones
that are in the same region and zone or close to your cluster. Enter the
file share name, and make a note it, along with the IP address for the
instance.

When the instance is created, a VPC network peering is automatically
created, which enables Filestore to access the VPC. On the other hand,
any Compute Engine VM or GKE cluster can access any Filestore instance
that\'s on the same VPC network. With the connectivity to the Filestore
the storage pods in OpenShift can read and write data it.

![Graphical user interface, text, application Description automatically
generated](media/image4.png){width="6.5in" height="5.216666666666667in"}

Filestore provides data encryption at rest and in transit, and comes
with two data recovery options, backups and snapshots. For high
availability, use Enterprise tier Filestore instances, which are
regional resources. Refer to GCP's [Filestore
documentation](https://cloud.google.com/filestore/docs/overview) for
more details.

## 

## Deploy a Dynamic Provisioning Storage Class

Google Filestore is a fully managed storage service and more scalable
than one single NFS server. When the Filestore instance is created and
file share enabled, the NFS server is ready without any configuration.
You can use the IP address and file share name to create OpenShift
persistent volume claims (PVC) and persistent volumes (PV) directly.

However, to create storages dynamically, it's necessary to deploy a
storage provision provider. For that we use the opensource project,
"[Kubernetes NFS Subdir External
Provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)."
NFS subdir external provisioner is an automatic provisioner that use
your existing and already configured NFS server to support dynamic
provisioning of PVs via PVCs. It is not recommended to use an [NFS
storage
provisioner](https://www.ibm.com/docs/en/mas-cd/continuous-delivery?topic=provisioner-setting-up-nfs-storage)
for workloads with heavy I/O performance requirements.

To deploy the storage provider, take the following steps, as outlined on
the
[support](https://www.ibm.com/support/pages/how-do-i-create-storage-class-nfs-dynamic-storage-provisioning-openshift-environment)
web page:

1.  Copy or download the yaml files from the /deploy folder in the
    github repo -- class.yaml, deployment.yaml, rbac.yaml,
    test-claim.yaml and test-pod.yaml. Note that the namespace "default"
    is used in the files, but you can change it.

2.  Replace the namespace in the deployment.yaml file. Also, replace the
    value of PROVISIONER_NAME in the spec section and change it from
    "k8s-sigs.io/nfs-subdir-external-provisioner" to something else, for
    example, "nfs-storage-provisioner". Also, replace the IP address and
    file share path in the env section and the volumes section.

3.  Replace the namespace in the rbac.yaml file. The namespace appears
    in a few places so do a global search and replace.

4.  Log in to the cluster and run the command lines to create a
    namespace, for example "nfs", storage class deployment, the storage
    class, and role based access (rbac).

> *Oc create namespace nfs*
>
> *oc create -f deployment.yaml*
>
> *oc create -f class.yaml*
>
> *oc create -f rbac.*yaml

5.  Create a role based on "hostmount-anyuid" which allows to provision
    the storage and assign the role to the service account, for example,
    "nfs-client-provisioner", which is specified in the rbac.yaml file.
    More info on the role at "[Managing security context
    constraints](https://docs.openshift.com/container-platform/4.11/authentication/managing-security-context-constraints.html)."

> *Oc create role use-scc-hostmount-anyuid --verb=use \--resource=scc
> --resource-name=hostmount-anyuid -n nfs*
>
> *ca dm policy add-role-to-user use-scc-hostmount-anyuid -z
> nfs-client-provisioner --role-namespace nfs -n nfs*

You can check the OpenShift cluster to make sure that the deployment is
successful. You can scale the deployment from the default 1 pod to 2 or
more.

![](media/image5.png){width="6.111459973753281in"
height="4.429501312335958in"}

You can create a PVC and a pod to test the storage class using
test-claim.yaml and test-pod.yaml files, or a sample yaml file that
includes PVC and pod in one file.

*apiVersion: v1*

*kind: PersistentVolumeClaim*

*metadata:*

*name: test-nfs-provisioner*

*namespace: nfs*

*spec:*

*storageClassName: nfs-client*

*accessModes:*

*- ReadWriteMany*

*resources:*

*requests:*

*storage: 10Mi*

*apiVersion: v1*

*kind: Pod*

*metadata:*

*labels:*

*run: ubuntu*

*name: ubuntu-test-nfs*

*namespace: nfs*

*spec:*

*containers:*

*- image: ubuntu*

*name: ubuntu*

*resources: {}*

*command: \["sleep", "3600"\]*

*volumeMounts:*

*- mountPath: /nfs*

*name: nfs-vol*

*volumes:*

*- name: nfs-vol*

*persistentVolumeClaim:*

*claimName: test-nfs-provisioner*

# The Infrastructure of OpenShift Deployment on GCP

With the installer and Daffy deployment options, the OpenShift cluster
is deployed in one region, and the master nodes and worker nodes of the
cluster are deployed in different zones in the same region on Google
Cloud.

## Network Topology

You can view a network topology of the deployment in your GCP project,
as shown below. The external load balancing is used to optimize network
traffic from different regions, whereas the internal load balancing is
used for internal network traffic within the OpenShift cluster.

When a Filestore instance is created, a network peering for the instance
is created automatically, enabling access between the instance and pods
in the cluster.

![A picture containing radar chart Description automatically
generated](media/image6.png){width="3.7197451881014874in"
height="2.7210575240594927in"}

## 

## Cloud Services in OpenShift Deployment

Depending on the size of the OpenShift cluster, virtual instances of
Compute Engine for master nodes and worker nodes along with persistent
disks are created. A cloud DNS is required prior to the deployment.
After the deployment, several GCP cloud services are created and
utilized, including one VPC network with two subnets, external and
internal load balancers, cloud router, internal and external IP
addresses, firewall rules, cloud routes and cloud NAT.

You can find other deployment options on GCP in [Red Hat
documentation](https://docs.openshift.com/container-platform/4.11/installing/installing_gcp/installing-restricted-networks-gcp.html),
for example, installing OpenShift into an existing VPC, installing a
private cluster, installing a cluster using Deployment Manager
templates, or installing a cluster using user provisioned infrastructure
(UPI).

## Infrastructure Architecture

The diagram below illustrates the high-level infrastructure of the
OpenShift deployment on GCP.

![A picture containing graphical user interface Description
automatically generated](media/image7.png){width="6.5in"
height="5.00625in"}

# 

# Install Maximo Application Suite

IBM Maximo Application Suite (MAS) enables you to monitor, manage and
maintain your

assets in a single platform and streamline your operations. With latest
releases 8.x, you can deploy MAS in the cloud.

Installing MAS takes two steps: install MAS Core first and then activate
individual applications. To install MAS core, you run an Ansible
playbook, which automates the installation process. Please refer to the
document "[OneClick Install for MAS
Core](https://ibm-mas.github.io/ansible-devops/playbooks/oneclick-core/)"
for more details.

## Define Environment Variables

It's worth noting that installing MAS Core requires IBM entitlement key
and MAS licensing information and define several environment variables.
Note that we use the NFS storage class.

*export MAS_INSTANCE_ID=inst1*

*export INSTANCE=inst1*

*#cataglog source: ibm-opertor - catalog / release -mas-curated-cat*

*export COMMON_SERVICES_CATALOG_SOURCE=release-mas-curated-cat*

*export SLS_CATALOG_SOURCE=release-mas-curated-cat*

*export SLS_LICENSE_ID=xxx*

*export SLS_LICENSE_FILE=\~/masconfig/entitlement.lic*

*export PROJECT_CPFS_OPS=ibm-common-services*

*export PROJECT_CPD_OPS=cpd-operators*

*export PROJECT_CATSRC=openshift-marketplace*

*export PROJECT_CPD_INSTANCE=cpd-instance*

*export PROJECT_TETHERED=cpd-tethered*

*export IBM_ENTITLEMENT_KEY=xxx*

*export IBM_ENTITLEMENT_USER=cp*

*export IBM_ENTITLEMENT_SERVER=cp.icr.io*

*#export ARTIFACTORY_USERNAME=xxx*

*#export ARTIFACTORY_APIKEY=xxx*

*export MAS_ICR_CP=cp.icr.io/cp*

*export MAS_ICR_CPOPEN=icr.io/cpopen*

*export MAS_ENTITLEMENT_USERNAME=cp*

*export MAS_CONFIG_DIR=\~/masconfig*

*export MAS_CATALOG_SOURCE=release-mas-curated-cat*

*export MAS_CHANNEL=8.x*

*export UDS_CONTACT_EMAIL=xxx@xxx.com*

*export UDS_CONTACT_FIRSTNAME=xxx*

*export UDS_CONTACT_LASTNAME=xxx*

*export PROMETHEUS_STORAGE_CLASS=nfs-client*

*export PROMETHEUS_ALERTMGR_STORAGE_CLASS=nfs-client*

*export PROMETHEUS_USERWORKLOAD_STORAGE_CLASS=nfs-client*

*export GRAFANA_INSTANCE_STORAGE_CLASS=nfs-client*

*export MONGODB_STORAGE_CLASS=nfs-client*

*export DB2_DATA_STORAGE_CLASS=nfs-client*

*export KAFKA_STORAGE_CLASS=nfs-client*

*export UDS_STORAGE_CLASS=nfs-client*

You can define and store the environment variables in a file, for
example "mas_var.sh", and then run the command line "*source
mas_var.sh*" to export them in the current terminal session.

## Install MAS Core

Login to the OpenShift cluster and install the MAS catalog by running
the command line:

*oc apply -f release-mas-curated-cat.yaml*

The *release-mas-curated-cat.yaml* file includes the image file in IBM
Container Registry.

*apiVersion: operators.coreos.com/v1alpha1*

*kind: CatalogSource*

*metadata:*

*creationTimestamp: \'2022-08-24T18:25:59Z\'*

*generation: 1*

*name: release-mas-curated-cat*

*namespace: openshift-marketplace*

*resourceVersion: \'1255203\'*

*uid: b3f9214e-2e5c-4e8b-ab53-68052d0a5099*

*spec:*

*displayName: release-mas-curated-cat*

*image: \>-*

*cp.icr.io/cpopen/ibm-operator-catalog@sha256:d1ccb02f6d6a74736fa59714c266069c78f3fb366353cdbcaee635a5720e6f27*

*publisher: \'\'*

*sourceType: grpc*

*status:*

*connectionState:*

*address: \'release-mas-curated-cat.openshift-marketplace.svc:50051\'*

*lastConnect: \'2022-08-25T14:14:52Z\'*

*lastObservedState: READY*

*registryService:*

*createdAt: \'2022-08-24T18:26:18Z\'*

*port: \'50051\'*

*protocol: grpc*

*serviceName: release-mas-curated-cat*

*serviceNamespace: openshift-marketplace*

Then, run the command line to execute the Ansible playbook. Download
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
and install it on your Bastion machine if you have not done so.

*ansible-playbook ibm.mas_devops.oneclick_core*

When the installation is completed successfully, you are provided with
the following login information.

*TASK \[ibm.mas_devops.suite_verify : gencfg-wsl : Watson Studio
Installation Summary:\]
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\**

*ok: \[localhost\] =\> {*

*\"msg\": \[*

*\"Maximo Application Suite is Ready, use the superuser credentials to
authenticate\",*

*\"Admin Dashboard \...
https://admin.inst1.apps.\<clustername\>.gcp.mydomainxxx.com\",*

*\"Username \...\...\.... eXTbAASkwXK4gJxfHoeWQi1HSu0pADPr\",*

*\"Password \...\...\.... cwtq9z5ApGhZF8K1hROcsLXkaL7zcPQE\" \]}*

# Activate MAS Manage

MAS Manage is usually the first application you activate, which involves
three key steps:

-   Deploy IBM Cloud Pak for Data

-   Create a DB2 Warehouse instance and configure database connection

-   Activate Manage and other applications

Alternatively, you can run an Ansible Playbook to create a DB2
Warehouse, configure database connection and activate MAS Manage all in
one step. Please refer to the document, "[Install Manage
Application](https://ibm-mas.github.io/ansible-devops/playbooks/oneclick-manage/)".

## Deploy IBM Cloud Pak for Data

Maximo Application Suite (MAS) requires database to store data using IBM
DB2, DB2 Warehouse, or other types of databases. To use IBM DB2 or DB2
Warehouse, you can install DB2 or DB2 Warehouse directly, or deploy IBM
Cloud Pak for Data (CP4D) which comes with DB2 and DB2 Warehouse.

To install CP4D version 4.5.x or higher, you can refer to the document,
"[Installing Cloud Pak for
Data](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.5.x?topic=installing)."
However, it is highly recommended that you use Daffy. Add the two
environment variables to the same xxx-env.sh file.

*CP4D_VERSION="4.5.3"*

*CP4D_ENABLE_SERVICE_DB2_WAREHOUSE=true*

Run the command line in the /data/daffy/env folder. Note that the cp4d
folder is used.

*/data/daffy/cp4d/build.sh \<replace with cluster name here without
"-env.sh"\>*

During the build process, Daffy runs the command to apply two storage
classes.

*/data/daffy/tmp/cpdcli/cpd-cli-linux-EE-11.0.0-20/cpd-cli manage
apply-cr \--components=cpfs,scheduler,cpd_platform \--release=4.5.0
\--cpd_instance_ns=cpd-instance
\--block_storage_class=ocs-storagecluster-ceph-rbd
\--file_storage_class=ocs-storagecluster-cephfs
\--license_acceptance=true*

When the installation is completed, which takes approximately 1 hour and
45 minutes or longer, you are provided with the following CP4D control
plane login information.

*Here is the login info for the CP4D Navigator console*

*\################################################################*

*Super User : admin*

*Password : rvGcd27nHWRd*

*CP4D Web Console :    *

## 

## Create DB2 Warehouse Instance

First, log in to the CP4D control plane and create a DB2 Warehouse
instance.

![Graphical user interface, application Description automatically
generated](media/image8.png){width="6.5in" height="5.091666666666667in"}

Follow the steps of the database instance creation from the portal:

-   Type: Select the DB2 Warehouse version installed.

-   Configure: For storage structure, select "Separate locations for all
    data". For production, select "Deploy database on dedicated nodes".

-   Advanced configuration: For workload, select "Operational
    Analytics".

-   System storage: Select the storage class, for example, "nfs-client".
    Choose the data size.

-   User storage: Select the storage class, for example, "nfs-client".
    Choose the data size. For access mode, select "ReadWriteOnce" or
    "ReadWriteMany".

-   Backup storage: Select the storage class, for example, "nfs-client".
    Choose the data size.

-   Transaction logs storage: Select the storage class, for example,
    "nfs-client". Choose the data size.

Create the database instance. Make a note of the host name and the JDBC
Connection URL (SSL).

*jdbc:db2://\<CLUSTER_ACCESSIBLE_IP\>:32319/BLUDB:user=-;password=\<password\>;securityMechanism=9;sslConnection=true;encryptionAlgorithm=2*

![Graphical user interface, text, application Description automatically
generated](media/image9.png){width="4.23055227471566in"
height="2.3466918197725284in"}

Click the "Download SSL Certificate" button to download the certificate.
You will need the info when configuring the database connection for MAS.

Create a new user, for example "maximo" and assign both the
"administrator" and "user" roles to it.

## Create Database Schema for Manage

Configure DB2 Warehouse and create MAS schema by running a set of
scripts. Please refer to the document, "[Configuring Db2
Warehouse](https://www.ibm.com/docs/en/maximo-manage/continuous-delivery?topic=deployment-configuring-db2-warehouse)".

[]{#_Toc118371200 .anchor}

## Configure Database Connection

Next, log into the MAS console and select the Administration page (the
gear icon) at upper right corner. Select Configuration from the
navigation menu on the left side of the page. You can configure database
connection at the System level, Workspace level and other levels or
scopes. Please refer to the document, "[Configuration
Settings](https://www.ibm.com/docs/en/mas85/8.5.0?topic=administering-configuring-suite)".

To configure database connection for the workspace, select the workspace
name, for example "masdev". Copy and paste the connection string you
noted from the previous step. Remove the user and password from it.
Replace CLUSTER_ACCESSIBLE_IP with the host name from the previous step.

*jdbc:db2://\<hostname\>:32319/BLUDB:securityMechanism=9;sslConnection=true;encryptionAlgorithm=2;*

Enter the user credentials you created at the previous step. Open the
certificate you downloaded, copy and paste the text to the Certificates
box and confirm/save it. Save the connection string. It takes a few
minutes to update the connection string in MAS.

![Graphical user interface, text, application, Teams Description
automatically generated](media/image10.png){width="5.618468941382327in"
height="2.9719061679790024in"}

## Activate MAS Manage

From the MAS console, select Workspace, activate "Manage" and deploy
"Optimizer". This process can take some time.

![Graphical user interface, application Description automatically
generated](media/image11.png){width="6.5in"
height="4.090972222222222in"}

When the activation is completed, the workspace and Manage show "Ready".

Next, create a new user or reset password for an existing user such as
"maxadmin". Then log out the admin account.

## Launch MAS Manage

You can find the web URL for the Manage application using the oc command
or from the OpenShift cluster console.

-   Run the oc command: *oc get routes -n \<install instance
    id\>-manage*

```{=html}
<!-- -->
```
-   From the cluster console, navigate to Networking, select the
    namespace \<install instance id\>-manage, and click on Routes.

To access the Manage application, add /maximo to the end of the route,
or replace "manage" with "home" in the URL without adding "/maximo".

*https://masdev.manage.\<install instance id\>.apps.\<cluster
name\>.gcp.mydomainxxx.com/**maximo***

*https://masdev.**home**.\<install instance id\>.apps.\<cluster
name\>.gcp.mydomainxxx.com*

Log in with a user account using the revised URL. Click on the
Applications icon (9 dots) at upper right corner. Select Manage to see
the dashboard.

![Graphical user interface, application, email, website Description
automatically generated](media/image12.png){width="5.18065179352581in"
height="2.6960422134733157in"}

# 

# Storages for OpenShift and Maximo

OpenShift Data Foundation (ODF) is in technical preview on GCP and
therefore NFS backed storage is used to provide the RWX storage mode for
Maximo. In the test environment, the custom storage class, "nfs-client",
is used for Maximo only, whereas the built-in storage classes are used
for OpenShift.

## OpenShift Storage Classes

With the installer option, three storage classes are available for the
cluster: standard, standard-csi, and pd-ssd. They support "read write
once" mode (RWO).

![Graphical user interface, application Description automatically
generated](media/image13.png){width="6.5in"
height="2.4256944444444444in"}

Daffy adds three storage classes, ocs-storagecluster-cephfs,
ocs-storagecluster-ceph-rbd and openshift-storage.noobaa.io when the
environemnt variable, *OCP_CREATE_OPENSHIFT_CONTAINER_STORAGE,* is set
to "true". They support both "read write once" (RWO) and "read write
many" (RWX) modes.

## Maximo DB2WH Storage

By default, OpenShift (version 4.8) uses a few storage classes in
various namespaces. The table below summarizes the usage of storage
classes from the console page of the OpenShift PVCs.

+---------------------------+------------------------------------------+
| **Storage Class**         | **Namespace**                            |
+===========================+==========================================+
| standard-csi              | ibm-common-services                      |
|                           |                                          |
|                           | grafana                                  |
|                           |                                          |
|                           | mongoce                                  |
|                           |                                          |
|                           | openshift-monitoring                     |
|                           |                                          |
|                           | openshift-user-workload-monitoring       |
+---------------------------+------------------------------------------+
| pd-ssd                    | openshift-storage                        |
+---------------------------+------------------------------------------+
| ocs-storagecluster-cephfs | cpd-instance (CP4D)                      |
+---------------------------+------------------------------------------+
| oc                        | openshift-storage                        |
| s-storagecluster-ceph-rbd |                                          |
|                           | cpd-instance (CP4D)                      |
+---------------------------+------------------------------------------+
| nfs-client                | cpd-instance (CP4D)                      |
+---------------------------+------------------------------------------+

For Maximo, DB2WH data, backup, temps, active logs and meta are in
storages provisioned by the "nfs-client" storage class.

![Graphical user interface Description automatically
generated](media/image14.png){width="6.5603116797900265in"
height="3.0649814085739284in"}

# Troubleshoot MAS Manage Activation

## Check the database connection string in Pods

Select project \<install instance id\>-manage from the cluster console.
Search Pods with the name "maxinst". Open the pod and a Terminal
session. Run the command line in the directory,
"/opt/IBM/SMP/maximo/tools/maximo/internal".

*./querycount.sh -qcount -tdummy_table*

In the example, we find that a ";" instead of ":" was used in the
connection string. Fix the connection string in the configuration
section on the MAS administration page.

*jdbc:db2://\<hostname\>:32319/BLUDB**;**securityMechanism=9;sslConnection=true;encryptionAlgorithm=2;*

![](media/image15.png){width="6.344966097987752in"
height="2.756940069991251in"}

## Check "ManageWorkspace" custom resource

Go to CustomResourceDefinitions under the Administration section on the
cluster console. Search "workspace". Select "ManageWorksapce". Go to the
Instances tab and select the instance named "\<install instance
id\>-\<workspace name\>". Check if the binding is set at workspace or
workspace-application and update it if necessary.\
\
![](media/image16.png){width="6.5in" height="3.03125in"}

## Check "jdbc" custom resource

Go to CustomResourceDefinitions under the Administration section on the
cluster console. Search "jdbc".

There are likely two resources or more in the \<install instance
id\>-core namespace, one for system and one for workspace. Open the
System resource and check the YAML detail.

![Graphical user interface, text Description automatically
generated](media/image17.png){width="6.5in"
height="3.0076388888888888in"}

# Shut Down the OpenShift Cluster

For dev/test environment, you may want to shut down the OpenShift
environment to reduce costs. You can run the command line below to shut
down the cluster gracefully.

*for node in \$(oc get nodes -o
jsonpath=\'{.items\[\*\].metadata.name}\'); do oc debug node/\${node}
\-- chroot /host shutdown -h 1; done *

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

# Next Steps

In the document we provide basic instructions on how to install
OpenShift, storage, database and Maximo Application Suite (MAS) on
Google Cloud and some troubleshooting tips. Please refer to other
documents on advanced topics such as how to install OpenShift in
existing environments or private clusters, how to extend OpenShift with
additional nodes and how to deploy MAS based on a reference
architecture.
