# Hybrid Service and Traffic Migration

### Your company has a Kubernetes cluster with 3 nodes. You’re migrating microservices from a legacy monolithic system. Your goal is to maintain availability while gradually shifting traffic from the old backend (outside the cluster) to the new backend (inside the cluster).

- Configure an internal ClusterIP service for the new backend.
- Create a ClusterIP Service and corresponding Endpoints manifest that points to the IP of the legacy backend outside the cluster. Do not use a selector, as the backend is external.
- Implement a ClusterIP hybrid service, manually specifying multiple endpoints:
  - One endpoint for the external backend (backend-v1)
  - Two endpoints for the in-cluster backend Pods (backend-v2)
- Create a LoadBalancer service with a custom nodePort for the frontend deployment. The frontend should reach the hybrid backend via http://backend-hybrid:8080.
- Validate that traffic flows correctly through all layers.
