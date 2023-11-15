# Configuring and Managing Storage in Kubernetes

## Storage Class
- Enables to define tiers/classes of storage and its attributes
- Enables **Dynamic Provisioning** (that is, dynamically allocating PVs from it)
    - PVC makes a claim from the Storage Class and then the storage provisioner creates the PV
- Requires defining infrastructure-specific parameters
- A Reclaim Policy must also be defined

```
apiVersion: storage.k8s.io/v1
kind: StorageClass                          # where the Storage Class is defined
metadata:
  name: managed-premium
parameters:                                 # are specific to the underlying storage provisioner
  kind: Managed
  storageaccounttype: Premium_LRS
provisioner: kubernetes.io/azure-disk       # the underlying storage provisioner
spec:
  persistentVolumeReclaimPolicy: Retain
```

### Dynamic Provisioning workflow
1. A `StorageClass` object must be created, with a definition on storage type and reclaim policy
2. Then, a `PersistentVolumeClaim` object should be created and set to point to the `StorageClass`
3. Inside of the Pod's spec, a `Volume` of the `PersistentVolumeClaim` type must be defined, pointing to the PVC object
4. When the Pod starts up, the PV is dinamically allocated

```
apiVersion: v1
kind: PersistentVolumeClaim                 # will dynamically provision storage from the StorageClass
metadata:
  name: pvc-azure-managed
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium         # must match to StorageClass' metadata
  resources:
    requests:
      storage: 10Gi
```

###### Return to [Summary](README.md)