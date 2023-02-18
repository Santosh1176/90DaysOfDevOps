# Configure Kubernetes Controlplane

Once, the CRI is installed successfully on both nodes, we are now ready to configure our Kubernetes Controlplane.
We need to perform the following actions on the ControlPlane node we have identified earlier. 

```bash
$ kubeadm init --kubeadm init --apiserver-advertise-address 192.168.56.11
```
> The kubeadm init command expects mainly two arguments among others. the --pod-network-cidr and --apiserver-advertise-address. The --pod-network-cidr enables inter-pod networking which we will install using Cilium. The --apiserver-advertise-address is the one we need to carefully assign the IP address of the Controlplane nodes API Server. We can get the IP address by using `ifconfig` or `ip a` command. 

This command will configure and bootstrap the controlplane node by installing all necessary components and provide an output with a `kubeadm join` command and other settings required.

for example:
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

kubeadm join 192.168.56.11:6443 --token 1gehfl.g3n31uj4cmvnzxug --discovery-token-ca-cert-hash sha256:17a0b8da44fe941c2c00808928a6bbce54d1e7b42d77c865b3e619192949856f
```

In case we miss this join command with the token, we can create a new token with the following command to be used on the Worker node:

```bash
vagrant@kubemaster:~$ kubeadm token create --print-join-command
kubeadm join 192.168.56.11:6443 --token 1gehfl.g3n31uj4cmvnzxug --discovery-token-ca-cert-hash sha256:17a0b8da44fe941c2c00808928a6bbce54d1e7b42d77c865b3e619192949856f 
```
We need to copy this `kubeadm join` command and apply this on our `kubenode01` worker node to join the cluster. Once we have applied this `join` command. We need to return to the Controlplane and configure the `kubeconfig` required for authentication to the API server.

We do this by creating a directory to place the cluster details, contexts, and credentials in the `mkdir -p $HOME/.kube` directory by coping the following:

```bash
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Once this is configured, our cluster should be almost up. We can check this by using `kubectl get nodes` command:

```bash
vagrant@kubemaster:~$ kubectl get nodes
NAME         STATUS     ROLES           AGE     VERSION
kubemaster   NotReady   control-plane   3m59s   v1.26.0
kubenode01   NotReady   <none>          8s      v1.26.0
```

We can see the Nodes are in **NotReady** state, this is because we have not yet implemented the [Pod Networking CNI plugin](https://github.com/containernetworking/cni#3rd-party-plugins) yet. For this experiment, we shall be using the [**Cilium**](https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/cilium-network-policy/) as our networking solution.

We first download the Cilium binaries by using `curl -LO https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz` command.
Then extract the downloaded file to your /usr/local/bin directory with the following command:
```bash
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
rm cilium-linux-amd64.tar.gz
```
After running the above commands, you can now install Cilium with the following command: `cilium install`. Once the cilium is installed, we can check the status of the Cilium by using the `cilium status` command to confirm that the cilium is correctly installed. Once the Cilium addon is installed, we should see the Pod networking enabled and our nodes in the **Ready** state:

```bash
✅ Cilium was successfully installed! Run 'cilium status' to view installation health
vagrant@kubemaster:~$ kubectl get nodes
NAME         STATUS     ROLES           AGE     VERSION
kubemaster   NotReady   control-plane   12m     v1.26.0
kubenode01   Ready      <none>          8m17s   v1.26.0
vagrant@kubemaster:~$ kubectl get nodes
NAME         STATUS   ROLES           AGE     VERSION
kubemaster   Ready    control-plane   12m     v1.26.0
kubenode01   Ready    <none>          8m23s   v1.26.0
```