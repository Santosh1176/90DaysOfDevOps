
# Creating a two-node Kubernetes Cluster with ContainerD as CRI and Cilium CNI addon

There are many ways to create a self-managed Kubernetes Cluster like [Kind], [Minikube], etc. Apart from these, there are many ways we can create managed Kubernetes clusters on cloud providers of our choice. The self-managed clusters created by the above tools are good for testing our workloads and integrations. given the complexity, though the kubeadm is not the most popular choice for creating a production-grade on-premise cluster. However, creating a cluster using Kubeadm can help in understanding the various components and configurations.

Kubeadm is a tool that performs the necessary actions to get a minimum viable cluster up and running within minutes. It also provides us flexibility in choosing necessary addons for CNI and CRI plugins among others. 

In this post I will provide an hands-on demo of installing a two-node Kubernetes Cluster built using the Kubeadm tool with ContainerD as a Container Runtime and Cilium as a CNI plugin.

Let's start with getting ready with our two nodes, which shall be virtual machines provisioned using VirtualBox and Vagrant. We will configure the VMs with SSH keys to enable communication between both VMs. Once we are ready with the VMs provisioned. We will start with configuring the nodes with the prerequisites like updating the packages, turning off the `swap memory`, etc.

We'll start with updating and upgrading the apt packages on both Nodes:
```bash
$ apt-get update && apt-get upgrade -y
``` 
> If required escalate the privileges by using the sudo privileges

Install the Kubernetes Packages using the apt repository:

`sudo apt-get install -y apt-transport-https ca-certificates curl`

Download the Google Cloud public signing key:

`curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg`

If you get an error like:
```bash
curl: (23) Failed writing body (0 != 1210)
```

This indicates the `/etc/apt/keyrings` directory does not exist. we need to create this specific directory to download the Google Cloud public signing keys.

Add the Kubernetes apt repository:

`echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list`

Update the `apt` packages and install `kubeadm`, `kubectl`, and `kubelet` packages from the apt package:
```bash
sudo apt-get install -y kubelet=1.26.0-00 kubeadm=1.26.0-00 kubectl=1.26.0-00
```

In the above snippet, we will be installing a specific version opf the tools i.e. `v1.26.0`.

We need to place a hold on the above-installed packages for any accidental upgrades:
`sudo apt-mark hold kubelet kubeadm kubectl`

> We need to repeat all the above processes on the kubenode01 as wll. We can choose if we need `kubectl` tool available on the worker node.

Once, the above steps are performed on both the ControlPlane and Worker Node. We need to perform a very important configuration on both nodes. **Disabling swap memory, enabling a couple of Kernel Modules, and Changing the Settings in sysctl**.

First, we need to [disble the swap](https://github.com/kubernetes/kubernetes/issues/53533) in order for the kubelet to work properly, we'll do it by first checking for the `/etc/fstab` and look for a line:

`/swap.img      none    swap    sw      0       0`

If this line is available in the `fstab` file, we can disable this setting by commenting it out. Instead of rebooting our nodes, we can apply the following command to disable the swap `sudo swapoff -a`.

Once the swap is disabled, we need to enable two kernel modules, `overlay` and `br_netfilter`:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

`sudo modprobe overlay`

`sudo modprobe br_netfilter`

```bash
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

Now, we add `sysctl` configurations:

```bash
sudo vi /etc/sysctl.d/kubernetes.conf
```
and add the following lines:



Once the file is saved, we must reload the `sysctl`:

`sudo sysctl --system`

In the next session we shall move ahead and install the ContainerD as CRI and Cilium as CNI addon. 