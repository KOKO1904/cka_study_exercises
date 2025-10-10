# Overlays introduction - Dev Overlay with Extra Labels

### You have a base Deployment webapp and Service. The base has 2 replicas and image nginx:stable

Tasks for the overlay dev:

- Add a label env=dev to all resources.
- Increase replicas to 3.
- Change the image tag to nginx:dev.
- Apply these changes using an overlay without touching the base files.

