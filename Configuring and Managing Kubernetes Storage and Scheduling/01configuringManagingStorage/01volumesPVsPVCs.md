# Configuring and Managing Storage in Kubernetes

## Persistent Storage
- How to persist data across a Pod's lifecycle?
    - With the following Storage API Objects:
        - **Volume**
        - **Persistent Volume** 
        - **Persistent Volume Claim**
        - **Storage Class**
    - On a high level, in the Pod spec, a **Volume** (NFS, Azure Disk, AWS EBS...) is defined
    - An independent API Object called **Persistent Volume Claim** allows to request storage from the cluster
    - The cluster must map this PVC back to an actual **Persistent Volume**, which is the actual storage in the cluster (and another independent API Object)

### Volumes
- Persistent storage deployed as part of the Pod spec, including technical implementation details for the storage
    - For example, if a NFS-type volume is declared, its server name or IP address must be included, along with exports accessed in the Pod's spec itself
    - Volumes tightly coupled with Pods exposes two real challenges:
        - Sharing code (portability) is limited (if this volume definition needs to be moved to another environment, the Pod's spec would need to be reworked)
        - Volumes have the same lifecycle as Pods, so once a Pod is destroyed, the definition of its volume is also destroyed

### Persistent Volumes
- Administrator-defined storage in the cluster (administrator is responsible for creating and deleting storage when needed), including technical implementation details for the storage
- Represents the actual storage itself for each individual storage element to be provisioned
- Has a completely independent lifecycle from the Pod, so it remains live as Pods come and go
- PV objects are created in the API Server, but actions (like mapping the storage to a Node or exposing the PV as a regular Linux mounting device inside the container's filesystem) are handled by the kubelet in each Node
    - PVs are mapped to the Node and then exposed into a Pod
- Parameters for definition:
    - Type (NFS, Fibre Channel, azureDisk...)
    - Capacity (amount of storage)
    - Access modes
    - `persistentVolumeReclaimPolicy`
    - Labels

```
apiVersion: v1
kind: PersistentVolume                  # where the PV is defined
metadata:
  name: pv-nfs-data
spec:
  capacity:
    storage: 10Gi                       # where storage capacity is defined
  accessModes:                          # where access modes are defined
    - ReadWriteMany                     # allows other types such as RWO and ROX
  nfs:                                  # PV type definition and specifications
    server: 172.16.94.5                 # IP of a storage server on the network
    path: "/export/volumes/pod"         # export on the NFS server to mount locally
```

#### Types
| Network | Block | Cloud |
| --- | --- | --- |
| NFS | Fibre Channel | AWS EBS |
| azureFile | iSCSI | azureDisk |
| --- | --- | gcePersistentDisk |

#### Access Modes
- **ReadWriteOnce (RWO)**
    - One Node can mount a volume for read/write access
- **ReadWriteMany (RWX)**
    - More than one Node can mount a volume for read/write access
- **ReadOnlyMany (ROX)**
    - More than one Node can mount a volume for read (only) access

### Persistent Volume Claims
- A request for storage by a user, allowing to ask the cluster for some amount of storage
- Requires specification of:
    - Size of PV to be used, and 
    - access mode to the PV
- Optionally, storage can be requested from a particular grouping in the cluster through a Storage Class (high-speed vs slower storage, for instance)
- Enable portability of the configurations (a Volume of the PVC type is defined in Pod's spec)
- When a Pod is defined to use a PVC, the cluster maps the PVC to a PV
- Parameters for definition (vital for the Binding process):
    - Access modes (same convention as for PVs)
    - Resources (requests for resources from the cluster - amount of storage to be allocated from the PV's capacity)
    - Storage Class name
    - Selector (performs a label query over the available PVs to match to the right one)

```
apiVersion: v1
kind: PersistentVolumeClaim             # where the PVC is defined
metadata:
  name: pvc-nfs-data
spec:
  accessModes:                          # where access modes are defined
    - ReadWriteMany                     # allows other types such as RWO and ROX
  resources:                            # resources to be requested
    requests:
      storage: 10Gi
```

#### Using PVs in Pods (inside of a Pod's spec)
```
...
spec:
  volumes:                                              # declaration of how many volumes are required
    - name: webcontent                                  
      persistentVolumeClaim:                            # volume type
        claimName: pvc-nfs-data
  containers:
  - name: nginx
    ...
    volumeMounts:                                       # how the volume should be mounted within the container
    - name: webcontent                                  # matching to the aforementioned volume
      mountPath: "/usr/share/nginx/html/web-app"        # mount location on the container's filesystem
```

### Static Provisioning
- Where the cluster administrator manually creates a PV for users and Pods
- Workflow:
    1. A `PersistentVolume` of the necessary type and specification is created, including implementation details for access to the storage
    2. A `PersistentVolumeClaim` is created, along with definition of size, access mode (and Storage Class, if so)
    3. A `Volume` of the `PersistentVolumeClaim` type is defined inside of the Pod's spec, which points to the individual PVC just defined

### Storage Lifecycle
| Binding -> | Using -> | Reclaim |
| --- | --- | --- |
| PVC created | Pod's Lifetime[^2] | PVC deleted |
| Control Loop finds PVC | --- | PV can be reclaimed[^3] |
| Control Loop matches PVC to PV[^1] | --- | Option to delete or retain |


[^1]: The Control Loop attempts to find a matching PV to the PVC either statically or dinamically, based on the parameters, using size, access mode, and (possible) Storage Class. If the Control Loop is unable to find a PV, the Pod remains pending until a PV is matched. Once bound, PVC and PV establish a one-to-one mapping for the lifecycle of that PVC.
[^2]: Part of the lifecycle where a bound PVC is used as Volume by a Pod during its lifetime
[^3]: Once a PVC is deleted, the PV can be reclaimed based on its reclaim policy's options: `Delete` (the default option) states that a PV deletion is authorized, while `Retain` saves PV state, allowing deletion to occur administratively.

###### Return to [Summary](README.md)