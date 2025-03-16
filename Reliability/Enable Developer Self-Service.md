# Enable Developer Self-Service in OpenShift

Managing resource usage and enabling secure, self-service operations for developers is critical in OpenShift. This guide simplifies configuring quotas, limit ranges, and project templates for real-world production usage.

---

## Managing Resource Usage with Quotas

### Challenges of Unmanaged Workloads
- Cluster resources (CPU, RAM, storage) are finite.
- Uncontrolled workloads can impact performance or lead to unexpected costs.

### Resource Controls in Kubernetes
1. **Resource Limits**: Set an upper limit on resources (e.g., CPU, memory) a workload can consume.
2. **Resource Requests**: Specify the minimum required resources to ensure workload stability.

### Project Quotas (ResourceQuota Objects)
- Limit resource usage within a namespace.
- Define limits using the `hard` key in the `spec` section.

#### Example Use Cases:
- **Compute Quotas**: Restrict CPU and memory usage (e.g., `limits.cpu`, `requests.memory`).
- **Object Count Quotas**: Limit the number of deployments or pods in a namespace.

#### How to Apply:
1. Use the OpenShift web console (Administration -> ResourceQuotas) or the `oc` command.
2. View current usage with `oc get quota` and `oc describe quota`.

### Cluster Resource Quotas (ClusterResourceQuota Objects)
- Apply quotas across multiple projects based on label selectors.
- Combine total usage across namespaces for enforcement.

#### How to Apply:
1. Define quotas in the `quota` key.
2. Use selectors to determine applicable namespaces.
3. Monitor with `oc get clusterresourcequota`.

---

## Limiting Resource Usage with Limit Ranges

### Problem
Uncontrolled workloads can exceed namespace quotas and disrupt other applications.

### Solution: Limit Ranges
Define default and maximum resource limits for pods to:
1. Prevent overconsumption.
2. Ensure consistent allocation.
3. Simplify workload configuration.

#### Key Limit Types:
1. **Default Limits**: Set default requests and limits for workloads.
2. **Maximum Limits**: Cap the maximum resources for any workload.
3. **Minimum Limits**: Ensure all workloads meet a minimum requirement.
4. **Limit-to-Request Ratios**: Control the ratio of requested to maximum resources.

#### How to Apply:
1. Define a `LimitRange` resource in the namespace.
2. Specify limits for CPU, memory, etc.

#### Example:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: example-limit-range
spec:
  limits:
  - default:
      cpu: "1"
      memory: "512Mi"
    defaultRequest:
      cpu: "0.5"
      memory: "256Mi"
    type: Container
```

---

## Project Templates and Self-Provisioning

### Project Templates
- Enhance security and simplify namespace creation.
- Include initial resources such as roles, quotas, limit ranges, and network policies.

#### How to Create:
1. Use `oc adm create-bootstrap-project-template` to generate a base template.
2. Modify to include desired configurations (e.g., resource quotas, network policies).
3. Add the template to the `openshift-config` namespace.
4. Update the `projects.config.openshift.io/cluster` resource to use the template.

#### Benefits:
- Streamlines project creation with pre-configured settings.
- Improves security and standardization.

### Managing Self-Provisioning Permissions
- **Default Setting**: All authenticated users have the `self-provisioner` role.
- Use `oc describe clusterrolebinding self-provisioners` to view permissions.
- Modify or restrict permissions by updating cluster role bindings.

#### Best Practices:
1. Limit self-provisioning permissions to authorized users only.
2. Combine with cluster quotas to manage total resource allocation across projects.

---

## Best Practices for Production
1. Use project quotas for granular namespace-level resource management.
2. Apply cluster resource quotas for multi-project control.
3. Define well-aligned limit ranges to prevent resource contention.
4. Customize project templates to include essential resources and policies.
5. Regularly review and manage self-provisioning permissions.

By implementing these strategies, you can ensure resource efficiency, security, and developer autonomy in your OpenShift cluster.
