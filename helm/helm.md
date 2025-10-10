# Helm

### Your manager has asked you to deploy Argo CD in a Kubernetes cluster using Helm. The goal is to test your ability to configure Helm charts using a values.yaml file and understand Kubernetes deployment options.

- Install the Argo CD chart using the repository: https://argoproj.github.io/argo-helm.

- Create a values.yaml file with the following configuration:
  - Enable redis-ha
  - Server:
    - Enable autoscaling with the next paramenters:
      - minReplicas: 2
      - maxReplicas: 3
      - Ingress:
        - Enabled with host argo-cd.local
        - Ingress classname traefik
        - Annotations: 
          - traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
          - traefik.ingress.kubernetes.io/router.middleware: "default-mymiddleware@kubernetescrd"
        - TLS using secret argo-cd-tls
  - repoServer:
    - Enable autoscaling with the next paramenters:
      - minReplicas: 2
      - maxReplicas: 4
  - applicationSet:
      - Change replicas to 2
- Create the chart named argo-chart using the values.yaml in the namespace argocd
- Verify that the settings defined in the values.yaml were successfully deployed.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm show values argo/argo-cd > argo-cd.yaml

# you can use for example: /^server:/ to efficiently search for through the YAML file
vim argo-cd.yaml

# values.yaml
redis-ha:
  enable: true
server:
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 3
  ingress:
    enabled: true
    ingressClassName: traefik
    annotations:
      traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
      traefik.ingress.kubernetes.io/router.middleware: default-mymiddleware@kubernetescrd
    hostname: argo-cd.local
    extraTls:
      - hosts:
        - argocd.example.com
        secretName: argo-cd-tls
repoServer:
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 4
applicationSet:
  replicas: 2

# We create the chart

kubectl create ns argocd
helm install -n argocd argo-chart argo/argo-cd -f values.yaml 

# We check if the changes were successful
helm get manifest argo-chart -n argocd | grep -i <value to search>

```

</p>
</details>

### Your team is developing a new application and decided to start with the database using a Master/Replica model deployed via Helm. For now, they want to deploy only the writer (master) node:

tasks:

- Create a Helm chart named app.
- PostgreSQL Master (Writer):
  - Create a StatefulSet template named writer in your Helm chart. 
    - Requirements:
      - 1 replica (master node)
      - Persistent Volume Claim for data storage in /var/lib/postgresql/data
      - Port 5432
      - Configure a Secret for PostgreSQL credentials (username, password, database)
      - Use a lightweight PostgreSQL image, e.g., bitnami/postgresql:15-debian-11-r0
      - Expose the writer via a ClusterIP Service for future readers to connect
- Verify your setup:
  - Use helm install to deploy the chart
  - Check that the writer pod is running
  - Ensure Persistent Volume is bound and the Secret is created

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
helm create app

# values.yaml
postgresql:
  image: bitnami/postgresql:15-debian-11-r0
  replicaCount: 1
  persistence:
    enabled: true
    size: 1Gi
  port: 5432
  credentials:
    username: postgres
    password: mysecurepassword
    database: mydatabase




# templates/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "app.fullname" . }}-secret
type: Opaque
stringData:
  POSTGRES_USER: {{ .Values.postgresql.credentials.username }}
  POSTGRES_PASSWORD: {{ .Values.postgresql.credentials.password }}
  POSTGRES_DB: {{ .Values.postgresql.credentials.database }}




# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "app.fullname" . }}-writer
spec:
  type: ClusterIP
  ports:
    - port: {{ .Values.postgresql.port }}
      targetPort: {{ .Values.postgresql.port }}
      name: postgres
  selector:
    app.kubernetes.io/name: {{ include "app.name" . }}
    role: writer




# templates/writer-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: {{ include "app.fullname" . }}-writer
spec:
  serviceName: {{ include "app.fullname" . }}-writer
  replicas: {{ .Values.postgresql.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "app.name" . }}
      role: writer
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "app.name" . }}
        role: writer
    spec:
      containers:
        - name: postgres
          image: {{ .Values.postgresql.image }}
          ports:
            - containerPort: {{ .Values.postgresql.port }}
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: {{ include "app.fullname" . }}-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "app.fullname" . }}-secret
                  key: POSTGRES_PASSWORD
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: {{ include "app.fullname" . }}-secret
                  key: POSTGRES_DB
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: {{ .Values.postgresql.persistence.size }}

# You can deploy the app by doing:
helm install app ./app
kubectl get pods
kubectl get pvc
kubectl get secret

```

</p>
</details>

### Your team already deployed the PostgreSQL writer using a StatefulSet. Now your manager wants to support replica readers, but only in environments where they are needed (e.g., staging/prod, not dev). To avoid maintaining multiple charts, the reader should be deployed only when enabled via values.yaml, using Helm conditionals. Also, replicas don’t need persistent identities, so they must use a Deployment, not a StatefulSet.

Tasks:

- Add this values to your values.yaml

```bash
reader:
  enabled: true   # If false, readers must not be deployed
  replicaCount: 3
  image:
    repository: bitnami/postgresql
    tag: 15-debian-11-r0
```

- Create a reader-deployment.yaml template that only renders if reader.enabled is true.

- Create a matching Service for the readers, also wrapped in a conditional.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
#Reader deployment
{{- if .Values.reader.enabled }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "app.fullname" . }}-reader
spec:
  replicas: {{ .Values.reader.replicaCount | default 1 }}
  selector:
    matchLabels:
      app: {{ include "app.name" . }}-reader
  template:
    metadata:
      labels:
        app: {{ include "app.name" . }}-reader
    spec:
      containers:
        - name: reader
          image: "{{ .Values.reader.image.repository }}:{{ .Values.reader.image.tag }}"
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: {{ include "app.fullname" . }}-secret
                  key: username
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "app.fullname" . }}-secret
                  key: password
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: {{ include "app.fullname" . }}-secret
                  key: database
          ports:
            - name: postgres
              containerPort: 5432
{{- end }}

# Service
{{- if .Values.reader.enabled }}
apiVersion: v1
kind: Service
metadata:
  name: {{ include "app.fullname" . }}-reader
spec:
  selector:
    app: {{ include "app.name" . }}-reader
  ports:
    - port: 5432
      targetPort: 5432
{{- end }}

```

</p>
</details>

