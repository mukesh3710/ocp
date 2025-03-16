---

# Lab: Deploy Web Applications with OpenShift

This guide walks through deploying a web application and database using OpenShift, ensuring reproducibility, persistent storage, and external access.

## Objectives
- Use image streams to ensure deployment reproducibility.
- Configure applications with Kubernetes secrets.
- Provide persistent storage for the database.
- Expose applications externally.

## Prerequisites
- OpenShift API URL: `https://api.ocp4.example.com:6443`
- OpenShift Console URL: `https://console-openshift-console.apps.ocp4.example.com`
- Login credentials:
  - Developer user: `developer/developer`

## Steps

### 1. Setup and Create a Project
1. Log in to the OpenShift cluster:
   ```bash
   oc login -u developer -p developer https://api.ocp4.example.com:6443
   ```
2. Create a project:
   ```bash
   oc new-project review
   ```

### 2. Configure the Database Image Stream
1. Create an image stream for the database:
   ```bash
   oc create istag mysql8:1 \
     --from-image registry.ocp4.example.com:8443/rhel9/mysql-80:1-228
   ```
2. Enable image lookup for the `mysql8` image stream:
   ```bash
   oc set image-lookup mysql8
   ```

### 3. Create a Secret for Database Configuration
1. Create the `dbparams` secret:
   ```bash
   oc create secret generic dbparams \
     --from-literal user=operator1 \
     --from-literal password=redhat123 \
     --from-literal database=quotesdb
   ```

### 4. Deploy the Database
1. Create the `quotesdb` deployment:
   ```bash
   oc create deployment quotesdb --image mysql8:1 --replicas 0
   ```
2. Add an image trigger for automatic updates:
   ```bash
   oc set triggers deployment/quotesdb --from-image mysql8:1 --containers mysql8
   ```
3. Add environment variables from the `dbparams` secret:
   ```bash
   oc set env deployment/quotesdb --from secret/dbparams --prefix MYSQL_
   ```
4. Add a 2GiB persistent volume:
   ```bash
   oc set volumes deployment/quotesdb --add \
     --claim-class lvms-vg1 --claim-size 2Gi --mount-path /var/lib/mysql
   ```
5. Scale up the deployment:
   ```bash
   oc scale deployment/quotesdb --replicas 1
   ```

### 5. Expose the Database
1. Create a service for `quotesdb`:
   ```bash
   oc expose deployment quotesdb --port 3306
   ```

### 6. Deploy the Frontend Application
1. Create the `frontend` deployment:
   ```bash
   oc create deployment frontend \
     --image registry.ocp4.example.com:8443/redhattraining/famous-quotes:2-42 \
     --replicas 0
   ```
2. Add environment variables from the `dbparams` secret and set the database hostname:
   ```bash
   oc set env deployment/frontend --from secret/dbparams --prefix QUOTES_
   oc set env deployment/frontend QUOTES_HOSTNAME=quotesdb
   ```
3. Scale up the deployment:
   ```bash
   oc scale deployment/frontend --replicas 1
   ```

### 7. Expose the Frontend Application
1. Create a service for `frontend`:
   ```bash
   oc expose deployment frontend --port 8000
   ```
2. Create a route to expose the service externally:
   ```bash
   oc expose service frontend
   ```
3. Retrieve the application URL:
   ```bash
   oc get route
   ```
   Example output:
   ```
   NAME       HOST/PORT                               PATH   SERVICES   PORT    TERMINATION   WILDCARD
   frontend   frontend-review.apps.ocp4.example.com          frontend   8000    None          None
   ```
4. Test the application:
   ```bash
   curl http://frontend-review.apps.ocp4.example.com
   ```

## Summary
- You deployed a MySQL database and a frontend web application.
- Ensured reproducibility using image streams and secrets.
- Provided persistent storage for the database.
- Exposed the application to external clients.

For production readiness, configure probes, resource limits, and additional security policies.

---
---

# Troubleshooting and Scaling Applications in OpenShift

This guide provides a streamlined process for troubleshooting and scaling applications in OpenShift with real-world production examples.

## Objectives
- Troubleshoot malfunctioning workloads.
- Fix a failed MySQL pod.
- Scale applications manually.
- Configure health probes and resource limits.

### Key Specifications
- **API URL:** `https://api.ocp4.example.com:6443`
- **Web Console URL:** `https://console-openshift-console.apps.ocp4.example.com`
- **Login Credentials:**
  - Developer: `developer / developer`
  - Admin: `admin / redhatocp`

### Application Setup
The application includes:
- **Frontend Deployment:** Web application serving quotes.
- **QuotesDB Deployment:** MySQL database storing quotes.

## Steps to Troubleshoot and Fix the Application

### Identify and Remove Excessive CPU-Consuming Workloads
1. Log in to the OpenShift Web Console.
2. Navigate to **Observe > Dashboards > Kubernetes / Compute Resources / Cluster**.
3. Inspect the **CPU Usage** graph and identify the namespace consuming the most CPU.
4. Delete the deployment consuming excessive resources:
   ```bash
   oc delete deployment <deployment-name> -n <namespace>
   ```

### Fix the QuotesDB Deployment

#### 1. Review Logs and Set Missing Environment Variables
- Check the failing pod logs:
  ```bash
  oc logs <quotesdb-pod-name>
  ```
- Add the required environment variables:
  ```bash
  oc set env deployment/quotesdb \
    MYSQL_USER=operator1 \
    MYSQL_PASSWORD=redhat123 \
    MYSQL_DATABASE=quotes
  ```

#### 2. Update MySQL Container Image
- Update the container image:
  ```bash
  oc set image deployment/quotesdb \
    mysql-80=registry.ocp4.example.com:8443/rhel9/mysql-80:1-237
  ```
- Verify the image update:
  ```bash
  oc get deployment/quotesdb -o wide
  ```

#### 3. Add Health Probes to QuotesDB
- Add a readiness probe:
  ```bash
  oc set probe deployment/quotesdb \
    --readiness -- mysqladmin ping
  ```
- Add a liveness probe:
  ```bash
  oc set probe deployment/quotesdb \
    --liveness -- mysqladmin ping
  ```

#### 4. Define Resource Limits for QuotesDB
- Configure resource requests and limits:
  ```bash
  oc set resources deployment/quotesdb \
    --requests cpu=200m,memory=256Mi \
    --limits cpu=500m,memory=1Gi
  ```

### Configure Frontend Deployment

#### 1. Add Health Probes
- Add a readiness probe:
  ```bash
  oc set probe deployment/frontend --readiness \
    --get-url=http://:8000/status
  ```
- Add a liveness probe:
  ```bash
  oc set probe deployment/frontend --liveness \
    --get-url=http://:8000/env
  ```

#### 2. Define Resource Limits
- Configure resource requests and limits:
  ```bash
  oc set resources deployment/frontend \
    --requests cpu=200m,memory=256Mi \
    --limits cpu=500m,memory=512Mi
  ```

#### 3. Scale the Frontend Deployment
- Scale the deployment to three replicas:
  ```bash
  oc scale deployment/frontend --replicas=3
  ```

## Verification

### Test Application Availability
1. Retrieve the application URL:
   ```bash
   oc get route
   ```
2. Test the application:
   ```bash
   curl http://frontend-compreview-scale.apps.ocp4.example.com
   ```

Expected Output:
```html
<html>
<head>
    <title>Quotes</title>
</head>
<body>
    <h1>Quote List</h1>
    <ul>
        <li>1: When words fail, music speaks. - William Shakespeare</li>
    </ul>
</body>
</html>
```

### Ensure All Pods Are Running
Verify that all pods are running:
```bash
oc get pods
```

## Outcomes
You should now have a fully functional, production-ready application in OpenShift, with resource limits, health probes, and scaling configured.


