Exercise 5:

A team of engineers has developed and deployed a sample application into the mercury namespace. The system includes:

A frontend pod (client) that periodically:
    Requests data from a backend via http://server-service:5678
    Sends logs to logger-service:9880

A backend pod (server) that exposes an HTTP server on port 5678

A logger pod (logger) running Fluentd to collect logs via HTTP on port 9880

Several NetworkPolicy objects have already been created in the namespace to restrict ingress and egress traffic between the pods.

Currently, the client pod has all egress traffic blocked by the policy egress-blocker.

There is a template YAML file that the engineers wrote to allow traffic from the client pod to the server pod and logger pod. After applying the policy, the client pod was able to connect
to the server pod and logger pod only by Pod IP, but not by the server-service and logger-service DNS names, due to DNS resolution being blocked by the current egress restrictions.
Your Task

Fix the DNS resolution issue so that the client pod can reach the backend using the server-service name (ClusterIP).

# Important:
# Do not modify or delete any existing NetworkPolicy. You may only create new NetworkPolicy objects to solve the problem.

