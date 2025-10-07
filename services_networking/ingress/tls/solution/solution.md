# Ingress with TSL solution

### First we create the deployments

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha
  template:
    metadata:
      labels:
        app: alpha
    spec:
      containers:
      - name: alpha
        image: nginx
        ports:
        - containerPort: 80

apiVersion: apps/v1
kind: Deployment
metadata:
  name: beta
spec:
  replicas: 1
  selector:
    matchLabels:
      app: beta
  template:
    metadata:
      labels:
        app: beta
    spec:
      containers:
      - name: beta
        image: nginx
        ports:
        - containerPort: 80
```

### Next, we expose the deployments.

```bash
apiVersion: v1
kind: Service
metadata:
  name: alpha
spec:
  selector:
    app: alpha
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80


apiVersion: v1
kind: Service
metadata:
  name: beta
spec:
  selector:
    app: beta
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80

# Verify the service endpoints:
kubectl get endpoints  # Ensure that the associated pods are correctly selected
```
### We manually create a Secret of type tls; alternatively, you can use cert-manager to generate it automatically.

```bash
#Creating the secret
openssl genrsa -out example.local.key 2048
openssl req -new -key example.local.key -out example.local.csr -subj "/CN=example.local"
openssl x509 -req -in example.local.csr -signkey example.local.key -out example.local.crt -days 365

kubectl create secret tls my-secret --cert=example.local.crt --key=example.local.key
```


### We create the ingress - In traefik v2 we also need to create a middleware to reedirect to / because nginx images don't have /alpha and /beta endpoints

```bash
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: connection-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
    traefik.ingress.kubernetes.io/router.middlewares: default-my-middleware@kubernetescrd
spec:
  ingressClassName: traefik
  tls:
  - hosts:
    - example.local
    secretName: my-secret 
  rules:
  - host: example.local
    http:
      paths:
      - path: /alpha
        pathType: Prefix
        backend:
          service:
            name: alpha
            port:
              number: 80
      - path: /beta
        pathType: Prefix
        backend:
          service:
            name: beta
            port:
              number: 80

# Middleware
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: my-middleware
spec:
  replacePath:
    path: "/"
```

### Finally we test the application with:

```bash
minikube tunnel

curl -k https://example.local/alpha
curl -k https://example.local/beta
```
