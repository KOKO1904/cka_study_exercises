# Cross-Namespace Routing with Kubernetes Gateway API

## You are working in a Kubernetes cluster with multiple teams. Your goal is to set up routing such that:

- There is a shared Gateway in the infra-team namespace.

- Two teams, team-blog and team-shop, each have their own namespaces (blog-ns and shop-ns).

- The blog-ns namespace has two services: blog-svc and home-svc.

- The shop-ns namespace has one service: store-svc.

- The Gateway should only allow routes from namespaces labeled with gateway-access: "enabled".

- The host is app.example.com

- Ensure the following routing:

|Host               | Path      | Backend Service |           Team            |       Namespace      |  
|-------------------|-----------|-----------------|---------------------------|----------------------|
|app.example.com    |  /	      | home-svc        |  team-blog                |       blog-ns        |
|app.example.com    |  /blog    | blog-svc        |  team-blog                |       blog-ns        |
|app.example.com    |  /store   | store-svc       |  team-shop                |       shop-ns        |


Tasks:

### 1. Set up namespaces

- Create multiple namespaces for different teams.

- Label the namespaces appropriately to control access to shared gateways.

- Assign each deployment, service and configMap to its respective namespace.

### 2. Deploy a shared Gateway

- Configure a Gateway in a central namespace.

- Define listeners for the desired protocol and port.

- Set rules to restrict which namespaces can attach routes.

- Configure TLS certificates if needed.

### 3. Define HTTPRoutes

- For each team, create routes in their respective namespaces.

- Specify routing rules for different paths or traffic patterns.

- Optionally configure traffic splitting or weights between multiple backends.

### 4. Attach routes to the Gateway

- Ensure each route references the shared Gateway using parentRefs or similar.

- Verify that only authorized namespaces are able to attach routes.

### 5. Validate routing behavior

- Test that traffic is routed correctly according to the defined rules.

- Confirm isolation between teams and namespaces.

Note: In controller.md, you can find the api gateway crd link to install it in your cluster

The documentation:

https://gateway-api.sigs.k8s.io/guides/multiple-ns/
