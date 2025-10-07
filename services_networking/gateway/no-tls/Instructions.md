# Convert an Ingress to a Gateway API in Kubernetes

---

### You are given a Kubernetes cluster with multiple applications deployed, each with its own Deployment, Service, and ConfigMap. Currently, traffic routing is handled using an Ingress with the NGINX Ingress Controller. Your task is to replace the Ingress with a Gateway API setup using NGINX Gateway Fabric.

Current setup includes:

- web-app and dashboard-app accessible via app.company.local

- orders-app and reports-app accessible via orders.company.local

- An Ingress named company-ingress routing traffic to the appropriate services based on path prefixes.

Objectives:

- Remove or disable the existing Ingress.

- Install and configure NGINX Gateway Fabric in the cluster.

- Define the necessary GatewayClass, Gateway, and HTTPRoute resources to replicate the routing behavior of the original Ingress.

Ensure that:

- app.company.local/ → web-app

- app.company.local/dashboard → dashboard-app

- orders.company.local/ → orders-app

- orders.company.local/reports → reports-app

- Validate that all applications are accessible via their respective hostnames and paths.

You can use the documentantion:

https://gateway-api.sigs.k8s.io/guides/migrating-from-ingress/