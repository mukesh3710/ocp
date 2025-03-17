## Persistent Storage in Kubernetes and OpenShift

Kubernetes and OpenShift provide Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and Storage Classes (SCs) to handle storage needs. This system allows separation of storage provisioning (Admin role) from storage consumption (Developer role).

---
### Key Storage Concepts in Kubernetes/OpenShift

| Component | Description | Role in Kubernetes/OpenShift |
|---|---|---|
| Persistent Volume (PV) | A physical storage resource provisioned in the cluster (e.g., NFS, iSCSI, AWS EBS). | Cluster Admin provisions this manually or dynamically via Storage Class. |
| Persistent Volume Claim (PVC) | A request by a pod for a specific amount of storage. | Developers create this to request storage. Kubernetes finds a matching PV. |
| Storage Class (SC) | Defines how storage should be provisioned dynamically. | Admins define different storage options (e.g., SSD, HDD, ReadWriteMany). |
---

### How Persistent Storage Works in Kubernetes & OpenShift

Cluster Administrator (Admin):
- Creates a PV (manually) or defines a Storage Class (SC) for dynamic provisioning.
- Ensures storage is available and correctly mounted.

Developer:
- Requests storage by creating a PVC.
- Kubernetes/OpenShift binds the PVC to an available PV or dynamically provisions storage via SC.
- Uses the PVC in their pod to access storage.
---

### Static vs. Dynamic Provisioning

Static Provisioning (Admin Pre-creates PVs):
- Admin manually creates PV before developers request storage.
- PVCs bind to available PVs if they match the request.
- If no PV matches, the PVC remains Pending.

Dynamic Provisioning (Using Storage Classes):
- Admin defines a Storage Class (SC).
- Developer doesn't need to know about PVs—storage is created automatically.
---

### Persistent Storage Lifecycle

| Step | Admin Actions | Developer Actions |
|---|---|---|
| Provision Storage | Create PV or SC | Create PVC |
| Bind Storage | Kubernetes matches PV/PVC | PVC gets bound to PV |
| Use Storage | Monitor storage usage | Mount PVC in pods |
| Cleanup | Delete PVs manually (if Retain) | Delete PVC when not needed |

---

### ACCESS MODES   
Access modes define how a Persistent Volume (PV) can be mounted by a pod. The available access modes are:

ReadWriteOnce (RWO):
- The volume can be mounted as read-write by a single node.
- Used in block storage solutions like AWS EBS, Azure Disks, or OpenStack Cinder.
- If multiple pods run on the same node, they can share the volume.

ReadOnlyMany (ROX):
- The volume can be mounted as read-only by multiple nodes.
- Useful for sharing static content across multiple nodes.

ReadWriteMany (RWX):
- The volume can be mounted as read-write by multiple nodes.
- Used in shared storage solutions like NFS, CephFS, and GlusterFS.

ReadWriteOncePod (RWOP) - (Kubernetes 1.22+):
- The volume can be mounted as read-write by a single pod in a cluster.
- Ensures exclusive access to a pod (e.g., for databases requiring strict access control).

---

### RECLAIM POLICY
The reclaim policy determines what happens to a Persistent Volume (PV) after it is released (i.e., when a Persistent Volume Claim (PVC) is deleted).

Retain:
- The PV and its data are preserved even after the PVC is deleted.
- Requires manual intervention to delete or reuse the volume.
- Useful when data persistence is critical (e.g., databases).

Delete:
- The PV and its underlying storage are automatically deleted when the PVC is deleted.
- Common in cloud-provisioned storage like AWS EBS, Azure Disks, and Google Persistent Disks.

Recycle (Deprecated):
- The PV is scrubbed (basic deletion like rm -rf /data) and made available for reuse.
- Deprecated in Kubernetes 1.11+ in favor of dynamic provisioning.
---
