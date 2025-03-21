# Chapter 8: Application Security

## Control Application Permissions with Security Context Constraints (SCCs)

Security Context Constraints (SCCs) in OpenShift control pod access to the host environment, including:
- Privileged containers
- Extra container capabilities
- Host directories as volumes
- SELinux context
- User ID restrictions

### Default SCCs in OpenShift:
- **restricted-v2**: Default for most pods, providing limited access.
- **Others**: `anyuid`, `hostaccess`, `privileged`, etc. (Use `oc get scc` for a full list).

### Using SCCs:
- View SCC details: `oc describe scc <scc_name>`
- Identify SCC used by a pod: `oc describe pod <pod_name> | grep scc`

### Addressing Challenges with restricted-v2:
- Some container images require relaxed SCCs (e.g., specific user ID or privileged ports).
- To find compatible SCCs:
  ```
  oc get deployment deployment-name -o yaml | oc adm policy scc-subject-review -f -
  ```

### Relaxing SCCs with Service Accounts:
1. **Create a Service Account:**
   ```
   oc create serviceaccount <service_account_name> -n <namespace>
   ```
2. **Associate SCC with Service Account (cluster-admin only):**
   ```
   oc adm policy add-scc-to-user <scc_name> -z <service_account_name> -n <namespace>
   ```
3. **Assign Service Account to Deployment:**
   ```
   oc set serviceaccount deployment/<deployment_name> <service_account_name>
   ```

**Important Notes:**
- Relaxing SCCs reduces cluster security—use cautiously.
- Only cluster admins can manage SCC assignments.

### Privileged Containers:
- Require elevated access beyond standard containers.
- Use SCCs and service accounts with privileged access carefully.

### Best Practices:
- Review SCC documentation before making changes.
- Use service accounts to securely manage pod permissions.

---

## Allow Application Access to Kubernetes APIs

### Protecting Your Cluster:
Kubernetes APIs allow modifications to cluster state. Grant access cautiously using RBAC (Role-Based Access Control).

### Application Service Accounts:
- Represent the application's identity in a pod.
- Used to grant API access for tasks such as monitoring, controllers, or operators.

### Granting Permissions:
1. **Define Roles or Cluster Roles:** Specify the required access.
2. **Bind Role to Service Account:** Use roles for namespace-specific access or cluster roles for global access.
3. **Specify Service Account in Pod Definition.**

**Example:**
```yaml
serviceAccountName: <service_account_name>
```

### Token Management:
- For OpenShift 4.11+, generate tokens manually using the TokenRequest API.
- Mount the token as a pod volume for application access.

### Scoping Access:
- Use roles for local namespace access.
- Use cluster roles for access across namespaces.
- Grant access only to the resources and actions required by the application.

---

## Cluster and Node Maintenance with Kubernetes Cron Jobs

### Use Cases:
- Automate tasks for cluster and application management.
  - **Cluster administrators**: Maintenance tasks.
  - **Application users**: Lifecycle tasks (e.g., backups).

### Kubernetes Batch API Resources:
- **Jobs**: Run tasks once.
- **Cron Jobs**: Schedule periodic tasks.
  - Create using: `oc create cronjob`
  - Define schedules using Linux cron syntax.

### Automating Application Maintenance:
- Example: Weekly database backups for a WordPress application.
  ```yaml
  schedule: "0 2 * * 1"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: <backup_image>
            args: ["backup"]
          restartPolicy: OnFailure
  ```

### Automating Cluster Maintenance:
- Example: Prune unused images on cluster nodes.
  - Store maintenance scripts in ConfigMaps.
  - Mount ConfigMaps as volumes in the pod.

### Security:
- Use service accounts with appropriate privileges for privileged tasks.

### Best Practices:
- Test cron jobs thoroughly before deployment.
- Use monitoring tools to track execution status.

---

# Summary
- **Security Context Constraints (SCCs):** Control pod access to the host environment. Assign SCCs to application service accounts cautiously.
- **Kubernetes API Access:** Use roles and service accounts for secure API access. Grant minimal required permissions.
- **Cron Jobs:** Automate cluster and application maintenance tasks using Kubernetes cron jobs.

