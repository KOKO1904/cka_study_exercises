# Convert an Ingress to a Gateway API in Kubernetes

### You are given a Kubernetes cluster with multiple applications deployed, each with its own Deployment, Service, and ConfigMap. Currently, traffic routing is handled using an Ingress with the NGINX Ingress Controller. Your task is to replace the Ingress with a Gateway API setup using NGINX Gateway Fabric, and configure TLS using cert-manager.

Current setup includes:

- home-svc and sales-svc accessible via app.home.local

- contact-svc and about-us-svc accessible via app.contact.local

- An Ingress named my-app-ingress routing traffic based on path prefixes /, /sales, /contact, /about.

Objectives:

- Remove or disable the existing Ingress.

- Install and configure NGINX Gateway Fabric in the cluster.

- Create a self-signed cert-manager Issuer and a Certificate covering app.home.local and app.contact.local.

- Define the necessary GatewayClass, Gateway, and HTTPRoute resources to replicate the routing behavior of the original Ingress.

Ensure the following routing:

|Host               | Path      | Backend Service |
|-------------------|-----------|-----------------|
|app.home.local     |  /	    | home-svc        |
|app.home.local	    |  /sales   | sales-svc       |
|app.contact.local  |  /contact | contact-svc     |
|app.contact.local  |  /about   | about-us-svc    |

- TLS must be enabled for both hosts using the cert-manager-generated secret.

- Validate that all applications are accessible via their respective hostnames and paths, using curl or a browser.


You can use the documentantion:

https://gateway-api.sigs.k8s.io/guides/migrating-from-ingress/