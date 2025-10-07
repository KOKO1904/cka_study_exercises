# Cross-Namespace Routing with Kubernetes Gateway API

## We first define three namespaces:

- blog-ns → used by the team-blog team to deploy their blog-related applications.

- shop-ns → used by the team-shop team to deploy their store-related applications.

- infra-team → a support namespace for the infrastructure team, which does not need to expose services through the Gateway.

- The namespaces blog-ns and shop-ns are labeled with gateway-access: "enabled". This label acts as an access control mechanism: only namespaces with this label are allowed to attach their routes (HTTPRoutes) to the shared Gateway.
  
- The infra-team namespace is created without this label, meaning that even if it hosts internal services, it cannot bind routes to the Gateway.

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: infra-team

---

apiVersion: v1
kind: Namespace
metadata:
  name: blog-ns
  labels:
    secure-gateway: "true"

---

apiVersion: v1
kind: Namespace
metadata:
  name: shop-ns
  labels:
    secure-gateway: "true"

```

## Next, we update the setup.yaml manifest so that each resource is deployed into the right namespace:

- The blog-related resources (such as blog-svc) are placed in the blog-ns namespace.

- The store-related resources (such as store-svc) are placed in the shop-ns namespace.

- The shared service (home-svc) is placed in the shared-ns namespace.

- The Gateway resource itself is placed in the infra-team namespace, since it is managed by the infrastructure team.

- This separation ensures that each team only manages its own workloads, while the infrastructure team centrally manages the shared Gateway.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: home
  namespace: blog-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: home
  template:
    metadata:
      labels:
        app: home
    spec:
      containers:
      - name: home
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: home-content
          mountPath: /usr/share/nginx/html/
      volumes:
      - name: home-content
        configMap:
          name: home-configmap

---


apiVersion: v1
kind: ConfigMap
metadata:
  name: home-configmap
  namespace: blog-ns
data:
  index.html: |
    <html><body><h1>Welcome to home page</h1></body></html>

---

apiVersion: v1
kind: Service
metadata:
  name: home-svc
  namespace: blog-ns
spec:
  selector:
    app: home
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog
  namespace: blog-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blog
  template:
    metadata:
      labels:
        app: blog
    spec:
      containers:
      - name: blog
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: blog-content
          mountPath: /usr/share/nginx/html/
      volumes:
      - name: blog-content
        configMap:
          name: blog-configmap

---


apiVersion: v1
kind: ConfigMap
metadata:
  name: blog-configmap
  namespace: blog-ns
data:
  index.html: |
    <html><body><h1>Welcome to blog page</h1></body></html>

---

apiVersion: v1
kind: Service
metadata:
  name: blog-svc
  namespace: blog-ns
spec:
  selector:
    app: blog
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: store
  namespace: shop-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: store
  template:
    metadata:
      labels:
        app: store
    spec:
      containers:
      - name: store
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: store-content
          mountPath: /usr/share/nginx/html/
      volumes:
      - name: store-content
        configMap:
          name: store-configmap

---


apiVersion: v1
kind: ConfigMap
metadata:
  name: store-configmap
  namespace: shop-ns
data:
  index.html: |
    <html><body><h1>Welcome to store page</h1></body></html>

---

apiVersion: v1
kind: Service
metadata:
  name: store-svc
  namespace: shop-ns
spec:
  selector:
    app: store
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

## After that, we create the gateway. We assume that the Gateway API CRDs are already installed in the cluster, and that the environment is running with NGINX Gateway Fabric (the controller implementation that processes these Gateway API resources). If you need to install it, refer to controllers.md

- The Gateway is deployed in the infra-team namespace, since it is managed by the infrastructure team. It defines:

- Listeners for handling incoming HTTP traffic on the desired host (app.example.com).

- Access restrictions, so that only namespaces labeled with gateway-access: "enabled" can attach their routes to this Gateway.

- This ensures a central point of control for routing, while still allowing each team to independently manage their own HTTPRoutes.

```bash
# First, we create the certificates using cert-manager

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: issuer-name
  namespace: infra-team
spec:
  selfSigned: {}

---

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-certificate
  namespace: infra-team
spec:
  secretName: my-secret
  dnsNames:
  - app.example.com
  issuerRef:
    kind: Issuer
    name: issuer-name

# Then we create the gateway

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: infra-team
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    hostname: "app.example.com"
    allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            secure-gateway: "true"
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "app.example.com"
    allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            secure-gateway: "true"
    tls:
      mode: Terminate
      certificateRefs:
      - name: my-secret

```

## Finally, we define the HTTPRoute resources that connect each team’s services to the shared Gateway. Each HTTPRoute is created inside the namespace of the team that owns the service:

- In shared-ns, an HTTPRoute sends requests from / on app.example.com to the home-svc service.

- In blog-ns, an HTTPRoute sends requests from /blog on app.example.com to the blog-svc service.

- In shop-ns, an HTTPRoute sends requests from /store on app.example.com to the store-svc service.

## Each HTTPRoute explicitly references the shared Gateway in the infra-team namespace using parentRefs. Because only namespaces labeled with gateway-access: "enabled" can attach routes, this guarantees that only blog-ns and shop-ns are allowed to bind routes to the Gateway, while other namespaces (like infra-team) cannot.

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-home
  namespace: blog-ns
spec:
  parentRefs:
  - name: my-gateway
    namespace: infra-team
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: home-svc
      port: 80

---

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-blog
  namespace: blog-ns
spec:
  parentRefs:
  - name: my-gateway
    namespace: infra-team
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /blog
    backendRefs:
    - name: blog-svc
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /

---

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: http-shop
  namespace: shop-ns
spec:
  parentRefs:
  - name: my-gateway
    namespace: infra-team
  hostnames:
  - app.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /shop
    backendRefs:
    - name: store-svc
      port: 80
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
```
