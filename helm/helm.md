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
