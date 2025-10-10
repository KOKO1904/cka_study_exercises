# Customize an Existing Nginx Deployment Using Kustomize

### You have the following existing Kubernetes resources in the setup folder, your task are:

- Create a ConfigMap named myconfig that contains application properties stored in a file called app.properties with the following content:

```bash
host=localhost
port=3306
```

- Use a patch to modify the existing Deployment so that:
  - The number of replicas is increased from 1 to 5.
  - The Nginx container uses the ConfigMap myconfig as an environment source.

- Define a kustomization.yaml that applies the ConfigMap and the patch to the existing Deployment and Service.

Note: Remember to not modify deployment.yaml or service.yaml directly.


