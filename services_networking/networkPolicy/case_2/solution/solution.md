# Allow Intra-Namespace Traffic While Denying External Access

### This NetworkPolicy should allow all pods within the same namespace to communicate freely.
### Note: You can inspect the labels of any namespace and you will see that almost all namespaces have a label called `kubernetes.io/metadata.name`.
### Using this label, we can apply a `namespaceSelector` to target a specific namespace when defining NetworkPolicy rules.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: dev
```

### To test this network policy, we need to exec to the busybox box pod and make a wget

```bash
kubectl exec -it busybox -n test -- sh

# You need the service IP or pod IP of apache-server
wget -qO- http://<apache-server-pod-ip>:80

# The connection should fail because the NetworkPolicy denies ingress from other namespaces.
```
