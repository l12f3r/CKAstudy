# Working with your Kubernetes Cluster

## kubectl introduction, usage and closer look

- Primary CLI tool for controlling workloads on a Kubernetes cluster
    - To perform **operations** (create, read, update or delete by interacting with the [API server](../01exploringKubernetesArchitecture/02kubernetesAPI.MD)) against
    - **resources** (objects like [Pods](../01exploringKubernetesArchitecture/03APIObjectsPods.MD), [Deployments](../01exploringKubernetesArchitecture/04APIObjectsControllers.MD#types-of-controllers), [Services](../01exploringKubernetesArchitecture/05APIObjectsServices.md) etc)
- Generates output on different formats (JSON, YAML)

## Main operations using kubectl
- `kubectl apply` or `kubectl create`: sends API request to create a Deployment resource
- `kubectl run`: starts a single Pod from an image (when Pod is not managed by a [controller](../01exploringKubernetesArchitecture/04APIObjectsControllers.MD))
- `kubectl config get-contexts`: information about the current cluster context (ensure authentication on the correct cluster)
    - `kubectl config use-context context-name@kubernetes` to select the proper context 
- `kubectl explain`: provides documentation on resources, lists required fields (`apiVersion`, `kind`, `spec` etc) for declaring resource
- `kubectl delete`: deletes resources
- `kubectl get`: lists resources, provides basic information about specified resource type
- `kubectl describe`: provides detailed resource information such as Labels, Taints, Conditions, Addresses, System Info, Non-terminated Pods and Events
- `kubectl exec`: execute a command on a container
- `kubectl logs`: obtain logs from a container's stdout
- `kubectl cluster-info`: lists/inspects cluster on current context (where the Control Plane 🧠 is running and KubeDNS information)
- `kubectl api-resources`: asks for known resources and lists its aliases (`SHORTNAMES`)
- `kubectl -h`: help page - can be combined with operations (`kubectl get -h` for example)
- `kubectl expose`: exposes the object as a Service, creating a Service for it
    - `--port=80`: Service's port within the internal cluster. Where cluster resources must point to
    - `--target-port=8080`: Pod's service port, defined when Pods are started

## Specifying the output format
- `-o wide`:  output additional info on deployed resources
- `yaml`: outputs content as YAML formatted API object
- `json`: outputs content as JSON formatted API object
- `dry-run`: prints the object but no further action is taken; for test reasons

## How a kubectl command should be
`kubectl [command] [type] [name] [flags]` is the example pattern herein used. Some examples of valid commands:
- `kubectl get pods podExample --output=yaml` would output information from the `podExample` Pod as YAML. If only `kubectl get pods` was entered, information on all Pods within the default namespace would be provided;
- `kubectl create deployment nginx --image=nginx` would create a deployment using the `nginx` image.

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/03workingK8sCluster#readme)