# Isolating Pods Within a Namespace

### We need to create a NetworkPolicy that denies all ingress traffic from pods within the same namespace (jupiter) but allows traffic from pods in any other namespace.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: jupiter
spec:
  podSelector:
    matchLabels:
      app: metrics
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchExpressions:
          - key: kubernetes.io/metadata.name
            operator: NotIn
            values:
            - jupiter
```

### We start testing the nginx pod in saturn namespace

```bash

# Open a shell in the nginx pod
kubectl exec -it -n saturn nginx -- sh


# From inside the pod, try to curl the metrics pod in jupiter namespace
curl http://metrics-exporter.jupiter.svc.cluster.local:9100

```

### Now, we create a temporary busybox pod to start test

```bash

# Launch a temporary busybox pod in jupiter namespace
kubectl run -it --rm busybox --image=busybox --restart=Never -n jupiter -- sh

# From inside the busybox pod, try to curl the metrics pod
wget -qO- http://metrics-exporter.jupiter.svc.cluster.local:9100

```
