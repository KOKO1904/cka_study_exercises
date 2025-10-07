Exercise 3:

Create a NetworkPolicy for pods labeled app=metrics in namespace jupiter that denies
ingress traffic from pods in the same namespace but allows ingress from pods in all 
other namespaces.

Verify:
1. Open a shell into the nginx pod in the saturn namespace and perform a curl request to the metrics pod.
2. Launch a temporary busybox pod in the jupiter namespace and attempt to connect to the metrics pod.
