# Restrict Backend Pods to Frontend-Only Ingress

### Create a NetworkPolicy that denies all ingress traffic to pods labeled app=backend in the prod namespace, except from pods labeled app=frontend in the prod namespace.

Verify:

- Get the caddy server ip. Create a temp busybox in the namespace prod name busybox to wget the caddy server ip.
- Shell into the curl client and curl the caddy server ip.
