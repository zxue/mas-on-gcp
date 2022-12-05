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

```
export MAS_INSTANCE_ID=inst1
export INSTANCE=inst1
#cataglog source: ibm-opertor - catalog / release -mas-curated-cat
export COMMON_SERVICES_CATALOG_SOURCE=release-mas-curated-cat
export SLS_CATALOG_SOURCE=release-mas-curated-cat
export SLS_LICENSE_ID=xxx
export SLS_LICENSE_FILE=~/masconfig/entitlement.lic
export PROJECT_CPFS_OPS=ibm-common-services        
export PROJECT_CPD_OPS=cpd-operators
export PROJECT_CATSRC=openshift-marketplace
export PROJECT_CPD_INSTANCE=cpd-instance
export PROJECT_TETHERED=cpd-tethered
export IBM_ENTITLEMENT_KEY=xxx
export IBM_ENTITLEMENT_USER=cp
export IBM_ENTITLEMENT_SERVER=cp.icr.io
#export ARTIFACTORY_USERNAME=xxx
#export ARTIFACTORY_APIKEY=xxx
export MAS_ICR_CP=cp.icr.io/cp
export MAS_ICR_CPOPEN=icr.io/cpopen
export MAS_ENTITLEMENT_USERNAME=cp
export MAS_CONFIG_DIR=~/masconfig
export MAS_CATALOG_SOURCE=release-mas-curated-cat
export MAS_CHANNEL=8.x
export UDS_CONTACT_EMAIL=xxx@xxx.com
export UDS_CONTACT_FIRSTNAME=xxx
export UDS_CONTACT_LASTNAME=xxx
export PROMETHEUS_STORAGE_CLASS=nfs-client
export PROMETHEUS_ALERTMGR_STORAGE_CLASS=nfs-client
export PROMETHEUS_USERWORKLOAD_STORAGE_CLASS=nfs-client
export GRAFANA_INSTANCE_STORAGE_CLASS=nfs-client
export MONGODB_STORAGE_CLASS=nfs-client
export DB2_DATA_STORAGE_CLASS=nfs-client
export KAFKA_STORAGE_CLASS=nfs-client
export UDS_STORAGE_CLASS=nfs-client
```

You can define and store the environment variables in a file, for
example "mas_var.sh", and then run the command line "*source
mas_var.sh*" to export them in the current terminal session.

## Install MAS Core

Login to the OpenShift cluster and install the MAS catalog by running
the command line:

```
oc apply -f release-mas-curated-cat.yaml
```

The *release-mas-curated-cat.yaml* file includes the image file in IBM
Container Registry.

```
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  creationTimestamp: '2022-08-24T18:25:59Z'
  generation: 1
  name: release-mas-curated-cat
  namespace: openshift-marketplace
  resourceVersion: '1255203'
  uid: b3f9214e-2e5c-4e8b-ab53-68052d0a5099
spec:
  displayName: release-mas-curated-cat
  image: >-
    cp.icr.io/cpopen/ibm-operator-catalog@sha256:d1ccb02f6d6a74736fa59714c266069c78f3fb366353cdbcaee635a5720e6f27
  publisher: ''
  sourceType: grpc
status:
  connectionState:
    address: 'release-mas-curated-cat.openshift-marketplace.svc:50051'
    lastConnect: '2022-08-25T14:14:52Z'
    lastObservedState: READY
  registryService:
    createdAt: '2022-08-24T18:26:18Z'
    port: '50051'
    protocol: grpc
    serviceName: release-mas-curated-cat
    serviceNamespace: openshift-marketplace
```

Then, run the command line to execute the Ansible playbook. Download
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
and install it on your Bastion machine if you have not done so.

```
ansible-playbook ibm.mas_devops.oneclick_core
```

When the installation is completed successfully, you are provided with
the following login information.

```
TASK [ibm.mas_devops.suite_verify : gencfg-wsl : Watson Studio Installation Summary:] 
********************************************
ok: [localhost] => {
    "msg": [
        "Maximo Application Suite is Ready, use the superuser credentials to authenticate",
        "Admin Dashboard ... https://admin.inst1.apps.<clustername>.gcp.mydomainxxx.com",
        "Username .......... eXTbAASkwXK4gJxfHoeWQi1HSu0pADPr",
        "Password .......... cwtq9z5ApGhZF8K1hROcsLXkaL7zcPQE"    ]}
```

[Back to ReadMe page](README.MD)
