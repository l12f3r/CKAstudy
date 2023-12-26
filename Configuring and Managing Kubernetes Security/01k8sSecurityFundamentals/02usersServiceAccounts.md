# Kubernetes Security Fundamentals

## Users 👤
- People who needs to access the cluster
- Login names and IDs are managed by external systems (certificates, tokens, password files) independent of Kubernetes
    - It means that there is no "user" API object
- The selected [authentication plugin](01authenticatingAPIServer.md#authentication-plugins) defines and implements authentication
    - Certificates: a request is authenticated when a valid, trusted certificate is presented as part of the HTTP request
- Authentication is pluggable; one or more authentication methods are possible
- Usernames are used for access control, authorization decisions and logging
- Users can be aggregated into **groups**
    - Groups are part of the certificate passed to the API Server during authentication: the Organization field on the certificate is used to define a listing of groups the user is a member of

## Service Accounts ⚙️
- Pod-based applications that need to authenticate to the API Server for executing actions
    - Retrieving information about cluster state
- Allows to **apply permissions** for authorizing actions to particular API objects/resources
- It is a namespaced API object managed in Kubernetes
    - Consists of Service Account name and credential, amongst other information
        - Credential: token
- Upon creation, each namespace on a cluster has a default `ServiceAccount`
    - All Pods running on a cluster must have a `ServiceAccount` defined on Pod spec; if not defined, the namespace's default `ServiceAccount` will be applied
    - `ServiceAccounts` can be created and used instead of the default one, granting more granular security

### Service Accounts Credentials
- Each Service Account object is tied to a credential that is stored as `Secret`, on the cluster
    - Contents of the `Secret`: CA certificate, authentication token and namespace of the `ServiceAccount`
    - By default, are mounted into Pods as files at `/var/run/secrets/kubernetes.io/serviceaccount`
- This credentials can be used to interact with the API Server
    - CA certificate enables the Pod-based application to trust the certificate
    - Token is used in token-based authentication to the API Server
    - Namespace defines the scope of resources to access
- In addition to authenticating to the API Server, it is also possible to store an **image pull secret** on a Service Account `Secret`
    - Credential information used to access a container registry that requires authentication

### How to create
- Imperatively: `kubectl create sa mysvcaccount1`
- Declaratively:
```
apiVersion: 1
kind: ServiceAccount                        # where the ServiceAccount is declared
metadata:
  name: mysvcaccount1
  namespace: default                        # other namespaces can be defined
```
- Defining on a Pod's spec:
```
...
spec:
  serviceAccount: mysvcaccount1             # where the created ServiceAccount is called; calls default if not defined
  containers:
  - image: nginx
    name: nginx
```

###### Return to [Summary](README.md)