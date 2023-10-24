# Create k8s Cluster with CRI-O 

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

[Docker Engine](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

[CRI-O](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)

[Containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

[Mirantis Container Runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#mcr)

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

### 5. Start and enable CRI-O
Let us reload systemd units and start the service while enabling it on boot.

```cmd

sudo systemctl daemon-reload
sudo systemctl enable crio --now

```

Confirm the app status:

```cmd

sudo systemctl status crio

```

You should see somethings like below:
● crio.service - Container Runtime Interface for OCI (CRI-O)
     Loaded: loaded (/lib/systemd/system/crio.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-10-22 06:08:25 UTC; 13s ago
       Docs: https://github.com/cri-o/cri-o
   Main PID: 4509 (crio)
      Tasks: 8
     Memory: 11.2M
        CPU: 142ms
     CGroup: /system.slice/crio.service
             └─4509 /usr/bin/crio

Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.032774592Z" level=info msg="Attempting to res>
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.032926421Z" level=info msg="Restore irqbalanc>
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.038904931Z" level=warning msg="Error encounte>
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.039256110Z" level=info msg="Registered SIGHUP>
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.039306000Z" level=info msg="Starting seccomp >
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.039377150Z" level=info msg="Create NRI interf>
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.039505020Z" level=info msg="NRI interface is >
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.039852890Z" level=error msg="Writing clean sh>
Oct 22 06:08:25 k8s-master crio[4509]: time="2023-10-22 06:08:25.039883630Z" level=error msg="Failed to sync p>
Oct 22 06:08:25 k8s-master systemd[1]: Started Container Runtime Interface for OCI (CRI-O).

### 6. Initialize master node
Now that we have taken care of the prerequisites, let us initialize the master node.

Login to the server to be used as master and make sure that the br_netfilter module is loaded:

```cmd

lsmod | grep br_netfilter

```

br_netfilter           28672  0
bridge                249856  1 br_netfilter

Let us also enable the kubelet service to start on boot.

```cmd

sudo systemctl enable kubelet

```

Pull the required images using this command:

```cmd

sudo kubeadm config images pull

```

You will see response as below:
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.28.2
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.28.2
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.28.2
[config/images] Pulled registry.k8s.io/kube-proxy:v1.28.2
[config/images] Pulled registry.k8s.io/pause:3.9
[config/images] Pulled registry.k8s.io/etcd:3.5.9-0
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.10.1


You can use these

kubeadm init options that are used to bootstrap cluster.

--control-plane-endpoint :  set the shared endpoint for all control-plane nodes. Can be DNS/IP
--pod-network-cidr : Used to set a Pod network add-on CIDR
--cri-socket : Use if have more than one container runtime to set runtime socket path
--apiserver-advertise-address : Set advertise address for this particular control-plane node's API server

If you don’t have a shared DNS endpoint, use this command:

```cmd

sudo kubeadm init \
  --pod-network-cidr=10.10.0.0/16

```

To bootstrap with a shared DNS endpoint, use this command. Note that the DNS name is the control plane API. If you don’t have the DNS record mapped to the API endpoint, you can get away with adding it to the /etc/hosts file.

```cmd

sudo vim /etc/hosts

```

10.148.0.3 k8s-node1
10.148.0.4 k8s-node2
10.148.0.5 k8s-master

Now create the cluster

```cmd

sudo kubeadm init \
  --pod-network-cidr=10.10.0.0/16 \
  --upload-certs \
  --control-plane-endpoint=k8s-master

```

This is the output:

root@k8s-master:~# sudo kubeadm init \
  --pod-network-cidr=10.10.0.0/16
[init] Using Kubernetes version: v1.28.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.148.0.5]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [10.148.0.5 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [10.148.0.5 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 6.002504 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: vbjfm4.w2uiwqfvt7pqcban
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

'''cmd
  
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

Alternatively, if you are the root user, you can run:

```cmd

  export KUBECONFIG=/etc/kubernetes/admin.conf

```

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

```cmd

kubeadm join 10.148.0.5:6443 --token vbjfm4.w2uiwqfvt7pqcban \
        --discovery-token-ca-cert-hash sha256:2a28796a4a0569118a1b6af689c410ce42be99a6853b1989ed0d90228017ad67

```

### 7. Scheduling pods on Master
By default, your Kubernetes Cluster will not schedule pods on the control-plane node for security reasons. It is recommended you keep it this way, but for test environments or if you are running a single node cluster, you may want to schedule Pods on control-plane node to maximize resource usage.

Remove the taint using this command:

```cmd

kubectl taint nodes --all node-role.kubernetes.io/master-

```

You should see similar output to this:

$ kubectl taint nodes --all node-role.kubernetes.io/master-
node/ubuntusrv.citizix.com untainted
This will remove the node-role.kubernetes.io/master taint from any nodes that have it, including the control-plane node, meaning that the scheduler will then be able to schedule pods everywhere.

### 8. Install network plugin on Master
Network plugins in Kubernetes come in a few flavors:

CNI plugins: adhere to the Container Network Interface (CNI) specification, designed for interoperability.
Kubenet plugin: implements basic cbr0 using the bridge and host-local CNI plugins
In this guide we will configure Calico.

Calico is a networking and network policy provider. Calico supports a flexible set of networking options so you can choose the most efficient option for your situation, including non-overlay and overlay networks, with or without BGP. Calico uses the same engine to enforce network policy for hosts, pods, and (if using Istio & Envoy) applications at the service mesh layer.

Install the Tigera Calico operator and custom resource definitions.

```cmd

kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml

```

Install Calico by creating the necessary custom resource. 

```cmd

kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml

```

Confirm that all of the pods are running with the following command.

```cmd

watch kubectl get pods -ncalico-system

```

Confirm master node is ready:

```cmd

kubectl get nodes -o wide

```

NAME                    STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
ubuntusrv.citizix.com   Ready    control-plane,master   23m   v1.23.3   10.2.40.239   <none>        Ubuntu 20.04.3 LTS   5.11.0-1019-aws   cri-o://1.22.1

### 9. Add worker nodes
With the control plane ready you can add worker nodes to the cluster for running scheduled workloads.

The join command that was given is used to add a worker node to the cluster.

```cmd

kubeadm join 10.148.0.5:6443 --token vbjfm4.w2uiwqfvt7pqcban \
        --discovery-token-ca-cert-hash sha256:2a28796a4a0569118a1b6af689c410ce42be99a6853b1989ed0d90228017ad67 

```

### 10. Deploying an application
Deploy sample Nginx application

```cmd

kubectl create deploy nginx --image nginx:latest

```

deployment.apps/nginx created

Check to see if pod started

```cmd

kubectl get pods

```

NAME                     READY   STATUS    RESTARTS   AGE
nginx-7c658794b9-t2fkc   1/1     Running   0          4m39s

### 11. RBAC

Add Role
```cmd

kubectl label node <node name> node-role.kubernetes.io/<role name>=<key>

```

Remove Role

```cmd

kubectl label node <node name> node-role.kubernetes.io/<role name>-

```

### Conclusion
In this guide we managed to set up a kubernetes server on an Ubuntu 22.04 server. You can now deploy apps in it.
