---
title: PowerScale CSI
Desription: Isilon Readme
---

# PowerScale CSI

## Installation Overview
Installation in a Kubernetes cluster should be done using the scripts within the `dell-csi-helm-installer` directory. 

The controller section of the Helm chart installs the following components in a Stateful Set:

* CSI Driver for PowerScale
* Kubernetes Provisioner, which provisions the provisioning volumes
* Kubernetes Attacher, which attaches the volumes to the containers
* Kubernetes Snapshotter, which provides snapshot support
* Kubernetes Resizer, which provides resize support

The node section of the Helm chart installs the following component in a Daemon Set:

* CSI Driver for PowerScale
* Kubernetes Registrar, which handles the driver registration

### Prerequisites

Before you install CSI Driver for PowerScale, verify the requirements that are mentioned in this topic are installed and configured.

#### Requirements

* Install Kubernetes.
* Configure Docker service
* Install Helm v3
* Install volume snapshot components
* Deploy PowerScale driver using Helm

**Note:** There is no feature gate that needs to be set explicitly for csi drivers from 1.17 onwards. All the required feature gates are either beta/GA.

## Configure Docker service

The mount propagation in Docker must be configured on all Kubernetes nodes before installing CSI Driver for PowerScale.

### Procedure

1. Edit the service section of */etc/systemd/system/multi-user.target.wants/docker.service* file as follows:

    ```
    [Service]
    ...
    MountFlags=shared
    ```
    
2. Restart the Docker service with systemctl daemon-reload and

    ```
    systemctl daemon-reload
    systemctl restart docker
    ```

## Install volume snapshot components

### Install Snapshot Beta CRDs
To install snapshot crds specify `--snapshot-crd` flag to driver installation script `dell-csi-helm-installer/csi-install.sh` during driver installation

### [Install Common Snapshot Controller](<https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-cis-volume-snapshot-beta/#how-do-i-deploy-support-for-volume-snapshots-on-my-kubernetes-cluster>), if not already installed for the cluster

    ```
    kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
    kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
    ```

## Install CSI Driver for PowerScale

Install CSI Driver for PowerScale using this procedure.

*Before you begin*
 * You must clone the source [git repository](https://github.com/dell/csi-isilon), ready for below procedure.
 * In the `dell-csi-helm-installer` directory, there should be two shell scripts, *csi-install.sh* and *csi-uninstall.sh*. These scripts handle some of the pre 
   and post operations that cannot be performed in the helm chart.

Procedure

1. Collect information from the PowerScale Systems like IP address, username  and password. Make a note of the value for these parameters as they must be entered in the secret.yaml 
   and values file.

2. Copy the helm/csi-isilon/values.yaml into a new location with name say *my-isilon-settings.yaml*, to customize settings for installation.

3. Edit *my-isilon-settings.yaml* to set the following parameters for your installation:
    
    The following table lists the primary configurable parameters of the PowerScale driver helm chart and their default values. More detailed information can be
   found in the  [`values.yaml`](helm/csi-isilon/values.yaml) file in this repository.
    
    | Parameter | Description | Required | Default |
    | --------- | ----------- | -------- |-------- |    
    | isiIP | "isiIP" defines the HTTPs endpoint of the PowerScale OneFS API server | true | - |
    | isiPort | "isiPort" defines the HTTPs port number of the PowerScale OneFS API server | false | 8080 |
    | isiInsecure | "isiInsecure" specifies whether the PowerScale OneFS API server's certificate chain and host name should be verified. | false | true |
    | isiAccessZone | The name of the access zone a volume can be created in | false | System |
    | volumeNamePrefix | "volumeNamePrefix" defines a string prepended to each volume created by the CSI driver. | false | k8s |   
    | enableDebug | Indicates whether debug level logs should be logged | false | true |
    | verbose | Indicates what content of the OneFS REST API message should be logged in debug level logs | false | 1 |
    | enableQuota | Indicates whether the provisioner should attempt to set (later unset) quota on a newly provisioned volume. This requires SmartQuotas to be  enabled.| false | true |
    | noProbeOnStart | Indicates whether the controller/node should probe during initialization | false | false |
    | isiPath | The default base path for the volumes to be created, this will be used if a storage class does not have the IsiPath parameter specified| false
    | /ifs/data/csi |
    | autoProbe |  Enable auto probe. | false | true |
    | nfsV3 | Specify whether to set the version to v3 when mounting an NFS export. If the value is "false", then the default version supported will be used (i.e. the mount command 
      will not explicitly specify "-o vers=3" option) | false | false |
    | ***Storage Class parameters*** | Following parameters are related to Storage Class |
    | name | "storageClass.name" defines the name of the storage class to be defined. | false | isilon |
    | isDefault | "storageClass.isDefault" defines whether the primary storage class should be the default. | false | true |    
    | reclaimPolicy | "storageClass.reclaimPolicy" defines what will happen when a volume is removed from the Kubernetes API. Valid values are "Retain" and "Delete".| false | Delete |
    | accessZone | The Access Zone where the Volume would be created | false | System |
    | AzServiceIP | Access Zone service IP if different from isiIP, specify here and refer in storageClass | false |  |
    | rootClientEnabled |  When a PVC is being created, it takes the storage class' value of "storageclass.rootClientEnabled"| false | false |

    Note: User should provide all boolean values with double quotes. This applicable only for my-isilon-settings.yaml. Ex: "true"/"false"

4. Create namespace

    Run `kubectl create namespace isilon` to create the isilon namespace. Specify the same namespace name while installing the driver. 
    Note: CSI PowerScale also supports installation of driver in custom namespace.

5. Create a secret file for the OneFS credentials by editing the secret.yaml present under helm directory. Replace the values for the username and password parameters.

    Use the following command to convert username/password to base64 encoded string
    ```
    echo -n 'admin' | base64
    echo -n 'password' | base64 
    ```  
    Run `kubectl create -f secret.yaml` to create the secret.

6. Install OneFS CA certificates by following the instructions from next section, if you want to validate OneFS API server's certificates. If not, create an empty secret using the 
   following command and empty secret should be created for the successful CSI Driver for DELL EMC Powerscale installation.
    ```
    kubectl create -f emptysecret.yaml
    ```

7. Install CSI driver for PowerScale by following the instructions from [README](dell-csi-helm-installer/README.md)

## Certificate validation for OneFS REST API calls 

The CSI driver exposes an install parameter 'isiInsecure' which determines if the driver
performs client-side verification of the OneFS certificates. The 'isiInsecure' parameter is set to true by default and the driver does not verify the OneFS certificates.

If the isiInsecure is set to false, then the secret isilon-certs must contain the CA certificate for OneFS. 
If this secret is an empty secret, then the validation of the certificate fails, and the driver fails to start.

If the isiInsecure parameter is set to false and a previous installation attempt to create the empty secret, then this secret must be deleted and re-created using the CA certs.
If the OneFS certificate is self-signed, then perform the following steps:

### Procedure

1. To fetch the certificate, run `openssl s_client -showcerts -connect <OneFS
IP> </dev/null 2>/dev/null | openssl x509 -outform PEM > ca_cert.pem`

2. To create the secret, run `kubectl create secret generic isilon-certs --from-file=ca_cert.pem -n isilon`
