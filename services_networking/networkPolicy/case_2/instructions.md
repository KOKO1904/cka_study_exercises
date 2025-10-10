# Allow Intra-Namespace Traffic While Denying External Access

### Your team is deploying an application in Kubernetes that has several pods distributed across two namespaces: dev and test.

### In the dev namespace you have:

- alpine-client (client pod)

- apache-server (web server pod)

### In the test namespace you have:

- busybox (test pod)

Tasks:

- Write a NetworkPolicy that denies all ingress traffic from pods in other namespaces to any pod in the dev namespace, while allowing all communication between pods within the same namespace.

- Test isolation from other namespaces
  - Try sending traffic from busybox (in the test namespace) to apache-server and confirm that it is blocked.
