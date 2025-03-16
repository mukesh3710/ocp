## Persistent Storage - NFS (Static)

### Static Provisioning

```yaml
# Admin Creates a Static PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-lun1
spec:
  storageClassName: static
  capacity:
    storage: 15Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /nfsshare 
    server: 192.168.22.1

# Developer Requests Storage via PVC:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  storageClassName: static
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi

# Developer deploy application and mount the PVC:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 8080
        volumeMounts: 
          - name: nginx-vol
            mountPath: /usr/share/nginx/html
      volumes:
        - name: nginx-vol
          persistentVolumeClaim:
            claimName: nginx-pvc
```
