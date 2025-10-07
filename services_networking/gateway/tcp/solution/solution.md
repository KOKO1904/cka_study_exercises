# TCP

### First, we create the gateway


```bash
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: infra-tcp-gateway 
spec:
  gatewayClassName: nginx
  listeners:
  - name: rabbitmq
    protocol: TCP
    port: 7000
    allowedRoutes:
      kinds:
      - kind: TCPRoute
  - name: postgresql
    protocol: TCP
    port: 7001
    allowedRoutes:
      kinds:
      - kind: TCPRoute

```

### Finally, we create the tcp routes

**RabbitMQ**

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: TCPRoute
metadata:
  name: rabbitmq
spec:
  parentRefs:
  - name: infra-tcp-gateway 
    sectionName: rabbitmq
  rules:
    backendRefs:
    - name: messaging-svc
      port: 5672

```

**PostgreSQL**
```bash
apiVersion: gateway.networking.k8s.io/v1
kind: TCPRoute
metadata:
  name: postgresql
spec:
  parentRefs:
  - name: infra-tcp-gateway 
    sectionName: postgresql
  rules:
    backendRefs:
    - name: db-svc
      port: 5432
```
