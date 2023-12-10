# Maintaining Kubernetes Clusters

## etcd 
- Key/value datastore used for persistently store cluster state data and objects
    - All API objects working on a cluster have their configuration and state stored in etcd as data
- Its contents must be protected from a disaster, and its availability must be ensured (**high availability** concept)
    - To do so, a solid **backup and restore** plan must be in place
- Represented on the cluster as a single Pod on a single Control Plane 🧠 node, with `/var/lib/etcd` as the default data directory
    - Such directory is backed by a `hostPath`-based volume mounted into the Pod from the base operating system at the same `/var/lib/etcd` location

### Backing up etcd
- `etcdctl` is the CLI for interacting with etcd
- Provides backup and restore capabilities for etcd datastores (primary backup method: using the **snapshot** capability)
    - A backup file with the entire state of the data stored in etcd is created
- Such backup files must be secured and/or encrypted to protect the sensitive information stored (remember: [Secrets are not encrypted, only hashed](/Configuring%20and%20Managing%20Kubernetes%20Storage%20and%20Scheduling/02configurationAsDataEnvironmentVariablesSecretsConfigMaps/02secrets.md))
- Backups must be copied offsite from the primary location ASAP (on another cloud region or availability zone, for example) to ensure high availability
- Backup processes must be automatized on either Linux or [Kubernetes](/Managing%20Kubernetes%20Controllers%20and%20Deployments/03daemonSetsJobs/02jobsCronJobs.md) cronjob scripts

#### Getting etcdctl
- Download the binary directly from GitHub and launch it directly on local system, or on the base operation system of a  Control Plane 🧠 node
    - Good for running backups with a shell script (from a Linux cronjob, for instance)
    - `RELEASE` must match to the release version (to find the release version, run `--version` inside of the etcd Pod: `kubectl exec -it $ETCD_POD -n kube-system -- /bin/sh -c 'ETCDCTL_API=3 /usr/local/bin/etcd --version' | head`)
```
export RELEASE="3.4.3"
wget https://github.com/etcd-io/etcd/releases/download/v${RELEASE}/etcd-v${RELEASE}-linux-amd64.tar.gz
tar -zxvf 
cd etcd-v${RELEASE}-linux-amd64.tar.gz
sudo cp etcdctl /usr/local/bin
```
- `exec` into the running etcd Pod (if running on a Pod on the Control Plane 🧠 node)
- Run it inside of an etcd container
    - Good for running backups from inside of containers (on a Kubernetes CronJob)

#### Syntax
1. Run the `ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /var/lib/dat-backup.db` command; where
    - `ETCDCTL_API=3` is the environment variable that declares the API version used for this command (3),
    - `--endpoints` is the URL endpoint to the etcd instance running as static Pod (etcd usually keeps it `localhost`, port 2379)
    - `--cacert` is the location of the [CA certificate](/Kubernetes%20Installation%20and%20Configuration%20Fundamentals/02installingConfiguringK8s/04bootstrappingClusterKubeadm.md#certificate-authority) bundle used to authenticate to etcd
    - `--cert` identifies the client connecting to etcd
    - `--key` is the key file for the cert
    - `snapshot save` creates a backup/snapshot of etcd on `/var/lib/dat-backup.db`
        - Locations for certificates and its keys can be found on the `/etc/kubernetes/manifests/etcd.yaml` file (hint: `| grep file`), or by describing the etcd Pod (`kubectl describe pod $ETCD_POD -n kube-system`)
2. Wait for the backup to be completed. To check if the backup file is valid (or its status), run `ETCDCTL_API=3 etcdctl --write-out=table snapshot status /var/lib/dat-backup.db`

### Restoring etcd
#### Single server Pod-based etcd
1. If restoring to an existing etcd server or Control Plane 🧠 node, the backup needs to be restored to other directory on the FS than the default `/var/lib/etcd`
2. If there is existing data on the default directory, it should be moved away to somewhere else and the etcd Pod must be stopped
3. After the etcd Pod is stopped, the restored data must be moved to `/var/lib/etcd`
    - An alternative would be not moving data to the default directory, but to any other location on the file system and then editing etcd's static Pod manifest to point to this other location
        - Edit the `spec.containers.command.--data-dir`, `spec.volumeMounts.mountPath` and `volumes.hostPath.path` parameters, pointing its values to the directory where backups are now settled
4. The kubelet starts the etcd Pod again, finding the restored data on the default directory and completing the restore

#### Syntax
1. Run the `ETCDCTL_API=3 etcdctl snapshot restore /var/lib/dat-backup.db` command; it restores the backed up database file into the current working directory, on a subfolder named `default.etcd`
2. If existent, the default `/var/lib/etcd` directory must be moved to another location: `mv /var/lib/etcd /var/lib/etcd.old`
3. Stop the static etcd Pod (it cannot be deleted to restart the container) by stopping the container at runtime level: 
    - `sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps` lists running processes within the container, from where the container ID can be obtained;
    - `sudo crictl --runtime-endpoint unix:///containerd/containerd.sock stop $CONTAINER_ID` to stop the etcd container running the Pod
4. Before the kubelet quickly restarts the container, the backed up content must be moved to the default directory: `mv ./default.etcd /var/lib/etcd`

###### Return to [Summary](README.md)