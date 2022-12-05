# Installing and Configuring Kubernetes

## Building your cluster

1. Install and configure packages 📦 and container runtime
2. Create the cluster
    - Firstly, bootstrap the control plane node 🧠 using [kubeadm](01installationConsiderationsMethodsReqs.md#installation-methods) and then the other components ([API server](/01exploringKubernetesArchitecture/02kubernetesAPI.MD), [etcd](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components), [Controller Manager](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components))
3. Configure Pod networking
    - This example demonstrates the usage of an overlay network
4. Join nodes to cluster
    - Worker nodes 👩‍🏭 for application workloads

## Required packages

| containerd | container runtime |
| [Kubelet](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components) | drives work on individual nodes on the cluster |
| [kubeadm](01installationConsiderationsMethodsReqs.md#installation-methods) | bootstraps cluster, joins additional nodes to cluster |
| [kubectl](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#kubectl) | CLI used to communicate with cluster |
