# OpenShift Updates: Real-World Production Usage

## Cluster Update Process in OpenShift Container Platform 4

### Key Concepts

1. **Red Hat Enterprise Linux CoreOS (RHCOS):** Base operating system for OCP 4 clusters.
2. **Update Channels:** Define update paths with varying stability and release frequency:
   - **candidate-4.x:** Latest features; for testing only, not supported.
   - **fast-4.x:** Early updates; suitable for development and QA.
   - **stable-4.x:** Well-tested updates; recommended for production.
   - **eus-4.x:** Extended Update Support for longer maintenance cycles.
3. **Cluster Version Operator (CVO):** Manages cluster updates and coordinates operator updates.
4. **Operator Lifecycle Manager (OLM):** Handles updates for operators within the cluster.
5. **Machine Config Operator:** Updates machine configurations and manages rolling upgrades.

### Update Process Overview

1. **Check Update Availability:**
   - **Web Console:** Navigate to "Administration" > "Cluster Settings" > "Available Updates."
   - **CLI:** Run `oc adm upgrade`.

2. **Review Update Information:**
   - Check available channels and versions.
   - Choose the desired version and channel.

3. **Prepare for Update:**
   - Pause machine health checks to prevent unnecessary reboots.
   - Update necessary operators via OLM.

4. **Initiate Update:**
   - **Web Console:** Select the desired version and initiate the update.
   - **CLI:** Use `oc adm upgrade --to-latest` (latest version) or `oc adm upgrade --to=VERSION` (specific version).

5. **Monitor Progress:**
   - Check CVO and operator statuses.
   - Review cluster version history: `oc describe clusterversion`.

6. **Verify Completion:**
   - Confirm updated version with `oc get clusterversion`.

### Notes:
- **No Rollback:** Cluster version rollback is not supported.
- **Disconnected Clusters:** Use an alternate update process.
- **Telemetry Data:** Red Hat uses anonymized telemetry for update recommendations.
- **Machine Reboots:** Nodes may reboot during updates.

---

## Detect Deprecated Kubernetes API Usage

### Understanding API Deprecation

- **API Lifecycle:** Categorized into alpha, beta, and stable.
  - Beta APIs: Deprecated 3 releases after introduction; removed 3 releases later.
  - Stable APIs: Deprecated but remain within the same major Kubernetes version.

### Identifying Deprecated APIs

1. **List Resources:**
   - `oc api-resources | grep <resource>`

2. **Check Deprecation Warnings:**
   - Example: `oc create -f cronjob-beta.yaml` (generates warnings for beta APIs).

3. **Find Deprecated Usage:**
   - Use `oc get apirequestcounts` to view API usage.
   - Check for deprecated API alerts:
     - `APIRemovedInNextReleaseInUse`
     - `APIRemovedInNextEUSReleaseInUse`

4. **Migration:**
   - Migrate workloads using deprecated APIs before upgrading to avoid disruptions.

---

## Update Operators with OLM

### Operator Update Process

1. **Choose Update Channel:** Select between stable, preview, or other available channels based on stability requirements.
2. **Set Approval Mode:**
   - **Automatic:** Updates applied automatically.
   - **Manual:** Updates require admin approval.
3. **Update Operators:**
   - **Web Console:** Review available updates and approve as needed.
   - **CLI:** Use `oc` commands to manage updates.

### Important Considerations

- Ensure operator compatibility with your OpenShift version before updating.
- Operators may leave residual resources after uninstallation; refer to documentation for cleanup.
- Test updates in staging environments before applying to production.

---

## Summary

1. **Cluster Updates:** Utilize Over-the-Air (OTA) updates for efficient and reliable cluster maintenance. Choose appropriate channels (candidate, fast, stable, or eus) based on environment needs.
2. **Deprecated APIs:** Monitor and migrate workloads to avoid disruptions caused by deprecated Kubernetes APIs.
3. **Operator Management:** Use OLM to manage operator updates, balancing stability with feature adoption. Set approval modes based on operational preferences.

By following these guidelines, you can ensure a secure, stable, and efficient OpenShift environment tailored to production needs.
