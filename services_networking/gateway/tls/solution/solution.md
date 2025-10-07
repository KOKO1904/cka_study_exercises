# Convert an Ingress to a Gateway API in Kubernetes with cert-manager

### First we install the cert-manager, the link is in the controller.md in this repository and then we create an issuer:

```bash

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: default
spec:
  selfSigned: {}

```

### Once we had the issuer, we need to create a certificate


```bash

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: multi-cert
  namespace: default
spec:
  secretName: multi-cert
  dnsNames:
  - app.home.local
  - app.contact.local
  issuerRef:
    name: selfsigned-issuer
    kind: Issuer

```

### Next, we had install the api gateway crd, link in controller md. After that, we create a gateway(Only the gateway manages the tsl):


```bash

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http-home
    protocol: HTTP
    port: 80
    hostname: app.home.local
  - name: https-home
    protocol: HTTPS
    port: 443
    hostname: app.home.local
    tls:
      mode: Terminate
      certificateRefs:
        kind: Secret
        secretName: multi-cert
  - name: http-contact
    protocol: HTTP
    port: 80
    hostname: app.contact.local
  - name: https-contact
    protocol: HTTPS
    port: 443
    hostname: app.contact.local
    tls:
      mode: Terminate
      certificateRefs:
        kind: Secret
        name: multi-cert


```

### The next step is to create separate http routes per hostname

```bash

# app.home.local
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-home
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - app.home.local
  rules:
  - matches:
    - path:
        value: /
        type: PathPrefix
    backendRefs: 
    - name: home-svc
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
  - matches:
    - path:
        value: /sales
        type: PathPrefix
    backendRefs:
    - name: sales-svc
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /


---

# app.contact.local
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-contact
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - app.contact.local
  rules:
  - matches:
    - path:
        value: /
        type: PathPrefix
    backendRefs: 
    - name: contact-svc
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
  - matches:
    - path:
        value: /about
        type: PathPrefix
    backendRefs:
    - name: about-us-svc
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
minikube tunnel

# In another terminal instance, get the nginx-gateway in namespace nginx-gateway external ip and write it in /etc/hosts like this:

<ip>   app.home.local app.contact.local

# Finally we test the gateway with:

curl http://app.home.local/
curl http://app.home.local/sales
curl http://app.contact.local/
curl http://app.contact.local/about

```
