## SCC

Security Context Constraints (SCC) in OpenShift define a set of security rules that control how pods run, such as:
- Running as a specific user or group
- Allowing/disallowing privileged containers
- Controlling volume access
- Restricting network capabilities

SCCs help enforce security policies at the namespace or cluster level. They are similar to Pod Security Policies (PSP) in Kubernetes but are specific to OpenShift.

---

### Types of SCC in OpenShift
| SCC Name          | Description                                                                  | Use Case                                                                   |
|-------------------|------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| privilegedFull    | Full access, allows privileged containers                                  | Required for low-level system services like Storage, Network plugins, and Monitoring agents |
| restricted        | Enforces strong security policies                                          | Default for most workloads (blocks privileged access)                     |
| nonroot           | Requires pods to run as a non-root user                                     | Recommended for standard applications                                      |
| anyuid            | Allows pods to run as any user                                               | Needed for applications that require specific UIDs                         |
| hostaccess        | Allows host namespace and networking access                                  | Used for custom networking configurations                                  |
| hostmount-anyuid  | Grants full access to host file system                                       | Required for storage solutions (e.g., NFS, GlusterFS)                      |
| hostnetwork       | Allows pods to use the host network                                          | Used for Load Balancers, DNS, and other network services                   |
| node-exporter     | Used for node-exporter monitoring                                            | Required for Prometheus node-exporter                                     |

---

### How to Check SCC for a Deployment
```sh
oc get scc # List available SCCs
oc adm policy who-can use scc privileged # Check SCC assigned to a service account
oc get pod <pod-name> -o yaml | grep -i scc # Check SCC for a running pod
oc describe scc <scc-name> # Check permissions of an SCC
```
---
### Troubleshoot SCC issue
```yaml
First, check which service account (SA) is used by the pod:
oc get pod <pod> -o jsonpath="{.spec.serviceAccountName}"

Check the Current SCC Assigned to the Service Account:
oc get rolebinding -n <NAMESPACE> | grep scc

Check the SCC for the default SA:
oc adm policy who-can use scc restricted

Check Pod Security Context Requirements:
oc get pod <pod> -o yaml | grep -A10 securityContext

Assign the Least Privilege SCC for Nginx:
# To allow Nginx to write to /var/cache/nginx, you need to assign an SCC that provides write access while avoiding root privileges
# anyuid allows the container to run as its default user, rather than being forced into an arbitrary UID. Apply it
oc adm policy add-scc-to-user anyuid -z default -n <NAMESPACE>

Restart the Deployment:
oc rollout restart deployment/nginx-deployment -n <NAMESPACE>

Verify the SCC Applied to the Running Pod & Check again current SCC assgined:
oc get pod <pod> -o yaml | grep -i scc
oc get rolebinding -n <NAMESPACE> | grep scc

Alternative:
Modify Nginx Deployment for Security Compliance
Instead of changing SCC, modify the deployment to use a writable volume.
Mount an emptyDir volume to /var/cache/nginx so Nginx can write to it.
```
---

### Running a Privileged Container for CSI Storage

Scenario: An organization wants to deploy a CSI storage driver (e.g., NetApp Trident, Ceph RBD, or OpenEBS) on OpenShift. The container needs:
- Host networking access
- Privileged mode to interact with the kernel
- Root permissions to mount volumes
```yaml
Check if the SCC is needed & verify if privileged allows running as root, mounting host volumes, and using the host network:
oc describe scc privileged

Assign the SCC to the service account:
oc adm policy add-scc-to-user privileged -z trident-controller -n storage

Verify SCC is applied to the deployment:
oc get pod -n storage -o yaml | grep -i scc
```

Outcome:
- The storage driver runs successfully.
- The SCC ensures only the required service accounts get privileged access, maintaining security compliance.

---
