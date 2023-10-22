# Create k8s Cluster with CRI-o

## In this demo, I be using 1 Master and 2 Worker nodes

### 1. Ensure that the server is up to date
It is always a good practice to ensure the system packages are updated. Use this command to ensure our Ubuntu system has up to date packages:

```cmd

sudo apt update
sudo apt -y upgrade

```

### 2. Install kubelet, kubeadm and kubectl
Once the servers are updated, we can install the necessary tools for kubernetes installation. These are kubelet, kubeadm and kubectl.

These are not found in the default Ubuntu repositories, so let us set up kubernetes repositories for this:

```cmd
sudo apt -y install curl apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

```

Then install the required packages.

```cmd

sudo apt update
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```

Confirm the installation by checking the version of kubectl.

```cmd

kubectl version --client && kubeadm version

```

You will see somthing like this:

Client Version: v1.28.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
kubeadm version: &version.Info{Major:"1", Minor:"28", GitVersion:"v1.28.2", GitCommit:"89a4ea3e1e4ddd7f7572286090359983e0387b2f", GitTreeState:"clean", BuildDate:"2023-09-13T09:34:32Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}
