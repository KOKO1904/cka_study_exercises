# Simple gateway

### First, we create the gateway class, which is automatically generated during the installation of NGINX Fabric.

```bash
# Gateway Class
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controller: k8s.nginx.org/gateway-controller
```

### Next, create the gateway

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: production
spec:
  gatewayClassName: nginx
  listeners:
  - port: 80
    protocol: HTTP
    name: http
```

### Finally, we create the http route

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: webapp-route
  namespace: production
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        value: /
        type: PathPrefix
    backendRefs:
    - name: webapp-route
      port: 80
```
