# Install api gateway

```bash

# Install api Gateway
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.1.0" | kubectl apply -f -

# Install NGINX Gateway Fabric
kubectl apply --server-side -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.1.0/deploy/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.1.0/deploy/default/deploy.yaml

```

# You can use cert-manager crd to generate auto-signed certificates (optional)

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```
