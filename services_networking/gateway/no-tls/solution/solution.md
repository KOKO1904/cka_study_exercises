# Api Gateway

### If you install api gateway using controller.md with the nginx fabric, you will get

```bash

# You can use this controller in gateways resources
NAME    CONTROLLER                                   ACCEPTED   AGE
nginx   gateway.nginx.org/nginx-gateway-controller   True       55s

```

### We create the gateway resource

```bash

# Gateway definition for company apps
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: company-gateway
spec:
  gatewayClassName: nginx  # We specify the gateway class
  listeners:
  - name: http-app
    protocol: HTTP
    port: 80
    hostname: app.company.local
  - name: http-orders
    protocol: HTTP
    port: 80
    hostname: orders.company.local

```

### The next step is to create separate http routes per hostname

```bash

# app.company.local
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-app
spec:
  parentRefs:
  - name: company-gateway
  hostnames:
  - app.company.local
  rules:
  - matches:
    - path:
        value: /
        type: PathPrefix
    backendRefs: 
    - name: web-app
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
  - matches:
    - path:
        value: /dashboard
        type: PathPrefix
    backendRefs:
    - name: dashboard-app
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /


---

# orders.company.local
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-orders
spec:
  parentRefs:
  - name: company-gateway
  hostnames:
  - orders.company.local
  rules:
  - matches:
    - path:
        value: /
        type: PathPrefix
    backendRefs: 
    - name: orders-app
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
  - matches:
    - path:
        value: /reports
        type: PathPrefix
    backendRefs:
    - name: reports-app
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /

```

### Finally, with minikube just do minikube tunnel to make the load balancer work

```bash
# Remove ingress
kubectl delete ingress company-ingress

# Delete the namespace ingress-nginx(can take a while)
kubectl delete ns ingress-nginx --grace-period=0 --force

minikube tunnel

# In another terminal instance, get the nginx-gateway in namespace nginx-gateway external ip and write it in /etc/hosts like this:

<ip>   app.company.local orders.company.local

# Finally we test the gateway with:

curl http://app.company.local/
curl http://app.company.local/dashboard
curl http://orders.company.local/
curl http://orders.company.local/reports
```


