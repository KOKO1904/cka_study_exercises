# Customize an Existing Nginx Deployment with Volumes and Replicas

### You have the following existing Kubernetes resources in the setup folder:

- deployment.yaml → an Nginx Deployment with 1 replica.
- service.yaml → a ClusterIP Service exposing the Nginx Deployment on port 80.ç

Your tasks are:

- Scale the Deployment:
  - Increase the number of Nginx replicas from 1 to 3 using Kustomize.

- Add a volume to the Nginx container:
  - Create an emptyDir volume named myvolume.
  - Mount it inside the container at /usr/share/nginx/html.

- Apply consistent labels:
  - Add common labels app: nginx and env: dev to all resources.

- Update the container image:
  - Change the Nginx image tag to 1.23.

- Use a patch file:
  - Implement the volume configuration and mounts via a patch.yaml.
 
Use replacements if necessary:
- Ensure the Service name matches the Deployment name dynamically using Kustomize replacements.

Note: Do not edit the original deployment.yaml or service.yaml.
