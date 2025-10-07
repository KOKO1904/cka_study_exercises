# Troubleshooting

## Renew the certificates

### You are the administrator of a Kubernetes cluster. The control plane certificates are about to expire, and kubeadm is unavailable. You need to regenerate the certificates manually using the cluster’s internal CA to ensure all components continue to communicate securely.

Tasks:

- Inspect the current certificate expiration dates
  - List all .crt files in /etc/kubernetes/pki and check their expiration dates using openssl.

- Generate a new private key for the API server
  - Use OpenSSL to create a new 2048-bit RSA key.

- Create a Certificate Signing Request (CSR)
  - Generate a CSR for the API server with the correct CN (common name).

- Sign the certificate with the internal Kubernetes CA
  - Use the existing CA certificate and CA key in /etc/kubernetes/pki to sign the CSR.
  - Set the validity period to 1 year (365 days).

- Replace the old certificate and key
  - Backup the old apiserver.crt and apiserver.key files.
  - Replace them with the newly generated ones.

- Restart the kubelet service
  - Apply the new certificates by restarting the kubelet on the control plane node.

- Repeat the process for other critical certificates:
  - etcd-server.crt
  - controller-manager.crt
  - scheduler.crt

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
# Api Server

# You can list all certificates in /etc/kubernetes/pki and verify their expiration dates.
for cert in /etc/kubernetes/pki/*.crt; do
    echo "$cert expires on:"
    sudo openssl x509 -in $cert -noout -enddate
done

# Generate private key
sudo openssl genrsa -out /etc/kubernetes/pki/apiserver.key 2048

# Generate CSR
sudo openssl req -new -key /etc/kubernetes/pki/apiserver.key \
  -out /etc/kubernetes/pki/apiserver.csr \
  -subj "/CN=kube-apiserver"

# Sign certificate with internal CA
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out /etc/kubernetes/pki/apiserver.crt \
  -days 365
  
# Etcd-server

# Generate private key
sudo openssl genrsa -out /etc/kubernetes/pki/etcd/etcd-server.key 2048

# Generate CSR
sudo openssl req -new -key /etc/kubernetes/pki/etcd/etcd-server.key \
  -out /etc/kubernetes/pki/etcd/etcd-server.csr \
  -subj "/CN=etcd-server"

# Sign certificate with internal CA
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/etcd-server.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out /etc/kubernetes/pki/etcd/etcd-server.crt \
  -days 365

# kube-controller-manager

# Generate private key
sudo openssl genrsa -out /etc/kubernetes/pki/controller-manager.key 2048

# Generate CSR
sudo openssl req -new -key /etc/kubernetes/pki/controller-manager.key \
  -out /etc/kubernetes/pki/controller-manager.csr \
  -subj "/CN=kube-controller-manager"

# Sign certificate with internal CA
sudo openssl x509 -req -in /etc/kubernetes/pki/controller-manager.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out /etc/kubernetes/pki/controller-manager.crt \
  -days 365

# kube-scheduler

# Generate private key
sudo openssl genrsa -out /etc/kubernetes/pki/scheduler.key 2048

# Generate CSR
sudo openssl req -new -key /etc/kubernetes/pki/scheduler.key \
  -out /etc/kubernetes/pki/scheduler.csr \
  -subj "/CN=kube-scheduler"

# Sign certificate with internal CA
sudo openssl x509 -req -in /etc/kubernetes/pki/scheduler.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out /etc/kubernetes/pki/scheduler.crt \
  -days 365

# Restart kubelet
sudo systemctl restart kubelet

```

</p>
</details>

## Issues with cluster initialization

### Someone on your team initialized a Kubernetes cluster and chose Calico as the network plugin. However, when you checked the pods in the kube-system namespace, you noticed that the Calico pods are not running properly:

| NAME                                    | READY | STATUS           | RESTARTS | AGE |
| --------------------------------------- | ----- | ---------------- | -------- | --- |
| calico-kube-controllers-fdf5f5495-vxpw5 | 0/1   | CrashLoopBackOff | 5        | 10m |
| canal-4xxhw                             | 0/2   | CrashLoopBackOff | 4        | 10m |
| canal-wlt99                             | 0/2   | CrashLoopBackOff | 4        | 10m |
| coredns-6ff97d97f9-4wljj                | 1/1   | Running          | 0        | 15m |
| coredns-6ff97d97f9-wmpcb                | 1/1   | Running          | 0        | 15m |
| kube-apiserver-controlplane             | 1/1   | Running          | 0        | 15m |
| kube-controller-manager-controlplane    | 1/1   | Running          | 0        | 15m |
| kube-scheduler-controlplane             | 1/1   | Running          | 0        | 15m |

Upon checking the logs in one of the Calico pods, you find:

```bash
ERROR: IP pool CIDR 10.244.0.0/16 is incompatible with PodCIDR 192.168.0.0/16 on this node
Failed to allocate pod IP: no available IPs in pool
```

Task:

As the cluster administrator, you are asked to resolve this issue and ensure that the Calico pods are running properly.

---

<details>
<summary>Show commands / answers</summary>
<p>
 
```bash
# Description of the error:
It’s actually a mismatch between the IP pool CIDR used by Calico (10.244.0.0/16) and the cluster’s PodCIDR (192.168.0.0/16).
This prevents Calico from allocating IPs to pods.
To fix it, you need to create or edit a custom-resources.yaml (Installation CRD) and set the IP pool CIDR to match the cluster’s PodCIDR, then apply it with kubectl apply -f custom-resources.yaml.

# From the log error, we can infer that the cluster was initialized using:
kubeadm init --pod-network-cidr=192.168.0.0/16

# Or you can use:
kubectl cluster-info dump | grep podCIDR

# Based on that, We edit the calico configuration to set the IP pool CIDR to match the cluster’s PodCIDR.:

apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - cidr: 192.168.0.0/16
      blockSize: 26
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

# Apply the corrected configuration
kubectl apply -f custom-resources.yaml

# Verify Calico pods are running
kubectl get pods -n kube-system
```

</p>
</details>


