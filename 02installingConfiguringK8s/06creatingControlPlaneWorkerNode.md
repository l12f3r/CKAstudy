# Installing and Configuring Kubernetes

## Creating a Control Plane 🧠 node

1. Download a YAML manifest: `wget https://docs.projectcalico.org/manifests/calico.yaml`
    - This is where the IP address range for the [Pod network](05podNetworkingFundamentals.md) is specified, under `CALICO_IPV4POOL_CIDR`
2. Create cluster configuration file using [kubeadm](01installationConsiderationsMethodsReqs.md#installation-methods): `kubeadm config print init-defaults | tee ClusterConfiguration.yaml`
    - This file will contain the configuration defaults for a cluster
        - Four changes must be performed on this file:
            - Control Plane node's IP endpoint must be entered on `localAPIEndpoint.advertiseAddress`;
            - `nodeRegistration.criSocket`: `\/run\/containerd\/containerd\/.sock`
            - Set the cgroup driver for the kubelet to systemd (running the code below, which does not exist on the file)
            ```cat <<EOF | cat >> ClusterConfiguration.yaml
            ---
            apiVersion: kubelet.config.k8s.io/v1beta1
            kind: KubeletConfiguration
            cgroupDriver: systemd
            EOF
            ```
            - Define the `kubernetesVersion` to the proper value
3. `sudo kubeadm init --config=CLusterConfiguration.yaml --cri-socket /run/containerd/containerd.sock` to start kubeadm passing the configuration and CRI socket files as parameters
    - CRI must be defined to avoid defaulting to Docker

## Run administratively with a non-privileged user

Once `kubeadm init` is completed, all Control Plane 🧠 [pods](03APIObjectsPods.MD) will be up and running. Some commands and parameters necessary to join additional nodes to the cluster will be printed, as well as instructions to run as admin with the user logged:

1. Create a directory for the kubeconfig file on ~: `mkdir -p $HOME/.kube`
2. Copy the default admin config file to the newly created directory, under a new name: `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
3. Change ownership of the config file to access with current user: `sudo chown $(id -u) :$(id -g) $HOME/.kube/config`

## Deploy a Pod network

`kubectl apply -f calico.yaml`

## Adding a Worker Node 👩‍🏭 to a cluster

1. **Disable swap and edit fstab**: `swapoff -a` and `vi /etc/fstab` (to ensure that `swap.img` is commented / not present)
2. **Run containerd prerequisites**: `sudo modprobe overlay` and `sudo modprobe br_netfilter` must be executed, then configured to be loaded on boot by running:
    ```
    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF
    ```
3. **Setup required sysctl parameters**:
    ```
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF
    ```
    - To apply such parameters without rebooting, run `sudo sysctl --system`
4. **Install containerd**: first, `sudo apt-get update`; then, `sudo apt-get install -y containerd`
5. **Configure containerd**: 
    - create a directory for configuration using `sudo mkdir -p /etc/containerd`, then use containerd to generate a default config file with `sudo containerd config default | sudo tee /etc/containerd/config.toml`;
    - edit the newly create config.toml file to set containerd's cgroup driver to systemd:
        - look for the line ending in `containerd.runtimes.runc` and add the following two lines, respecting indentation:
        ```
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
        ```
    - `sudo systemctl restart containerd`
6. **Install kubernetes packages**:
    - add Google's app repository GPG key: `curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`
    - add Kubernetes repo: 
    ```
    sudo bash -c 'cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF'
    ```
    - `sudo apt-get update`, then `sudo apt-get install -y kubelet kubeadm kubectl`
7. **Join node to cluster**: run `sudo kubeadm join 0.0.0.0:0 --token blkx8 sha256:hash`, changing those zeroes for the IP address and port of the [API server](02kubernetesAPI.MD), `blkx8` for the bootstrap token and `sha256:hash` for the CA cert hash
    - Token and cert hash are printed after `kubeadm init` is completed on the Control Plane 🧠

The following processes occur during join:
- **Node downloads cluster information**: some metadata as well
- **Generate and submit a certificate signing request (CSR)**: for the kubelet on the recently joined node to authenticate to the API server
- **CA automatically signs the CSR**: certificate is then dowloaded and stored on `/var/lib/kubelet/pki` of the node to be provisioned
- **Generate and configure `kubelet.conf`** to `/etc/kubernetes` on the node