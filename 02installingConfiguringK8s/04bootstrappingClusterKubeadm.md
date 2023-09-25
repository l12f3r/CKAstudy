# Installing and Configuring Kubernetes

## Bootstrapping a Cluster with kubeadm

`kubeadm init` begins the process of creating a cluster. By default, it triggers a series of phases:
1. **Pre-flight checks**: series of checks to ensure the cluster is properly created (permissions, system requirements, container runtime compatibility etc)
2. **Certificate Authority creation**: for authentication and encryption
3. **Generates `kubeconfig` files**: to locate and authenticate against the [API server](../01exploringKubernetesArchitecture/02kubernetesAPI.MD)
4. **Generates static [Pod](../01exploringKubernetesArchitecture/03APIObjectsPods.MD) manifests**
5. **Waits for the Control Plane 🧠 pods to start**: so that the API server, [etcd](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components) and other components are started up as Pods
6. **Taints the Control Plane 🧠 node**: so that user Pods are never scheduled on the Control Plane 🧠 node, only system Pods
7. **Generates a bootstrap token**: used for joining additional nodes to the cluster
8. **Starts add-on components**: [DNS and kube-proxy](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#add-on-pods-%F0%9D%9F%AD-provide-special-services-to-the-cluster) pods

### Certificate Authority

- A self-signed Certificate Authority (CA) is created by default
  - If required by the organization, it can be integrated with an external PKI
- This CA is used to secure cluster communications
  - Generates server certificates used by the [API server](../01exploringKubernetesArchitecture/02kubernetesAPI.MD) for HTTPS encryption
- Also used to generate certificates for user/cluster component authentication
- CA files live in `/etc/kubernetes/pki` and are distributed to each node on cluster upon joining additional nodes

### `kubeconfig` files

- Configuration files that defines how to connect to the cluster's [API server](../01exploringKubernetesArchitecture/02kubernetesAPI.MD), living on `/etc/kubernetes`
- Contents:
  - Certificates used for client authentication
  - Cluster's API server network location (its IP addresss or DNS name usually)
- List of files:
  - `admin.conf`: cluster's administrator account (kubernetes-admin)
  - `kubelet.conf`, `controller-manager.conf` and `scheduler.conf`: used to help locainge the API server and present the correct client certificate for authentication for each of these components

### Static Pod manifests

- Lives in `/etc/kubernetes/manifests`
- Describes Pod configurations
- Generated for each of the Control Plane 🧠 components ([etcd](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components), [API server](../01exploringKubernetesArchitecture/02kubernetesAPI.MD), [Controller Manager](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components), [Scheduler](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#control-plane-node-components))
- Written to the file system and monitored by the [Kubelet](../01exploringKubernetesArchitecture/07k8sClusterComponents.md#worker-node-components)

###### Return to [Summary](https://github.com/l12f3r/CKAstudy/tree/main/02installingConfiguringK8s#readme)