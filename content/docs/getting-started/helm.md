---
title: "Installation using Common helm Installer"
linkTitle: "Common helm Installation"
weight: 2
Description: >
  Installation Scripts for Dell EMC CSI drivers using common helm installer
---

## Installation scripts and README.md files
The scripts and documentation are located within the [installer](installer) directory. These scripts are designed
to be placed within the CSI Driver repos in a directory called `/dell-csi-helm-installer` and require the driver helm chart
to be located within the `/helm` directory of the same repo.

## Copying the scripts to target driver repos
A helper script is available that will copy the common installer scripts and docuemtnation to each driver. This script can be found at `util/update-driver.sh`
and will perform the following steps for each driver:
* Clone the driver repo
* Either checkout or create a branch for the changes to be applied to. If the user supplied branch name exists, it will be used. If it does not exist,
one will be created.
* Copy the common installer scripts to the target branch
* Copy the documentation to the target branch. While copying, it will substitute:
*   The driver arrays formal name (PowerMax, PowerFlex, PowerStore, PowerScale, Unity) for any text matching \_\_PROPERPRODUCTNAME\_\_
*   The driver arrays informal name, always in lower case (powermax, vxflexos, powerstore, isilon, unity) for text matching \_\_LOWERPRODUCTNAME\_\_
* After each file is copied, a `git diff` will be performed so the user can inspect the changes and validate them
* Once all files are copied, the files will be added to a `git commit` with a commit message that has been supplied by the user.
* At this stage, if there are any changes, the user will be prompted if they would like to `git push` the changes to the git remote
* If the `git push` is successful, the script can automatically create a PR in the target repository, if one does not exist. If the user elects to not create a PR, one can be created manually.

A video demonstrating the use of the `util/update-driver.sh` script has been made, It can be accessed (Passcode: ftju8Np*) at https://Dell.zoom.us/rec/share/qAOiNw3PSy1TtHBuzI9R6hN3W1PfZr_w8rKVk5X8Zg1axQqjYNYKG89mabbIkeef.9pLpkZfdJ3kEGKq4

This script will create a temporary working directory that must be manually cleaned up. The location of this directory will be displayed at the end of the script.

Help is available for this script

```
[dell-csi-helm-installer]# update-driver.sh  -h
Help for update-driver.sh
This script will copy the Common Helm Installer scripts to a branch within each driver and push them to the remote

Usage: update-driver.sh options...
  --directory[=]<directory>            Directory to copy files from.
                                       Default is "/dell/git/dell-csi-helm-installer/installer"
  --branch[=]<branch>                  Branch to copy to. This branch will be created if it does not exist at the remote.
                                       Default is "feature/update-common-installer-scripts"
  --commit[=]<commit message>          Git commit message.
                                       Default is "Addressing installation review comments across all drivers"
  --assume-yes                         Assume 'yes' for any questions. Default is not to assume this.
  -h                                   Help
```
