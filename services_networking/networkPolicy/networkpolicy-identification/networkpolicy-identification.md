# Network Policy Identification

### The goal of these exercises is not to fully solve each scenario. Instead, you must identify:
- The pod(labels) the policy applies to, including its namespace

- The type of policy (Ingress, Egress, or both)

- The target pod(labels) referenced in the podSelector

- Any namespaceSelector involved

- The ports defined

Example:

```bash
# Ingress
TargetPod:
Namespace:
PolicyType:
AllowedFromPodSelector:
AllowedFromNamespace:
Ports:

# Egress
TargetPod: 
Namespace: 
PolicyType: 
AllowedToPodSelector: 
AllowedToNamespace: 
Ports: 

```

### In the namespace production, create a NetworkPolicy that denies all ingress traffic to pods labeled app=payment-service except from pods labeled app=frontend within the same namespace. All other ingress traffic should be blocked.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: app=payment-service
Namespace: production
PolicyType:  Ingress
AllowedFromPodSelector: app=frontend
AllowedFromNamespace: production
Ports:
```
</p>
</details>


### Write a NetworkPolicy in the development namespace that allows pods labeled role=debugger to send egress traffic only to pods labeled role=api-server in the testing namespace on TCP port 8080 and 8443.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: role=debugger
Namespace: development
PolicyType: Egress
AllowedToPodSelector: role=api-server
AllowedToNamespace: testing
Ports: 8080, 8443
```

</p>
</details>


### Implement a NetworkPolicy in namespace analytics that permits ingress to pods labeled service=data-collector only from pods with label service=event-processor in the monitoring namespace.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: service=data-collector
Namespace: analytics
PolicyType: Ingress
AllowedFromPodSelector: service=event-processor
AllowedFromNamespace: monitoring
Ports:
```

</p>
</details>


### Configure a NetworkPolicy that blocks all egress traffic from any pod in the legacy namespace except for traffic to pods labeled env=production within the same namespace on ports 3306 and 5432.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: all pods
Namespace: legacy
PolicyType: Egress
AllowedToPodSelector: env=production
AllowedToNamespace: legacy
Ports: 3306, 5432
```

</p>
</details>


### In namespace core, allow inbound connections to pods labeled service=processor only from pods labeled service=ingestor in the same namespace.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: service=processor
Namespace: core
PolicyType: Ingress
AllowedFromPodSelector: service=ingestor
AllowedFromNamespace: core
Ports:
```
</p>
</details>

