# Storage

### Utilize persistent and ephemeral volumes:

Important Notes:
- PVs are cluster-scoped resources.
- PVC and Pod or other resource must be in the same namespace to bind and work correctly.
- PV must be Available to bind; once Bound, it can’t bind again until released.
- PV and PVC must have matching StorageClass and compatible AccessModes (RWO, ROX, RWX).
- PVC’s storage request cannot exceed PV’s capacity.
- Ephemeral volumes are for temporary storage, no PV/PVC needed.
- When bound, PV status changes to Bound and links to the PVC.
- Deleting PVC triggers PV’s reclaimPolicy: delete, recycle, or retain.

## PV and PVC

### Your company requires you to configure storage and deploy an application in their Kubernetes cluster using an NFS backend. You have access to an NFS server at IP 10.0.0.100, exporting the path /exports/data.

Your tasks are:

- Create a PersistentVolume (PV) named nfs-pv that uses the NFS backend at 10.0.0.100:/exports/data, with:
  - Capacity: 5Gi
  - Access mode: ReadWriteMany
- Create a StorageClass named nfs-storage that uses the hypothetical non-dynamic NFS provisioner example.com/nfs. Configure it with a reclaim policy of Retain and set the volume binding mode to Immediate.
- Create a PersistentVolumeClaim (PVC) named nfs-pvc requesting 2Gi of storage using the nfs-storage StorageClass.
- Deploy an nginx Deployment named webapp with 2 replicas in the default namespace.
- Mount the PVC to the pods at the path /usr/share/nginx/html.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash

# Note:
# If you don’t remember the exact structure of a PersistentVolume, StorageClass, or other Kubernetes resources, you can use these commands to explore their definitions and available API resources:

kubectl api-resources | grep storageclass
kubectl api-resources | grep persistentvolume
kubectl explain storageclass --recursive
kubectl explain persistentvolume --recursive

#These commands help you review the resource fields and hierarchy directly from the Kubernetes API documentation.

# First we need to create the storageclass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: example.com/nfs
reclaimPolicy: Retain
volumeBindingMode: Immediate

# Next we create the pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteMany
  storageClassName: nfs-storage
  nfs:
    path: /exports/data
    server: 10.0.0.100

# Then we create the pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  resources:
    requests:
      storage: 2Gi
  accessModes:
  - ReadWriteMany
  storageClassName: nfs-storage

# Finally we create the deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: myvolume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: myvolume
        persistentVolumeClaim:
          claimName: nfs-pvc

```

</p>
</details>

## Storage Classes

### The company CloudShift Inc. has instructed its DevOps team to create a StorageClass named csi-retain-sc with the following specifications:

- Use provisioner: csi-driver.example-vendor.example
- Set this StorageClass as default
- reclaimPolicy set to Retain
- Allow volume expansion
- Include the parameter guaranteedReadWriteLatency: "true"
- Add mount option discard
- Use volumeBindingMode: WaitForFirstConsumer

Save the configuration as sc-default-retain.yaml and apply it to the cluster.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-retain-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true"
mountOptions:
- discard
volumeBindingMode: WaitForFirstConsumer
```

</p>
</details>

###  The company CloudShift Inc. requests verification of which StorageClass is currently set as default.

You must:

- Run the appropriate Kubernetes command

- Provide the output showing the correct storageclass.kubernetes.io/is-default-class annotation.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
kubectl get storageclass -o custom-columns=Name:.metadata.name,Default:.metadata.annotations."storageclass\.kubernetes\.io/is-default-class"
```

</p>
</details>

### Create a dynamic StorageClass named my-storage using the provisioner dynamic.com/provisioner, and set it as the default StorageClass. Then, create a PersistentVolumeClaim named my-pvc requesting 4 Gi of storage with access mode ReadWriteOnce, using the my-storage StorageClass. Finally, create an Nginx Pod named nginx that mounts this PVC at the path /pvc/data/.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash

#Storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage
  storageclass.kubernetes.io/is-default-class: "true"
provisioner: dynamic.com/provisioner

# Note: You don´t need to create a pv because is a dynamic provisioner

# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  resources:
    requests:
      storage: 4Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: my-storage

# Nginx pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: my-volume
      mountPath: /pvc/data
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: my-pvc


```

</p>
</details>
