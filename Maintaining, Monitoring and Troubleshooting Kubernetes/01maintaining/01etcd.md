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
- `exec` into the running etcd Pod (if running on a Pod on the Control Plane 🧠 node)
- Run it inside of an etcd container
    - Good for running backups from inside of containers (on a Kubernetes CronJob)

#### Syntax
1. Run the `ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /var/lib/dat-backup.db` command; where
    - `ETCDCTL_API=3` is the environment variable that declares the API version used for this command (3),
    - `--endpoints` is the URL endpoint to the etcd instance running as static Pod (etcd usually keeps port 2379)
    - `--cacert` is the location of the CA certificate bundle used to authenticate to etcd
    - `--cert` identifies the client connecting to etcd
    - `--key` is the key file for the cert
    - `snapshot save` creates a backup/snapshot of etcd on `/var/lib/dat-backup.db`
2. Wait for the backup to be completed. To check if the backup file is valid (or its status), run `ETCDCTL_API=3 etcdctl --write-out=table snapshot status /var/lib/dat-backup.db`

### Restoring etcd


###### Return to [Summary](README.md)