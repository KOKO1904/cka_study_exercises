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

### In the namespace payments, ensure that only pods identified as role=billing-engine can communicate with pods tagged tier=processor. No other workloads should establish that connection within the namespace.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: tier=processor
Namespace: payments
PolicyType: Ingress
AllowedFromPodSelector: role=billing-engine
AllowedFromNamespace: payments (optional, can be empty)
Ports:
```

</p>
</details>

### Configure restrictions in analytics so that module=data-cruncher is able to request information from module=collector. At the same time, the collector group must not initiate any connections back to data-cruncher.
---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: module=data-cruncher
Namespace: analytics
PolicyType: Egress
AllowedToPodSelector: module=collector
AllowedToNamespace: analytics (optional, can be empty)
Ports:

TargetPod: module=collector
Namespace: analytics
PolicyType: Egress
AllowedToPodSelector: []
AllowedToNamespace: []
Ports:

```

</p>
</details>

### In research-env, workloads tagged agent=trainer should be cut off from the internet or any external endpoints. They should only exchange traffic with pods labeled agent=model-store within the same namespace.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: agent=trainer
Namespace: research-env
PolicyType: Egress
AllowedToPodSelector: agent=model-store
AllowedToNamespace: research-env
Ports:

TargetPod: agent=trainer
Namespace: research-env
PolicyType: Ingress
AllowedFromPodSelector: agent=model-store
AllowedFromNamespace: research-env
Ports:
```

</p>
</details>

### Namespace edge-services: traffic should only originate from pods labeled region=west-hub when they need to reach app=gateway-core. Any attempt from other labels or namespaces to contact gateway-core must be prevented.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: app=gateway-core
Namespace: edge-services
PolicyType: Ingress
AllowedFromPodSelector: region=west-hub
AllowedFromNamespace: edge-services (optional)
Ports:
```

</p>
</details>

### In internal-tools, the set labeled tier=frontend-suite may send requests to tier=report-engine, but under no circumstance should report-engine initiate connectivity toward other workloads.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: tier=report-engine
Namespace: internal-tools
PolicyType: Ingress
AllowedFromPodSelector: tier=frontend-suite
AllowedFromNamespace: tier=frontend-suite (optional)
Ports:

TargetPod: tier=report-engine
Namespace: internal-tools
PolicyType: Egress
AllowedToPodSelector: [] 
AllowedToNamespace: []
Ports: 
```

</p>
</details>

### Within the iot-mgmt namespace, connections to unit=sensor-hub must come strictly from pods labeled unit=relay-station. Additionally, the sensor-hub pods should not establish connectivity toward other groups.

---

<details>
<summary>Show commands / answers</summary>
<p>

```bash
TargetPod: unit=sensor-hub
Namespace: iot-mgmt
PolicyType: Ingress
AllowedFromPodSelector: unit=relay-station
AllowedFromNamespace: iot-mgmt (optional)
Ports:

TargetPod: unit=sensor-hub
Namespace: iot-mgmt
PolicyType:  Egress
AllowedToPodSelector: [] 
AllowedToNamespace: []
Ports: 
```

</p>
</details>

