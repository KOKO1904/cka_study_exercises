# Simple Gateway

## You are tasked with setting up a Gateway API configuration to route HTTP traffic to a backend service called webapp running in the production namespace. <br>

Requirements:

- Create a GatewayClass named nginx-gatewayclass that uses the controller k8s.nginx.org/gateway-controller.

- Create a Gateway named nginx-gateway in the production namespace that:

  - Uses the nginx-gatewayclass.

  - Listens on port 80 for HTTP traffic.

- Create an HTTPRoute named webapp-route in the production namespace that:

  - Associates with the nginx-gateway.

  - Routes all traffic with path prefix / to the service webapp on port 80.

**Use minikube to practice this scenario, use minikube tunnel so the Gateway (usually of type LoadBalancer) gets an external IP** <br>
kubernetes.io > docs > concepts > services-networking > [Gateway](https://kubernetes.io/docs/concepts/services-networking/gateway/)

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash

# Install api Gateway
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.1.0" | kubectl apply -f -

# Install NGINX Gateway Fabric
kubectl apply --server-side -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.1.0/deploy/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.1.0/deploy/default/deploy.yaml

#Controller
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx-gatewayclass
spec:
  controller: k8s.nginx.org/gateway-controller

#Gateway
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: production
spec:
  gatewayClassName: nginx-gatewayclass
  listeners:
  - port: 80
    protocol: HTTP
    name: http

#HTTPRoute
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: webapp-route
  namespace: production
rules:
- matches:
  - path:
      value: /
      type: Prefix
  backendRefs:
    name: webapp-route
    port: 80


```

</p>
</details>

