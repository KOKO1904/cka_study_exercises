# Restrict Backend Pods to Frontend-Only Ingress

### This NetworkPolicy restricts all incoming traffic to pods labeled app=backend in the prod namespace, except from pods labeled app=frontend in the same namespace.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: case1
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
```

### We test using the curl-client pod

```bash
kubectl get pods -o wide

kubectl exec -it curl-client -n prod -- curl http://<caddy-server-ip>:80
```
