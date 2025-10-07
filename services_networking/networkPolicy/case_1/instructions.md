Exercise 1:

Create a NetworkPolicy that denies all ingress traffic to pods labeled app=backend in the prod namespace, except from pods labeled app=frontend in the prod namespace.

Verify:

1. Get the caddy server ip. Create a temp busybox in the namespace prod name busybox to wget the caddy server ip.
2. Shell into the curl client and curl the caddy server ip.
