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

### 3. Disable Swap and Enable Kernel modules
Use this command to turn off swap

```cmd

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a

```

Next we need to enable kernel modules and configure sysctl.

To enable kernel modules

```cmd

sudo modprobe overlay
sudo modprobe br_netfilter

```

Add additional settings to sysctl

```cmd

sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

```

Reload sysctl

```cmd

sudo sysctl --system

```

### 4. Install Container runtime
A container runtime is the software that is responsible for running the containers. When installing kubernetes, you need to install a container runtime into each node in the cluster so that Pods can run there.

Supported container runtimes are:

[Docker Engine] (https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker) (Not Recommended - Dockershim has been removed from the Kubernetes project as of release 1.24)
[CRI-O] (https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)
[Containerd] (https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
[Mirantis Container Runtime] (https://kubernetes.io/docs/setup/production-environment/container-runtimes/#mcr)

**Note:**
Kubernetes releases before v1.24 included a direct integration with Docker Engine, using a component named dockershim. That special direct integration is no longer part of Kubernetes (this removal was announced as part of the v1.20 release).

You have to choose one runtime at a time. In this guide we will use CRI-O.

First add CRI-O repo. We are going to run it as root.

```cmd

sudo -i

```

Some parameters we need:

*OS=xUbuntu_22.04*
*VERSION=1.28*

|OS Version   | $OS          |
|:-----------:|:------------:|
|Ubuntu 22.04 |xUbuntu_22.04 |
|Ubuntu 21.10 |xUbuntu_21.10 |
|Ubuntu 21.04 |xUbuntu_21.04 |
|Ubuntu 20.10 |xUbuntu_20.10 |
|Ubuntu 20.04 |xUbuntu_20.04 |

To check Ubuntu version

```cmd

lsb_release -a

```

Since I am running Ubuntu 22.* on both nodes, I will set the value as below:

**OS=xUbuntu_22.04**

Next, set the VERSION of cri-o to download. This should match the version of Kubernetes you are planning to deploy. I will be using Kubernetes 1.28 and hence will set VERSION to 1.28.

**VERSION=1.28**

```cmd

echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -

```

We can now install CRI-O after ensuring that our repositories are up to date.

```cmd

sudo apt update
sudo apt install cri-o cri-o-runc

```

To confirm the installed version, use this command:

```cmd

apt-cache policy cri-o

```

You should see something as below:
cri-o:
  Installed: 1.28.1~0
  Candidate: 1.28.1~0
  Version table:
*** 1.28.1~0 500
        500 http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.28/xUbuntu_22.04  Packages
        100 /var/lib/dpkg/status
     1.27.1~0 500
        500 http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.27/xUbuntu_20.04  Packages
