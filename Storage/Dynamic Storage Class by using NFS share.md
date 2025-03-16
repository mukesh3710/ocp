##  Dynamic Storage Class by using NFS share

### Configure NFS  
- Create a NFS share
- Export the share
---

### Deployment
- Create deployment.yaml
- Update the namespace
- Update env variable - PROVISIONER_name, NFS_SERVER, PATH, VOLUME
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: nfs
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-storage # Update here
            - name: NFS_SERVER
              value: 192.168.22.1 # Update here
            - name: NFS_PATH
              value: /shares/nfs-sc # Update here
      volumes:
        - name: nfs-client-root # update here 
          nfs:
            server: 192.168.22.1 # update here
            path: /shares/nfs-sc # update here 
```
---

### RBAC
- Create rbac.yaml
- Update the namespace 
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: nfs
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: nfs
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: nfs
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: nfs
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: nfs
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```
---

### Class
- Create class.yaml
- Update the provisioner name
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: nfs-storage # Update here 
parameters:
  archiveOnDelete: "false"
```
- To make class default add annotations
- oc edit sc nfs-client
```yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

---
###
- Apply all 3 playbooks 
- Deployment will fail as nfs volume is not allowed to be used, we need to assing appropriate secruity context
```yaml
oc create role use-scc-hostmount-anyuid --verb=use --resource=scc --resource-name=hostmount-anyuid -n nfs
oc get roles -n nfs

# Find the user which need to assign this role from deployment in serviceAccountName yaml (nfs-client-provisioner)
oc adm policy add-role-to-user use-scc-hostmount-anyuid -z nfs-client-provisioner --role-namespace nfs -n nfs
```

---
### Test Storage class
- Create a PVC
- Create a POD
- Validate with "oc exec -it ubuntu-test-nfs sh"

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-nfs-provisioner
  namespace: default
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ubuntu
  name: ubuntu-test-nfs
  namespace: default
spec:
  containers:
  - image: ubuntu
    name: ubuntu
    resources: {}
    command: ["sleep", "3600"]
    volumeMounts:
      - mountPath: /nfs
        name: nfs-vol
  volumes:
    - name: nfs-vol
      persistentVolumeClaim:
        claimName: test-nfs-provisioner
```

