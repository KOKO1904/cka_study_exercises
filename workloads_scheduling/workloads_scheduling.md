# Workloads & Scheduling

## Sidecar containers

## Your company has a Deployment named webapp-prod in the default namespace. This Deployment runs a main container called web-server that writes log messages to a file located at /var/log/app.log every second.

Your task:

- Add a sidecar container to the Deployment named log-forwarder.
- The sidecar must:
  - Use the image fluent/fluentd:v1.16-3.
  - Mount a shared volume with the main container at /var/log/ to access logs.
  - Set the following environment variables:

  ```bash
  FLUENTD_ARGS = --no-supervisor -q
  FLUENTD_CONF = /fluentd/etc/fluent.conf
  ```
- Mount a ConfigMap containing the fluent.conf file into the sidecar container at /fluentd/etc/fluent.conf.
- Do not modify the main container web-server.
- The sidecar must run alongside the main container

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp-prod
  template:
    metadata:
      labels:
        app: webapp-prod
    spec:
      containers:
        - name: web-server
          image: alpine
          command:
            - sh
            - -c
            - |
              mkdir -p /var/log
              while true; do
                echo "$(date) log message" >> /var/log/app.log
                sleep 1
              done

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: default
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/app.log
      pos_file /var/log/fluentd.pos
      tag app.logs
      format none
    </source>

    <match **>
      @type stdout
    </match>
```

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp-prod
  template:
    metadata:
      labels:
        app: webapp-prod
    spec:
      containers:
        - name: web-server
          image: alpine
          command:
            - sh
            - -c
            - |
              mkdir -p /var/log
              while true; do
                echo "$(date) log message" >> /var/log/app.log
                sleep 1
              done
          volumeMounts:
            - name: log-volume
              mountPath: /var/log
        - name: log-forwarder
          image: fluent/fluentd:v1.16-3
          env:
          - name: FLUENTD_ARGS
            value: "--no-supervisor -q"
          - name: FLUENTD_CONF 
            value: "/fluentd/etc/fluent.conf"
          volumeMounts:
            - name: log-volume
              mountPath: /var/log
            - name: configMapVolume
              mountPath: /fluentd/etc
      volumes:
        - name: log-volume
          emptyDir: {}
        - name: configMapVolume
          configMap:
            name: fluentd-config
```

</p>
</details>

## Horizontal Pod Autoscaler

### You have a Deployment named web-app running in the namespace production with 2 replicas. Your goal is to create a Horizontal Pod Autoscaler (HPA) for this deployment that automatically scales the number of replicas based on CPU usage.

Requirements

- The HPA should maintain between 2 and 5 replicas.

- The target average CPU utilization should be 60%.

- The scaling behavior should:

  - Allow scaling up with a maximum of 1 replica increase per 30 seconds.

  - Allow scaling down with a maximum of 1 replica decrease per 1 minute.

  - Stabilize scaling down for 2 minutes (meaning it will not scale down faster than this window).

<details>
<summary>Show commands / answers</summary>
<p>

```bash
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
  namespace: production
spec:
  scaleTargetRef:
  - kind: Deployment
    name: web-app
    apiVersion: apps/v1
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      policies:
      - type: Pods
        value: 1
        periodSeconds: 30
      stabilizationWindowSeconds: 0
    scaleDown:
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
      stabilizationWindowSeconds: 120

```

</p>
</details>

## Priority Class

### The company TechNova Inc. requires that critical workloads be prioritized appropriately to ensure system stability and performance under resource pressure.

Your task:

- Identify the user-defined PriorityClass with the highest value currently configured in the cluster.

- Create a PriorityClass named critical-priority for critical workloads, setting its value to 5 less than the highest user-defined PriorityClass value found.

- Deploy a DaemonSet called log-agent in the monitoring namespace using the official image fluent/fluentd:v1.14.2.

- Assign the critical-priority PriorityClass to the log-agent DaemonSet pods.

- Verify that the DaemonSet rollout completes successfully and that pods are scheduled with the assigned priority.

Note: Under resource constraints, pods with lower priority in the monitoring namespace should be evicted to ensure the log-agent pods can run.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#First we identify the pod with the highest priority
k get pc -A | sort -nr

| NAME                   | VALUE       | GLOBAL-DEFAULT | AGE | PREEMPTIONPOLICY       |
|------------------------|-------------|----------------|-----|------------------------|
| system-node-critical   | 2000001000  | false          | 47h | PreemptLowerPriority   |
| system-cluster-critical| 2000000000  | false          | 47h | PreemptLowerPriority   | 

# The value that we must use is:

2000001000 - 5 = 2000000995

# Next we create the priorityclass

apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-priority
value: 2000000995
preemptionPolicy: PreemptLowerPriority


# Finally we create the daemon set

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: log-pod
  template:
    metadata:
      labels:
        app: log-pod
    spec:
      priorityClassName: critical-priority
      containers:
      - name: log-agent
        image: fluent/fluentd:v1.14.2
```

</p>
</details>

## Pod Scheduling — Affinity, Taints, and Tolerations

### Your company wants to deploy its applications with the following requirements:

- There are two nodes: controlplane and node01.

- controlplane has a taint: role=backend:NoSchedule.

- node01 has a label env=dev.

- Deploy four pods with these placement rules:

- **frontend** 
  - Image: Nginx
  - Labels: app=frontend
  - Requirement:
    - Should be co-located with the backend pod using PreferredDuringSchedulingIgnoredDuringExecution affinity (soft preference, scheduling can still happen elsewhere if necessary).

- **backend**
  - Image: busybox
  - Labels: app=backend
  - Requirement:
    - Should be able to run on controlplane using tolerations for the role=backend:NoSchedule taint.

- **cache**
  - Image: busybox
  - Labels: app=cache
  - Requirement:
    - Should avoid nodes with label env=dev using RequiredDuringSchedulingIgnoredDuringExecution node affinity (hard requirement, must not schedule there).

- **db**
  - Image: mysql
  - Labels: app=db
  - Requirement:
    - Should run on node01 using RequiredDuringSchedulingIgnoredDuringExecution node affinity (hard requirement) without any tolerations.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
# Frontend
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 30
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - backend
        topologyKey: kubernetes.io/hostname
  containers:
  - name: frontend
    image: nginx
    ports:
    - containerPort: 80
  dnsPolicy: ClusterFirst
  restartPolicy: Never

---

#Backend
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  tolerations:
  - key: role
    operator: Equal
    value: backend
    effect: NoSchedule
  containers:
  - name: backend
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    ports:
    - containerPort: 80
  dnsPolicy: ClusterFirst
  restartPolicy: Never

---

# Cache
apiVersion: v1
kind: Pod
metadata:
  name: cache
  labels:
    app: cache
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: NotIn
            values:
            - dev
  containers:
  - name: cache
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    ports:
    - containerPort: 80
  dnsPolicy: ClusterFirst
  restartPolicy: Never

---

# DB
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: db
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - dev
  containers:
  - name: mysql
    image: mysql
    ports:
    - containerPort: 3306
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "12345678"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

  
</p>
</details>

## The company TechNova wants to deploy its applications with the following requirements:

- There are two nodes: master and worker01.

- master has a taint: role=admin:NoExecute.

- worker01 has a label tier=testing.

- Deploy four pods with these placement rules:

- **api**
  - Image: nginx
  - Labels: service=api
  - Requirements:
    - Should be scheduled together with the auth pod using PreferredDuringSchedulingIgnoredDuringExecution pod affinity (soft preference).
    - Should avoid nodes running the logger pod using PreferredDuringSchedulingIgnoredDuringExecution pod anti-affinity (soft preference).

- **auth**
  - Image: busybox
  - Labels: service=auth
  - Requirement:
    - Should tolerate the role=admin:NoExecute taint on the master node and be able to run there.

- **logger**
  - Image: busybox
  - Labels: service=logger
  - Requirements:
    - Should avoid nodes with label tier=testing using RequiredDuringSchedulingIgnoredDuringExecution node affinity (hard requirement).
    - Should only be scheduled on Linux nodes (kubernetes.io/os=linux) using RequiredDuringSchedulingIgnoredDuringExecution node affinity.

- **db**
  - Image: mysql
  - Labels: service=db
  - Requirements:
    - Should run on worker01 only using RequiredDuringSchedulingIgnoredDuringExecution node affinity (hard requirement).
    - Should prefer namespaces labeled env=staging using PreferredDuringSchedulingIgnoredDuringExecution namespace selector (soft preference).

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#api
apiVersion: v1
kind: Pod
metadata:
  name: api
  labels:
    service: api
spec:
  containers:
  - name: api
    image: nginx
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              service: auth
          topologyKey: "kubernetes.io/hostname"
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              service: logger
          topologyKey: "kubernetes.io/hostname"

#auth
apiVersion: v1
kind: Pod
metadata:
  name: auth
  labels:
    service: auth
spec:
  containers:
  - name: auth
    image: busybox
    command: ["sleep", "3600"]
  tolerations:
  - key: "role"
    operator: "Equal"
    value: "admin"
    effect: "NoExecute"

#logger
apiVersion: v1
kind: Pod
metadata:
  name: logger
  labels:
    service: logger
spec:
  containers:
  - name: logger
    image: busybox
    command: ["sleep", "3600"]
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "tier"
            operator: "NotIn"
            values: ["testing"]
          - key: "kubernetes.io/os"
            operator: "In"
            values: ["linux"]

#db
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    service: db
spec:
  containers:
  - name: db
    image: mysql
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - matchExpressions:
        - key: "tier"
          operator: "In"
          values: ["testing"]
    namespaceAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - matchExpressions:
        - key: "env"
          operator: "In"
          values: ["staging"]

```

</p>
</details>

## The company DataStream Inc. wants to deploy a static pod on a node called node01.

Requirements:

- The pod should run a simple nginx container.

- It must always run on node01, regardless of the scheduler.

- The pod should have the label app=webserver.

- The pod must restart automatically if the container crashes.

Tasks:

- Write the YAML manifest for the static pod.

- Explain where this YAML file should be placed on the node for Kubernetes to automatically manage it.

- Mention what component reads the static pod YAML and ensures it runs.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#YAML for the static pod (/etc/kubernetes/manifests/nginx-static.yaml on node01):
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  labels:
    app: webserver
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  restartPolicy: Always
```

</p>
</details>
