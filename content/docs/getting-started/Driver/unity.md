---
title: Unity CSI
Desription: Unity Readme
---

# Unity CSI

## Test deploying a simple pod with Unity storage
Test the deployment workflow of a simple pod on Unity storage.

1. **Verify Unity system for Host**

    After helm deployment `CSI Driver for Node` will create new Host(s) in the Unity system depending on the number of nodes in kubernetes cluster.
    Verify Unity system for new Hosts and Initiators
    
2. **Creating a volume:**

    Create a file (`pvc.yaml`) with the following content.
    
    **Note**: Use default FC, iSCSI, NFS storage class or create custom storage classes to create volumes. NFS protocol supports ReadWriteOnce, ReadOnlyMany and ReadWriteMany access modes. FC/iSCSI protocol supports ReadWriteOnce access mode only.

    **Note**: Additional 1.5 GB is added to the required size of NFS based volume/pvc. This is due to unity array requirement, which consumes this 1.5 GB for storing metadata. This makes minimum PVC size for NFS protocol via driver as 1.5 GB, which is 3 GB when created directly on the array.

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: testvolclaim1
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
      storageClassName: unity
    ```

    Execute the following command to create volume
    ```
    kubectl create -f $PWD/pvc.yaml
    ```

    Result: After executing the above command, PVC will be created in the default namespace, and the user can see the pvc by executing `kubectl get pvc`. 
    
    **Note**: Verify unity system for the new volume

3. **Attach the volume to Host**

    To attach a volume to a host, create a new application(Pod) and use the PVC created above in the Pod. This scenario is explained using the Nginx application. Create `nginx.yaml` with the following content.

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pv-pod
    spec:
      containers:
        - name: task-pv-container
          image: nginx
          ports:
            - containerPort: 80
              name: "http-server"
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: task-pv-storage
      volumes:
        - name: task-pv-storage
          persistentVolumeClaim:
            claimName: testvolclaim1
    ```

    Execute the following command to mount the volume to kubernetes node
    ```
    kubectl create -f $PWD/nginx.yaml
    ```

    Result: After executing the above command, new nginx pod will be successfully created and started in the default namespace.

    **Note**: Verify unity system for volume to be attached to the Host where the nginx container is running

4. **Create Snapshot**
    The following procedure will create a snapshot of the volume in the container using VolumeSnapshot objects defined in snap.yaml. 
    The following are the contents of snap.yaml.
    
    *snap.yaml*

    ```
    apiVersion: snapshot.storage.k8s.io/v1beta1
    kind: VolumeSnapshot
    metadata:
      name: testvolclaim1-snap1
      namespace: default
    spec:
      volumeSnapshotClassName: unity-snapclass
      source:
        persistentVolumeClaimName: testvolclaim1
    ```
    
    Execute the following command to create snapshot
    ```
    kubectl create -f $PWD/snap.yaml
    ```
    
    The spec.source section contains the volume that will be snapped in the default namespace. For example, if the volume to be snapped is testvolclaim1, then the created snapshot is named testvolclaim1-snap1. Verify the unity system for new snapshot under the lun section.
    
    **Note**:
    
    * User can see the snapshots using `kubectl get volumesnapshot`
    * Notice that this VolumeSnapshot class has a reference to a snapshotClassName:unity-snapclass. The CSI Driver for Unity installation creates this class as its default snapshot class. 
    * You can see its definition using `kubectl get volumesnapshotclasses unity-snapclass -o yaml`.
          
5. **Delete Snapshot**

    Execute the following command to delete the snapshot
    
    ```
    kubectl get volumesnapshot
    kubectl delete volumesnapshot testvolclaim1-snap1
    ```
6.  **To Unattach the volume from Host**

    Delete the Nginx application to unattach the volume from host
    
    `kubectl delete -f nginx.yaml`

7. **To delete the volume**

    ```
    kubectl get pvc
    kubectl delete pvc testvolclaim1
    kubectl get pvc
    ```

8. **Volume Expansion**

    To expand a volume, execute the following command to edit the pvc:
    ```
    kubectl edit pvc pvc-name
    ```
    Then, edit the "storage" field in spec section with required new size:
    ```
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi #This field is updated from 5Gi to 10Gi which is required new size
    ```
    **Note**: Make sure the storage class used to create the pvc have allowVolumeExpansion field set to true. The new size cannot be less than the existing size of pvc.

9. **Create Volume Clone**

    Create a file (`clonepvc.yaml`) with the following content.

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: clone-pvc
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
      dataSource:
        kind: PersistentVolumeClaim
        name: source-pvc
      storageClassName: unity
    ```

    Execute the following command to create volume clone
    ```
    kubectl create -f $PWD/clonepvc.yaml
    ```
    **Note**: Size of clone pvc must be equal to size of source pvc.

    **Note**: For NFS protocol, user cannot expand cloned pvc.

    **Note**: For NFS protocol, deletion of source pvc is not permitted if cloned pvc exists.

10. **Create Volume From Snapshot**

    Create a file (`pvcfromsnap.yaml`) with the following content.

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: pvcfromsnap
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
      dataSource:
        kind: VolumeSnapshot
        name: source-snapshot
        apiGroup: snapshot.storage.k8s.io
      storageClassName: unity
    ```

    Execute the following command to create volume clone
    ```
    kubectl create -f $PWD/pvcfromsnap.yaml
    ```
    **Note**: Size of created pvc from snapshot must be equal to size of source snapshot.

    **Note**: For NFS protocol, pvc created from snapshot can not be expanded.

    **Note**: For NFS protocol, deletion of source pvc is not permitted if created pvc from snapshot exists.
