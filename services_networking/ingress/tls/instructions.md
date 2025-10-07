# NGINX Deployments with TLS-enabled Ingress

## Create two NGINX deployments, expose them as services, and configure an Ingress with TLS to route traffic to them.

Tasks:

**Deployments**:
- Deploy two separate NGINX deployments named alpha and beta.
  - Each deployment should run on port 80.
  - Each deployment should run one replica
  - Pods in deployment alpha should have the label app: alpha y same with beta: app:beta

**Services**:
- Expose Services, Create ClusterIP services for both deployments.
  - Each service should expose port 80.

**Ingress**
- Create Ingress, Name the Ingress: connection-ingress.
  - Install traefik controller by using:
    
    ```bash
    helm repo add traefik https://traefik.github.io/charts
    helm repo update
    kubectl create namespace traefik
    helm install traefik traefik/traefik --namespace traefik
    ```

  - Route requests to the corresponding services based on the path:
    - Requests to `/alpha` and `/beta` should be routed to the corresponding services..
    - Enable TLS for secure traffic by creating a TLS secret named `my-secret` and generating the corresponding certificates.
    - The host for testing: `example.local`.

Once done, test your setup by running:

curl -k https://example.local/alpha or curl -k https://example.local/beta


