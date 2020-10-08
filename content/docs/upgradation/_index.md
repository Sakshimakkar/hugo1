---
title: Upgradation
tags: 
 - upgrade
 - csi-driver
weight: 1
Description: Upgrades in various drivers
---

## CSI-PowerScale
### Upgrade CSI Driver for DELL EMC PowerScale from previous versions
The CSI Driver for Dell EMC PowerScale prior to v1.2.0 supports only Helm 2. Users can upgrade the driver using Helm 2 or Helm 3.
Use one of the following two approaches to upgrade the CSI Driver for Dell EMC PowerScale:
* Upgrade CSI Driver for Dell EMC PowerScale v1.1.0 to v1.2.0 using Helm 2
* Migrate from Helm 2 to Helm 3

#### Upgrade CSI Driver for Dell EMC PowerScale v1.1.0 to v1.2.0 using Helm 2
1. Get the latest code from github.com/dell/csi-isilon (v1.2.0)
2. Uninstall the existing v1.1.0 driver using uninstall.sh under csi-isilon/helm
3. Prepare myvalues.yaml
4. Run the ./install.sh command to upgrade the driver
5. List the pods with the following command (to verify the status)

   `kubectl get pods -n isilon`

#### Migrating from Helm 2 to Helm 3 (applicable for CSI Driver for Dell EMC PowerScale v1.2.0 or higher)
1. Get the latest code from github.com/dell/csi-isilon by executing the following command.

    `git clone -b <VERSION> https://github.com/dell/csi-isilon.git`
    where <VERSION> is the version of CSI Driver for Dell EMC PowerScale
2. Uninstall the CSI Driver for Dell EMC PowerScale using the uninstall.sh script under csi-isilon/helm using Helm 2.
3. Go to https://helm.sh/docs/topics/v2_v3_migration/ and follow the instructions to migrate from Helm 2 to Helm 3.
4. Once Helm 3 is ready, install the CSI Driver for Dell EMC PowerScale using install.sh script under csi-isilon/helm.
5. List the pods with the following command (to verify the status)

   `kubectl get pods -n isilon`

## CSI-Unity
### Upgrade CSI Driver for Unity

Preparing myvalues.yaml is the same as explained above.

To upgrade the driver from csi-unity v1.2.1 in k8s 1.16 to csi-unity 1.3 in k8s 1.17:
1. Remove all volume snapshots, volume snapshot content and volume snapshot class objects.
2. Upgrade the Kubernetes version to 1.17 first before upgrading CSI driver.
3. Uninstall existing driver.
4. Uninstall alpha snapshot CRDs.
5. Verify all pre-reqs to install csi-unity v1.3 are fulfilled.
6. Install the driver using installation steps from [here](#install-csi-driver-for-unity)

**Note**: User has to re-create existing custom-storage classes (if any) according to latest (v1.3) format.


