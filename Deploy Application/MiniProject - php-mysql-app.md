## Mini Project php-mysql-app 

Key Resources:
Pods: Smallest unit containing one or more containers.
Deployments: Ensure a specified number of pod replicas are running; ideal for stateless applications.
Projects: Enhanced namespaces with annotations and access control.
Services: Enable internal pod-to-pod communication.
Persistent Volume Claims (PVCs): Request persistent storage without knowledge of underlying infrastructure.
Secrets: Store sensitive information securely.

---
### Create a New Project
```yaml
oc new-project php-mysql-app
```
---
### Create a Secret for MySQL Credentials
Create a new mysql-deployment.yaml configuration file:
```yaml
apiVersion: v1
data:
  MYSQL_DATABASE: YXBwZGI=
  MYSQL_PASSWORD: YXBwcGFzc3dvcmQ=
  MYSQL_ROOT_PASSWORD: cm9vdHBhc3N3b3Jk
  MYSQL_USER: YXBwdXNlcg==
kind: Secret
metadata:
  name: mysql-secret
  namespace: php-mysql-app
type: Opaque
```
Apply the configuration:
```bash
oc apply -f mysql-deployment.yaml
```
Option 2: Create a secter with command line
```bash
oc create secret generic mysql-secret \
  --from-literal=MYSQL_ROOT_PASSWORD='rootpassword' \
  --from-literal=MYSQL_USER='appuser' \
  --from-literal=MYSQL_PASSWORD='apppassword' \
  --from-literal=MYSQL_DATABASE='appdb'
```
---
### Deploy MySQL Database:
Create a YAML file (mysql-deployment.yaml) for MySQL deployment.
- PersistentVolumeClaim (mysql-pvc):
  - Requests 5GB of shared storage (ReadWriteMany) using the nfs-client storage class.
- Deployment (mysql):
  - Creates 1 MySQL replicas.
  - Uses the mysql:8.0 Docker image.
  - Retrieves MySQL credentials from a mysql-secret secret.
  - Mounts the shared storage (mysql-pvc) to /var/lib/mysql for data persistence.
- Service (mysql):
  - Exposes the MySQL deployment on port 3306, allowing other applications to connect.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteMany  # Ensure RWX for multiple replicas
  resources:
    requests:
      storage: 5Gi  # Adjust size as needed
  storageClassName: nfs-client  # Use default storage class

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1  # Two MySQL replicas sharing the same PVC (not recommended for production)
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql  # MySQL Data Directory
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc  # Attach the PVC

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```
Verify MySQL Pod Status:
```yaml
oc get pods -l app=mysql # Check if the MySQL pod is running and ready
oc logs -l app=mysql # If the pod is in CrashLoopBackOff or Error
oc get svc mysql # Check MySQL Service

Connect to MySQL and Run Queries:
oc rsh deploy/mysql
mysql -u$MYSQL_USER -p$MYSQL_PASSWORD -h 127.0.0.1
SHOW DATABASES;
USE my_database;  -- Replace with your actual database
SHOW TABLES;
SELECT NOW();
```
---
### Create a ConfigMap for PHP App Configuration

Create a new php-config.yaml file
```yaml
apiVersion: v1
data:
  DB_HOST: mysql
  DB_NAME: appdb
  DB_USER: appuser
kind: ConfigMap
metadata:
  name: php-config
  namespace: php-mysql-app
```
Apply the configuration:
```yaml
oc apply -f php-config.yaml
```
Option2: Create configmap with command   
```yaml
oc create configmap php-config \
  --from-literal=DB_HOST='mysql' \
  --from-literal=DB_USER='appuser' \
  --from-literal=DB_NAME='appdb'
```
---
### Deploy PHP Web Application:
Create a YAML file (php-deployment.yaml) for PHP deployment.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: php-pvc
  namespace: php-mysql-app
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-client # Use the same StorageClass as MySQL

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-app
  namespace: php-mysql-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-app
  template:
    metadata:
      labels:
        app: php-app
    spec:
      containers:
      - name: php-app
        image: php:apache
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: php-config
              key: DB_HOST
        - name: DB_NAME
          valueFrom:
            configMapKeyRef:
              name: php-config
              key: DB_NAME
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        ports:
        - containerPort: 80
        volumeMounts:
        - name: php-code
          mountPath: /var/www/html
      volumes:
      - name: php-code
        persistentVolumeClaim:
          claimName: php-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: php-service
  namespace: php-mysql-app
spec:
  selector:
    app: php-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
Apply the configuration:
```yaml
oc apply -f php-deployment.yaml
```
---
### Expose PHP Application Using OpenShift Route:
```yaml
oc expose service php-service --name=php-route --hostname=php-app.custom.apps.lab.ocp.lan
```
---
Verify the Deployment:
```yaml
oc get pods -n php-mysql-app
oc get services -n php-mysql-app
oc get routes -n php-mysql-app
```
---
Check Apache Configuration 
- Ensure an Index File Exists
```yaml
oc rsh deploy/php-app
ls -l /var/www/html/index.*
echo "<?php phpinfo(); ?>" > /var/www/html/index.php
```
