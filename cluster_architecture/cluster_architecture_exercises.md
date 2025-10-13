# Cluster Architecture, Installation and Configuration (25%)

## Node Preparation for Kubeadm Setup

### Your manager has informed you that you need to prepare a node in order to install a Kubernetes cluster using the Calico CNI (Container Network Interface). Your task is to ensure the node is fully prepared for cluster deployment. This includes:

- Disabling swap memory, as Kubernetes requires swap to be off for proper operation.
- Configuring the necessary system settings, including kernel modules and sysctl parameters, to support networking for pods.
- Verifying that the node has a valid hostname, a reachable IP address, and can communicate with other nodes in the cluster.
- Ensuring that the required ports for Kubernetes and Calico networking are open, and that no conflicting network services are running.

**You can use killercoda playgrounds to practice this scenario** <br>
killercoda.com > Playgrounds > Scenario > Ubuntu > [Preparing a node](https://killercoda.com/playgrounds/scenario/ubuntu) <br>
kubernetes.io > docs > reference > setup-tools > [Install Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
# 1. Disable swap memory (required by Kubernetes)
sudo swapoff -a
# To make it permanent, comment out any swap entries in /etc/fstab
sudo sed -i.bak '/ swap / s/^/#/' /etc/fstab

# 2. Load required kernel modules
sudo modprobe br_netfilter
sudo modprobe overlay

# 3. Set sysctl parameters for Kubernetes networking
sudo sysctl -w net.bridge.bridge-nf-call-iptables=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1
sudo sysctl -w net.ipv4.ip_forward=1


# 4. Verify hostname and network connectivity
hostnamectl
ip addr show
ping <other-node-ip>  # check connectivity with other nodes if any

# 5. Ensure required ports are open
# Example: kube-apiserver (6443), etcd (2379-2380), kubelet (10250)
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250/tcp

# Node is now ready to initialize the cluster and deploy Calico

```

</p>
</details>




## Updating the cluster

### You are the administrator of a Kubernetes cluster currently running version 1.33.2. You want to upgrade the control plane node to version 1.33.5-1.1 using kubeadm.

Tasks:

- Prepare the node by allowing the kubeadm package to be upgraded.

- Update the package list and install the specific version 1.33.5-1.1 of kubeadm.

- Prevent the kubeadm package from being automatically upgraded in the future.

- Drain the control plane node to safely evict workloads before upgrading.

- Check the available upgrades and confirm that version 1.33.5-1.1 can be applied.

- Apply the upgrade to the control plane node using kubeadm.

- Update the kubelet package to the new version 1.33.5-1.1.

- Update the kubectl package to the new version 1.33.5-1.1.

- Restart the kubelet service to apply the changes.

- Uncordon the control plane node to make it schedulable again.

**You can use killercoda playgrounds to practice this scenario** <br>
killercoda.com > Playgrounds > Scenario > kubernetes > [Update cluster](https://killercoda.com/playgrounds/scenario/kubernetes)

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#Upgrading the control-plane
ssh control-plane

k get nodes
NAME    STATUS     ROLES           AGE   VERSION
node1   Ready   control-plane      2m    v1.33.2
node2   Ready      <none>          84s   v1.33.2

#Update kubeadm to matching version
sudo apt-mark unhold kubeadm
sudo apt update
sudo apt-cache madison kubeadm

kubeadm | 1.33.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
kubeadm | 1.33.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
kubeadm | 1.33.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
kubeadm | 1.33.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
kubeadm | 1.33.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages
kubeadm | 1.33.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.33/deb  Packages


sudo apt install kubeadm=1.33.5-1.1
sudo apt-mark hold kubeadm

#Check upgrade plan and confirm version 1.33.5-1.1 is available:
sudo kubeadm upgrade plan
# [upgrade] Fetching available versions to upgrade to
# [upgrade/versions] Cluster version: 1.33.5
# [upgrade/versions] kubeadm version: v1.33.5

sudo kubeadm upgrade apply 1.33.5


#Updating the kubelet - Make sure to create a new control-plane for this
kubeadm token create --print-join-command
kubeadm init phase upload-certs --upload-certs

# This give you something like this (We add the --control-plane flag because this node will temporarily act as a control plane while we update the original control plane.)
kubeadm join <IP-del-control-plane>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --certificate-key <key-obtenido-del-init-phase-upload-certs> --control-plane

kubectl drain control-plane --ignore-daemonsets
# node/kube-control-plane cordoned

sudo apt-mark unhold kubelet
sudo apt install -y kubelet=1.33.5-1.1
sudo systemctl daemon re-exec
sudo systemctl daemon-reload
sudo systemctl restart kubelet
sudo apt-mark hold kubelet

kubectl uncordon control-plane

#We remove the temporal node
kubectl drain <nodo-temporal> --ignore-daemonsets --delete-local-data
kubectl delete node <temporal-node>
(In the temporal node)
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes /var/lib/kubelet /var/lib/etcd /var/lib/cni
sudo systemctl stop containerd
sudo systemctl restart containerd
sudo systemctl stop kubelet
sudo systemctl disable kubelet

#Update kubectl to matching version
sudo apt-mark unhold kubectl
sudo apt install -y kubectl=1.33.5-1.1
sudo apt-mark hold kubectl

#Upgrading the worker nodes
sudo apt-mark unhold kubeadm
sudo apt update
sudo apt-cache madison kubeadm
sudo apt install -y kubeadm=1.33.5-1.1
sudo apt-mark hold kubeadm

#Upgrading the kubelet configuration
sudo apt-mark unhold kubelet
sudo kubeadm upgrade node

kubectl drain node2 --ignore-daemonsets

sudo apt install -y kubelet=1.33.5-1.1
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon node2
sudo apt-mark hold kubelet

sudo apt-mark unhold kubectl
sudo apt install -y kubectl=1.33.5-1.1
sudo apt-mark hold kubectl
```

</p>
</details>

## Backup and Restore etcd in a Kubernetes Cluster

### Your company has a Kubernetes cluster with a single control plane node. You’ve been asked to perform a backup of the etcd datastore and later simulate a disaster recovery scenario by restoring the cluster from that backup.

Your Tasks:

- Access the control plane node and locate the running etcd container or service.

- Create a snapshot backup of etcd and save it to /backup/etcd-snapshot.db.

- The certificate files are located in /var/lib/minikube/certs/etcd/

- The etcd server listens on https://127.0.0.1:2379

- Restore etcd using the snapshot previously created.

- The new etcd data directory will be in: /mnt/etcd-data

- Verify that the cluster is functional by checking the nodes and pods.

**You can use killercoda playgrounds to practice this scenario** <br>
killercoda.com > Playgrounds > Scenario > kubernetes > [Backing up the cluster](https://killercoda.com/playgrounds/scenario/kubernetes)

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
ssh control-plane

etcdctl version

k get pods -n kube-system
NAME                   READY   STATUS RESTARTS AGE
etcd-control-plane     1/1    Running    0     5s

#Look for --listen-client-urls in describe por endpoints
kubectl describe pod etcd-kube-control-plane -n kube-system
# We have that the endpoint is localhost and the port is 2379

 
sudo ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db --endpoints=https://127.0.0.1:2379 --cacert=/var/lib/minikube/certs/etcd/ca.crt --cert=/var/lib/minikube/certs/etcd/server.crt --key=/var/lib/minikube/certs/etcd/server.key
sudo ETCDCTL_API=3 etcdutl snapshot restore /backup/etcd-snapshot.db --data-dir /var/lib/etcd > restore.txt 2>&1

# Modify the new data-dir in this file under hostPath
vim /etc/kubernetes/manifests/etcd.yaml

- hostPath:
    path: /var/lib/etcd-restore # CHANGE THIS LINE
    type: DirectoryOrCreate
  name: etcd-data

# checking nodes
kubectl get nodes
```

</p>
</details>

## Setting Up Containerd in the Cluster

### Your manager created an EC2 instance with an Ubuntu AMI in AWS. He installed Docker via the Ubuntu repositories and tasked you, as the cluster manager, with installing kubeadm using containerd as the container runtime.

Your tasks:

- SSH into the EC2 instance (public IP: 10.0.0.1).

- Configure containerd (note: Docker installs containerd by default).

- Enable the systemd cgroup driver for containerd.

- Configure kubelet to explicitly use the containerd CRI socket.

- Restart kubelet and ensure it is running with the correct container runtime.

- Verify that the node appears as Ready.

**You can use killercoda playgrounds to practice this scenario** <br>
killercoda.com > Playgrounds > Scenario > Ubuntu > [Containerd Practice](https://killercoda.com/playgrounds/scenario/ubuntu) <br>
kubernetes.io > docs > reference > setup-tools > [Install Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash

# Step 1: SSH to the node and configure containerd
ssh ubuntu@10.0.0.1

# Generate the containerd configuration
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# Enable cgroup driver
sed -i 's/SystemdCgroup = false/SystemdCgroup = true' /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd

# Install kubeadm, kubelet, and kubectl (if not installed)
# Follow official documentation: https://kubernetes.io/docs/reference/setup-tools/kubeadm/

# Once installed, initialize the cluster and specify containerd as the container runtime
kubeadm init --apiserver-advertise-address=<hostname> --cri-socket=unix:///var/run/containerd/containerd.sock

# Verify that kubelet is running with the correct container runtime
sudo systemctl status kubelet

# Finally we check if the installation was successful
kubectl get nodes -o wide

```

</p>
</details>

## Setting Up CRI-O in the Cluster

### Your manager has tasked you with preparing a control-plane node that previously had a Kubernetes cluster. You need to remove all existing configurations and then install a fresh, clean cluster using kubeadm and the container runtime CRI-O.

Tasks:

- Stop all Kubernetes-related services and the container runtime.

- Run kubeadm reset to clean up the existing cluster.

- Remove all residual Kubernetes and CNI directories.

- Clear iptables rules related to Kubernetes and CNI.

- Delete virtual network interfaces created by CNI plugins (Flannel, Calico, etc.).

- Configure kubelet to explicitly use the appropriate CRI socket.

- Initialize a new cluster with kubeadm init, using a configuration file that specifies the CRI socket.

- Set up kubectl for the user managing the cluster.

- Verify the cluster is operational and that nodes are ready (kubectl get nodes).

Note: Install cri-o using this link: https://download.opensuse.org/repositories/isv:/cri-o:/stable:/v1.33/deb/amd64/cri-o_1.33.0-2.1_amd64.deb

**Use a VM or WSL to practice this scenario, because in Killercoda you cannot fully switch the container runtime to CRI-O.** <br>

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
# To clean the cluster you need to type:
sudo kubeadm reset -f

# Remove residual folders
rm -rf /etc/cni/net.d
rm -rf /var/lib/kubelet/*
rm -rf /var/lib/cni
rm -rf /var/lib/etcd
rm -rf /var/run/kubernetes

# Clear ip table routes
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X

# Delete virtual network interfaces
ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:21:27:19 brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:14:90:41:5e brd ff:ff:ff:ff:ff:ff
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether 8e:80:1e:29:91:ee brd ff:ff:ff:ff:ff:ff
5: cali6f0dd2aae09@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-47df0d66-bd87-8da6-db99-5db44b54f5e6
8: calic4f3c10e5e1@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-c7a063e4-5406-5891-1d72-61761268f46b


sudo ip link delete flannel.1
sudo ip link delete cali6f0dd2aae09
sudo ip link delete calic4f3c10e5e1

# If you have containerd installed, its important to stop it
sudo systemctl stop containerd

# We are going to use crio as the crio runtime

curl -Lo cri-o_1.33.0-2.1_amd64.deb https://download.opensuse.org/repositories/isv:/cri-o:/stable:/v1.33/deb/amd64/cri-o_1.33.0-2.1_amd64.deb
sudo dpkg -i cri-o_1.33.0-2.1_amd64.deb

# We enable the service
sudo systemctl start crio
sudo systemctl enable crio

# We configure crio to make him use systemd
mkdir -p /etc/crio
crio config default | sudo tee /etc/crio/crio.conf > /dev/null

# We enable the cgroup driver in crio
# Remove the hash
cgroup_manager = "systemd"

# Restart the crio service
sudo systemctl restart crio

# we configure kubeadm to use crio
kubeadm init --apiserver-advertise-address=<ip> --cri-socket=unix:///var/run/crio/crio.sock

# Change kubelet config to use the crio runtime (skip this step if you didnt use the parameter --cri-socket in kubeadm init)
vim /var/lib/kubelet/kubeadm-flags.env 

KUBELET_KUBEADM_ARGS="--container-runtime-endpoint=unix:///var/run/crio/crio.sock --pod-infra-container-image=registry.k8s.io/pause:3.10"

# Once saved, check if kubelet is using crio
ps -ef | grep kubelet

# We restart kubelet
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart kubelet

Note: If you're using killercoda, you will get this, even if you configure crio(kubectl get nodes -o wide):
NAME           STATUS     ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   NotReady   control-plane   8m30s   v1.33.2   172.30.1.2    <none>        Ubuntu 24.04.3 LTS   6.8.0-51-generic   containerd://1.7.27
# this is because kubelet in Killercoda is tied to containerd, it will always show containerd://, regardless of any CRI-O configuration you try
# The node is NotReady because a CNI network plugin for pods has not been installed and declared.
```

</p>
</details>

## Setting Up Cri-Docker in the Cluster

### An older computer is being used that cannot run the latest versions of Docker. Docker was installed through the Ubuntu docker.io repositories, but due to compatibility limitations, CRI-Docker will be used as the container runtime for Kubernetes to ensure proper functionality and performance. You are tasked with installing cri-dockerd v0.2.0 and configuring it as a systemd service.

Tasks:

- Install cri-docker using this link: https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.0/cri-dockerd-v0.2.0-linux-amd64.tar.gz
- Configure the CRI Docker service using the following systemd parameters:
  - Use the docker.sock as the container runtime endpoint
  - Use the the cni plugin network
  - Use the pod-infra-container-image: k8s.gcr.io/pause:3.6
- Configure Docker’s daemon.json to use systemd as the cgroup driver.

**You can use killercoda playgrounds to practice this scenario** <br>
killercoda.com > Playgrounds > Scenario > Ubuntu > [Container Runtime](https://killercoda.com/playgrounds/scenario/ubuntu) <br>

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#We download the file
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.0/cri-dockerd-v0.2.0-linux-amd64.tar.gz

#We extract the file
tar -xzvf cri-dockerd-v0.2.0-linux-amd64.tar.gz

#We move the extracted cri-dockerd to /usr/local/bin
mv cri-dockerd /usr/local/bin

# Next, we create the systemd service

sudo vim /usr/lib/systemd/system/cri-docker.service
#Cri-docker service
[Unit]
Description=Cri-docker service
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/cri-dockerd  \
  --docker-endpoint=unix:///var/run/docker.sock \
  --network-plugin=cni \
  --pod-infra-container-image=k8s.gcr.io/pause:3.6
Restart=always

[Install]
WantedBy=multi-user.agent

# To enable Cgroupdriver we need to edit /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

# The systemd service will create a cri-dockerd.sock and you will use it with kubeadm

kubeadm init --cri-socket=unix:///var/run/cri-dockerd.sock

```

</p>
</details>

## Advanced RBAC: Creating a Role, RoleBinding, and Modifying Kubeconfig

### Set up a new Kubernetes user named alice with limited access to the projectx namespace. The user should have permissions to perform the following verbs on pods in that namespace:  list, get, update, delete, and create. You'll generate credentials, configure RBAC, and create a kubeconfig for access.

**Use minikube to practice this scenario** <br>

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#Generate the certificates
openssl genrsa -out alice.key 2048
openssl rsa -in alice.key -check
openssl req -new -key alice.key -out alice.csr -subj "/CN=alice/O=developer-group"

cat alice.csr | base64 | tr -d '\n'

#Upload the csr to kubernetes cluster | certificate.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: alice-certificate
spec:
  request: <csr token>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

# Let kubernetes sign the certificate
kubectl apply -f certificate.yaml
kubectl certificate approve alice-certificate

#Get the cert
kubectl get csr alice-certificate -o jsonpath='{.status.certificate}' | base64 -d > alice.crt

#Create the role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developers
  namespace: projectx
rules:
  apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "update", "delete", "create"]

or

kubectl create role developers  --resource=pods --verbs=list,get,update,delete,create

#Create the rolebinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developers-binding
  namespace: projectx
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developers
  apiGroup: rbac.authorization.k8s.io

or

kubectl create rolebinding developers-binding --role=developers --user=alice

# Configure kubeconfig

apiVersion: v1
kind: Config
clusters:
- name: kubernetes
  cluster:
    server: https://<apiserver:6443>
    certificate-authority: /etc/kubernetes/pki/ca.crt
contexts:
- name: alice-context
  context:
    user: alice
    cluster: kubernetes
    namespace: projectx
users:
- name: alice
  user:
    client-certificate: <certificate>
    client-key: <clientkey>

current-context: alice-context

# Install kubectl on the developer machine, mv the kubeconfig.yaml to ~/.kube/config, hand it over ca.crt, alice.crt and alice.key
# Test with kubectl auth can-i get pods --as=alice -n development

```

</p>
</details>

## Advanced RBAC: Creating a ServiceAccount and Modifying Kubeconfig

### Set up a new Kubernetes ServiceAccount named deploybot in the appenv namespace. The ServiceAccount should have permissions to perform the following verbs on deployments and replicasets in that namespace: list, get, update, delete, and create. You will generate a token, configure RBAC, and create a kubeconfig for access.

**Use minikube to practice this scenario** <br>

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#First we create the namespace and the serviceaccount
kubectl create ns appenv
kubectl create sa deploybot

#Then we generate a token for that ServiceAccount.
kubectl create token deploybot -n appenv

#We create the role and the rolebinding
# We have 2 options: (via kubectl and manual)
kubectl create role deploymanager --resource=deployments,replicasets --verb=list,get,update,create,delete -n appenv
kubectl create rolebinding deploymanager-binding --role=deploymanager --serviceaccount=appenv:deploybot -n appenv

or

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deploymanager
  namespace: appenv
rules:
  apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "update", "create", "delete"]

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deploymanager-binding
  namespace: appenv
subjects:
- kind: ServiceAccount
  name: deploybot
  namespace: appenv
roleRef:
  kind: Role
  name: deploymanager
  apiGroup: rbac.authorization.k8s.io

Then we create the kubeconfig but we are going to use a base64 certificate authority
base64 /etc/kubernetes/pki/ca.cert

apiVersion: v1
kind: Config
clusters:
- name: kubernetes
  cluster:
    server: https://<apiserverip>:6443
    certificate-authority-data: <base64>
contexts:
- name: bot-context
  context:
    cluster: kubernetes
    user: deploybot
    namespace: appenv
user:
- name: deploybot
  user:
    token: <token>

current-context: bot-context

```

</p>
</details>

