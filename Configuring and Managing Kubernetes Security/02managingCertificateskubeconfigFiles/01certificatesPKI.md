# Managing Certificates and kubeconfig Files

## Certificates and PKI in Kubernetes
- K8s uses certificates to provide **TLS encryption**
    - Also used for authenticating users and system components (like the Scheduler and Controller Manager 🧠, and kubelets and kube-proxy 👩‍🏭)
- Default kubeadm-based clusters create a **self-signed root Certificate Authority**
    - Used to create certificates for the aforementioned system components
        - Such certificates are used for authentication and encryption
    - Keys and certificates for the CA are stored in `/etc/kubernetes/pki`: `ca.crt` and `ca.key`
        - `ca.key` is the private key associated with the CA - must be safely kept 🔐
    - External/trusted CAs can also be created
- kubeadm-based clusters also get a user named `kubernetes-admin` by default, occupying the cluster admin role
    - Authenticates to the cluster using a certificate defined on `admin.conf` kubeconfig file

### `ca.crt`
- CA certificate used by clients to trust certificates signed by the CA
- Can be found on Worker nodes 👩‍🏭
    - During the cluster build, the kubeadm copies it from Control Plane's 🧠 `/etc/kubernetes/pki` to then-joining Worker nodes 👩‍🏭
- Also appear as part of kubeconfig files for encryption between clients and the API Server
    - Included to trust the CA certificate presented by the API Server
- Part of the secret automatically created upon creating a [`ServiceAccount`](../01k8sSecurityFundamentals/02usersServiceAccounts.md)
    - Allows Pod-based applications to trust the certificate presented by the API Server

### How certificates are used in k8s
- When users want to consume the Kubernetes API (using kubectl, for instance), the API Server has a server certificate providing an encrypted HTTPS endpoint for this consumption
- On the client side, the [CA certificate](#cacrt) must be defined as part of the cluster definition on the kubeconfig file so that the client can trust the certificate presented by the API Server
    - Client certificate and private key are also defined on the kubeconfig file
- In parallel, Service Accounts may use tokens for authenticating Pod-based applications to the cluster, but the CA certificate is included in its secret, so those applications can also trust the certificates signed by the self-signed CA
- There is also a collection of certificates used for authentication upon reading and writing operations to etcd, Controller Manager and Scheduler 🧠
- On the Worker node 👩‍🏭, each kubelet has a kubeconfig file (`kubeconfig.conf`), used to authenticate the kubelet to the API Server, while kubeproxy's kubeconfig file is stored as ConfigMap in the `kube-system` namespace
    - This ConfigMap is exposed into the `kube-proxy` Pod as a volume mounted

###### Return to [Summary](README.md)