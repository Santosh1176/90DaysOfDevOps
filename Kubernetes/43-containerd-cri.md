
# Installing ContainerD

With the [Deprication of Docker](https://kodekloud.com/blog/kubernetes-removed-docker-what-happens-now/) for Kubernetes since v1.25.0, [Containerd](https://www.docker.com/blog/what-is-containerd-runtime/) is one of the preffered choices of Ops team. Containerd is a Container-Runtime developed by Docker that manages the container lifecycle. In February 2019, Containerd became an official project within the Cloud Native Computing Foundation (CNCF).

While there are multiple ways to install Containerd, we shall be using the method of installing it using the `apt` package. The Containerd runtime needs to be installed on both the nodes. The first thing to do is configure the persistent loading of the necessary Containerd modules by using the following commands:

```bash
sudo tee /etc/modules-load.d/containerd.conf << EOF
overlay
br_netfilter
EOF
```
Reload the sysctl configurations with `sudo sysctl --system` command.

Install the necessary dependencies with the following command:

`sudo apt install curl gnupg2 software-properties-common apt-transport-https ca-certificates -y`

Now, we need to add the GPG keys with `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -` command.

Adding the repository with `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"` command.

Now, we are ready to install the Containerd package through the `apt` package manager using the following command:

```bash
sudo apt update
sudo apt install containerd.io -y
```
Once, we have successfully installed the Containerd. We need to load the Containerd configurations, for this, we might need to gain access to `sudo` privileges `sudo -i`:

```bash
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
```
Now, restart the Containerd systemd service and enable it:

```bash
systemctl restart containerd
systemctl enable containerd
```

By this, we have installed Containerd as a CRI installed on both of our nodes.