# Managing Objects with Labels, Annotations and Namespaces

## Namespaces

- Gives the ability to subdivide a cluster and its resources, providing resource isolation / organization
    - Conceptually, subdivide the cluster in "virtual clusters"
    - Objects are deployed on a Namespace
- Has nothing to do with Linux namespace concept

### Benefits

- Security boundary for Role-base Access Controls (RBACs)
    - Limits who can access what inside of the cluster
- Naming boundary
    - Allows two resources in separate namespaces to have the same name
    - A specific resource can be in only one namespace at a time

## Working with Namespaces

- Namespaces can be created (`kubectl create`), queried (`kubectl get` or `kubectl describe`) and deleted (`kubectl delete`)
- Objects in a Namespace can be operated (i.e., delete all Pods on a spacific Namespace)
- Some objects are Namespaced, some aren't
    - 👍 Resources: Pods, Controllers, Services
        - To list all Namespaced resources on the etcd, run `kubectl api-resources --namespaced=true`
    - 👎 Physical: PersistentVolumes, Nodes, CRDs, Namespaces
        - To list all Non-Namespaced resources on the etcd, run `kubectl api-resources --namespaced=false`
- Namespaces created by default:
    - `default` - for when resources are deployed without specifying a Namespace
    - `kube-public` - created automatically, readable by all users, stores objects to be shared between Namespaces across the whole cluster (ConfigMaps, for instance)
    - `kube-system`: system Pods (where the API Server, etcd, Controller Manager, kube-proxy etc resides)
- User-defined Namespaces are the custom ones created by the user. Where workloads or resources are deployed into
    - Can be created Imperatively, with `kubectl`, or
    - Declaratively in a Manifest in YAML/JSON

## Creating Namespaces and creating objects in Namespaces

```
#playground1
apiVersion: v1                      # Example of a YAML Manifest declaring a Namespace named 'playgroundinyaml'
kind: Namespace
metadata:
    name: playgroundinyaml
---
apiVersion: apps/v1                 # Example of how an object is declared within a Namespace
kind: Deployment
metadata:
    namespace: playgroundinyaml
```

To run it imperatively, run `kubectl create namespace playground1`. To create an object imperatively on an existing Namespace, run `kubectl run nginx --image=nginx -n playground1` (changing `nginx` for the proper data).

###### Return to [Summary](README.md)