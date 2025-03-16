## Pod Scheduling Using Limits & Quotas in OpenShift

In OpenShift, resource requests and limits ensure fair resource allocation, while resource quotas and limit ranges enforce restrictions at the namespace level to prevent resource exhaustion.

1️⃣ Resource Requests & Limits (Per Pod/Container Level)
- Requests: The minimum guaranteed CPU/memory for a pod.
- Limits: The maximum CPU/memory a pod can use.

Example Use Cases:
- Prevent resource starvation → Set requests so critical apps always get minimum CPU/memory.
- Avoid noisy neighbors → Set limits so one pod doesn’t consume all resources.
- Efficient workload scheduling → Scheduler places pods on nodes with enough requested resources.

Defining Requests & Limits in a Deployment:
```yaml
containers:
  - name: my-app
    image: my-app-image
    resources:
      requests:
        cpu: "500m"    # Minimum 0.5 CPU
        memory: "256Mi" # Minimum 256MB RAM
      limits:
        cpu: "1000m"   # Max 1 CPU
        memory: "512Mi" # Max 512MB RAM
```
---

2️⃣ Resource Quotas & Limit Ranges (Namespace Level)
- ResourceQuota: Restricts the total number of resources used in a namespace.
- LimitRange: Defines default limits for containers if not specified.

Example Use Cases:
- Prevent resource overuse per project → Set quotas per namespace.
- Enforce default limits → Ensure every container gets reasonable CPU/memory allocations.
- Optimize multi-tenant clusters → Prevent one team from consuming all cluster resources.

Defining Resource Quota for a Namespace:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team1
spec:
  hard:
    pods: "10"                  # Max 10 pods allowed
    requests.cpu: "4"           # Max 4 CPUs requested
    requests.memory: "8Gi"      # Max 8GB memory requested
    limits.cpu: "8"             # Max 8 CPUs allowed
    limits.memory: "16Gi"       # Max 16GB memory allowed
```

Defining LimitRange for Default Limits:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team1
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:
        cpu: "250m"
        memory: "128Mi"
```
---

3️⃣ Real-World Best Practices

💡 Multi-Tenant OpenShift Cluster:
- Each team has a namespace with quotas to prevent overuse.
- Default CPU/memory limits are set using LimitRange.
- Critical applications have higher requests & limits to guarantee availability.

💡 Resource Optimization for a Large Enterprise:
- Development namespace: Lower limits (cpu: 500m, memory: 256Mi).
- Production namespace: Higher guaranteed resources (cpu: 2, memory: 4Gi).
- Database namespace: Strict quota (requests.memory: 16Gi, limits.memory: 32Gi).

💡 Cost Control in Cloud Environments:
- Prevent unnecessary scaling by enforcing CPU/memory requests.
- Use auto-scaling with limits to dynamically adjust workloads.

---

4️⃣ How to Validate Pod Scheduling
```sh
oc get resourcequota -n <namespace> # Check applied quota in a namespace
oc get limitrange -n <namespace> # View applied limit ranges
oc describe pod <pod-name> # Verify pod scheduling & resource usage
```
---

5️⃣ Best Practices for Real-World OpenShift Quota & Limits
- Set quotas based on real usage patterns → Avoid under-utilization.
- Enforce strict limits in production → Prevent excessive resource consumption.
- Use soft & hard constraints wisely → Example: ML workloads can request more dynamically.
- Monitor & adjust regularly → Use Prometheus/Grafana to analyze quota violations.
- Combine with Horizontal Pod Autoscaler (HPA) → Scale within defined limits.

🚀 With these strategies, enterprises effectively manage OpenShift workloads, ensuring efficiency, fairness, and cost control!
