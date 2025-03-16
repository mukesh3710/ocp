## Control Pod Scheduling


OpenShift provides multiple ways to control where and how pods are scheduled across cluster nodes. This is critical for performance, resource utilization, HA, compliance, and workload segregation.

Use Cases for Controlling Pod Scheduling
- Ensure Pods Run on Specific Nodes (e.g., GPU nodes, storage-heavy nodes, specific hardware).
- Avoid Running Certain Pods Together (e.g., HA deployments, DB master-slave separation).
- Load Balancing Across Nodes (e.g., distribute workloads across all available nodes).
- Ensure Pods Run Close to Each Other (e.g., microservices that communicate frequently).
- Prevent Pods from Running on the Same Node (e.g., redundancy for failover).

---
Methods to Control Scheduling in OpenShift
---

1️⃣ Assign Pod to a Specific Node: Using nodeName (direct assignment)
- Hard binding – Ensures a pod runs only on a specified node.
- Use Case: Running a pod on a dedicated node for licensing compliance or performance needs.
- Limitation: No fallback if the node is down.

---

2️⃣ Using nodeSelector (basic key-value matching)
- Ensures pods run on nodes matching specific labels.
- Example (assign pod to nodes labeled as gpu=true)
- Use Case: Assigning AI/ML workloads to GPU-enabled nodes.

---

3️⃣ Node Affinity (Advanced Scheduling)
- More powerful than nodeSelector.
- Supports soft (preferred) and hard (required) rules.
- Example (Prefer nodes with SSD storage)
- Use Case: Run database workloads only on SSD-backed nodes.

---

4️⃣ Pod Affinity & Anti-Affinity
- Pod Affinity: Ensures pods are scheduled together (e.g., microservices in the same rack for low latency).
- Pod Anti-Affinity: Ensures pods do not run together (e.g., HA deployments).
- Example (Pod Affinity - same zone for low latency)
- Use Case: Run frontend and backend together in the same availability zone.
- Use Case: Ensure Redis replicas run on different nodes for high availability.

---

5️⃣ Taints and Tolerations
- Taints prevent scheduling unless a pod has a matching toleration.
- Used to isolate workloads.
- Taint a node to allow only critical workloads
- Allow a pod to tolerate this taint
- Use Case: Ensure only system-critical workloads run on control-plane nodes.

---
---
## High-Level Workflow for Pod Scheduling in OpenShift

The below workflow ensures efficient pod scheduling in a 3 Master, 10 Worker OpenShift cluster by applying node labels, taints, tolerations, affinity, and anti-affinity rules.

---

1️⃣ Node Labeling & Tainting
- Master Nodes: Run only short-lived workloads (max 10 pods per node).
- SSD-Backed Nodes: Dedicated for databases.
- GPU Nodes: Reserved for AI/ML workloads.
- Availability Zones: Ensure microservices & Redis replicas are spread correctly.
```sh
oc label node <node-name> key=value
oc adm taint nodes <node-name> key=value:NoSchedule
```

---

2️⃣ Scheduling Rules
- Short-lived workloads (e.g., batch jobs) → Only on master nodes, max 10 pods per node.
- Databases → Must run only on SSD-backed nodes (hard rule).
- AI/ML Workloads → Must run only on GPU-enabled nodes (hard rule).
- Redis Replicas → Each replica runs on a different node (anti-affinity).
- Microservices → Co-located in the same availability zone for low latency (affinity).

---

3️⃣ Validation & Monitoring
Check node labels # oc get nodes --show-labels
Verify pod placement # oc get pods -o wide
Inspect scheduling decisions # oc get pods -o wide
