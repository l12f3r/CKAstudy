# Installing and Configuring Kubernetes

## Getting Kubernetes
- Maintained on GitHub: https://github.com/kubernetes/kubernetes

## Building your cluster
1. Install and configure packages 📦 and container runtime
2. Create the cluster
    - Firstly, bootstrap the control plane node 🧠 using [kubeadm](01installationConsiderationsMethodsReqs.md#installation-methods) and then the other components ([API server](/01exploringKubernetesArchitecture/02kubernetesAPI.MD), [etcd](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components), [Controller Manager](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components))
3. Configure Pod networking
    - This example demonstrates the usage of an overlay network
4. Join nodes to cluster
    - Worker nodes 👩‍🏭 for application workloads

## Required packages
| Package  | Reason why required |
| ------------- | ------------- |
| containerd | container runtime |
| [Kubelet](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components) | drives work on individual nodes on the cluster |
| [kubeadm](01installationConsiderationsMethodsReqs.md#installation-methods) | bootstraps cluster, joins additional nodes to cluster |
| [kubectl](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#kubectl) | CLI used to communicate with cluster |


### Installing on Ubuntu VMs [^1]:
1. Install container runtime: `sudo apt-get install -y containerd`
2. Add GPG key for the apt repo where Kubernetes packages live: `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`
3. Add apt repo to local repo list: 
```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
4. Update package list: `apt-get update`
5. Install remaining required packages ([Kubelet](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components), [kubeadm](01installationConsiderationsMethodsReqs.md#installation-methods) and [kubectl](/01exploringKubernetesArchitecture/07k8sClusterComponents.md#kubectl)): `apt-get install kubelet kubeadm kubectl`

[^1]: Should be executed on all nodes (both control plane 🧠 and worker 👩‍🏭 ones)