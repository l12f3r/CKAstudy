# Managing Certificates and kubeconfig Files

## kubeconfig files and Certificate-based Authentication
- A kubeconfig file defines the **network location** of the cluster, as well as how a **client authenticates** to the API Server
    - When a client of the API Server makes a request, such information is read from the kubeconfig file
- Works for both **users or system components**
- Contains three certificate-related elements:
    - [`ca.crt`](01certificatesPKI.md#cacrt) CA certificate presented by the API Server
        - Server certificate on the API Server is signed by kubeadm-based clusters' CA by default
    - Client certificate
    - Client private key
        - Both need to be defined when defining users that authenticate to the API Server

###### Return to [Summary](README.md)