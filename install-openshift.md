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

```
./openshift-install create cluster \--dir \<installation_directory\>
```

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

```
./openshift-install create install-config \--dir <installation_directory\>

./openshift-install create cluster \--dir \<installation_directory\>
```

Sample install-config.yaml file

```
apiVersion: v1
baseDomain: gcp.xxx.com
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform: {}
  replicas: 3
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform: {}
  replicas: 3
metadata:
  creationTimestamp: null
  name: <replace cluster name here>
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  gcp:
    projectID: <replace gcp project name here>
    region: <e.g. us-east1>
publish: External
pullSecret: 'replace redhat pull secrets here'
sshKey: <replace ssh key here>
```

When the installation is completed, you are provided with the following
cluster console login info.

```
INFO To access the cluster as the system:admin user when using 'oc',
run 'export KUBECONFIG=/home/\<username\>/Downloads/xxx/auth/kubeconfig'

INFO Access the OpenShift web-console here:

https://console-openshift-console.apps.\<clustername\>.gcp.mydomainxxx.com

INFO Login to the console with user: \"kubeadmin\", and password: \"nVwnr-ma84g-kUob3-766B8\"
```

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

```
DAFFY_UNIQUE_ID="replace with email address here"
DAFFY_DEPLOYMENT_TYPE=Enablement
BASE_DOMAIN="gcp.mydomainxxx.com"
CLUSTER_NAME="replace with cluster name here for example mycluster"
OCP_INSTALL_TYPE="gcp-ipi"
OCP_RELEASE="4.8.49"
VM_TSHIRT_SIZE="Large"
GCP_PROJECT_ID="replace with GCP project name here"
GCP_REGION="us-east1"
OCP_CREATE_OPENSHIFT_CONTAINER_STORAGE="true"
CP4D_VERSION="4.5.3"
CP4D_ENABLE_SERVICE_DB2_WAREHOUSE=true
```

To create an OpenShift, run the command line in the "/data/daffy/env"
folder. The installation takes approximately 45 minutes. Note that the
ocp folder is used.

```
/data/daffy/ocp/build.sh <replace with cluster name here without "-env.sh"
```

During installation, Daffy checks if all required APIs have been enabled
in your GCP project. Also, Daffy checks if required roles have been
assigned to the service account.

While not recommended, you could disable the role assignment validation
by commenting out the *gcpValidateRolesRequired* line in the
"/data/daffy/ocp/functions.sh" file if your service account has been
assigned to the "owner" role instead of individual roles.

```
gcpInstallGCloud
gcpValidateAPIServicesRequired
#gcpValidateRolesRequired
gcpValidateConstraintsRequired
gcpValidateDNSZone
```

When installation is completed, which takes about 35 minutes or longer,
you are provided with the cluster console login info. Note that Daffy
creates an additional user account, "ocpadmin".

```
Here is the login info you can use for all services and console:
#####################################################################################
Super User            :      kubeadmin
Password              :      xxx
Admin User            :      ocpadmin
Password              :      xx
OpenShift Web Console :      https://console-openshift-console.apps.clustername.gcp.mydomainxxx.com
OC Commandline        :      export KUBECONFIG=/var/ibm-ocp/mas16/kubeconfig
OC Login command      :      oc login https://api.<clustername>.gcp.mydomainxxx.com:6443 -u ocpadmin -p xxx –insecure-skip-tls-verify
OC Client Download    :      https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.49
Install Temp Files    :      /data/daffy/tmp/mas16/ocp
openshift-install Dir :      /data/daffy/tmp/mas16/ocp/ocp-install
```

[Back to ReadMe page](README.MD)