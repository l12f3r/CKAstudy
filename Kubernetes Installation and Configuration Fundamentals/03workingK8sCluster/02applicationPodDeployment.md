# Working with your Kubernetes Cluster

## Application and Pod deployment in Kubernetes

### Configurations

- **Imperative**: command line, one object at the time 
    - `kubectl create deployment nginx --image=nginx`
- **Declarative**: desired state described in manifest file (YAML or JSON) 
    - `kubectl apply -f deployment.yaml`

## Example of a basic Deployment manifest file

```
# This is an excerpt of an example deployment.yaml file.
# To execute it, run 'kubectl apply -f deployment.yaml'

apiVersion: apps/v1             #Version of K8s API applied in this manifest
kind: Deployment                #Kind of object to be defined, described in kubectl api-resources
metadata:                       #Description of work being done
  name: hello-world             #Declaration of Deployment's name
spec:                           #Implementation details of the object
  replicas: 2                   #Number of replicas of Pods required for this Deployment
  selector:                     #Reference to know which Pods are member of this Deployment
    matchLabels:
      app: hello-world
  template:                     #Definition of Pods created by this Deployment
    metadata:
      labels:                   #How Deployment tracks which pods are members
        app: hello-world        #Must match spec.selector.matchLabels
    spec:
      containers:               #Definition of containers started by Pods in this Deployment
      - name: hello-app
        image: gcr.io/google-samples/hello-app:1.0
```

- Use `kubectl explain` to quickly lookup the fields for an object to help fill out the implementation details
- Use `--dry-run=client -o yaml` to quickly generate a representation of a manifest imperatively 
    - `kubectl create deployment hello-world --image=gcr.io./google-samples/hello-app:1.0 --dry-run=client -o yaml > deployment.yaml`

## Application deployment process

1. 🧠 On a cluster already set, `kubectl apply -f deployment.yaml` is executed. This manifest file describes objects to be created. This deployment creates a [ReplicaSet](../01exploringKubernetesArchitecture/04APIObjectsControllers.MD#types-of-controllers), which creates [Pods](../01exploringKubernetesArchitecture/03APIObjectsPods.MD) based on its `spec`. This request is sent to the [API server](/01exploringKubernetesArchitecture/02kubernetesAPI.MD);
2. 🧠 The API server parses the information defined on the manifest and stores such info on [etcd](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components);
3. 🧠 The [Controller Manager](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components) is listening for any new objects. It then starts up a controller for this deployment, which creates another ReplicaSet 
    - This one creates the required number of Pods to support the configuration and writes such info back in etcd;
4. 🧠 The [scheduler](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components) listens to etcd for unassigned Pods, scheduling it to the nodes. Such changes are updated in etcd;
5. 👩‍🏭 The Pod starting process begins with [kubelets](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components) getting workload from the API server - it listens to Pods scheduled for that node;
6. 👩‍🏭 The kubelet sends a message to the [container runtime](01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components) to pull down the appropriate images specified on the Pod spec, and then starts the Pod within the node;
    - If this Pod is a member of a Service, then the Service information is updated on [kube-proxy](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components)

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/03workingK8sCluster#readme)