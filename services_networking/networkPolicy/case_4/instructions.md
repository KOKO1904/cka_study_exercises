Excersice 4:

You are given a Kubernetes namespace called webapp that contains multiple workloads: frontend, backend, Redis, and a MySQL database. 
Each workload has different labels and is managed by Deployments or StatefulSets.

Several NetworkPolicies are applied in the namespace to control traffic between pods.

Your manager has noticed that the backend pod cannot connect to Redis. 
Your task is to investigate the resources, labels, and NetworkPolicies to identify why the connection 
is failing and fix the issue by adjusting the NetworkPolicy configuration. (Don't modify the existing network policies nor add new ones)

1. Inspect the Deployments, StatefulSets, and Services in the webapp namespace.

2. Review the labels assigned to each pod and workload.

3. Analyze the existing NetworkPolicies and understand what traffic is allowed or denied.

4. Identify why the backend cannot connect to Redis.

5. Verify that connectivity is restored.
