## Edge & Passthrough Routes

Below are the complete steps to set up an NGINX application in OpenShift 4.16, configure Edge & Passthrough Routes, create a self-signed TLS certificate, and validate HTTPS access.

---

🔹 Step 1: Deploy NGINX in OpenShift
We'll deploy NGINX with a Service to expose it internally.

Create the Deployment & Service
Create a file nginx-deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx:latest
          ports:
            - containerPort: 80
            - containerPort: 443
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
  type: ClusterIP
```
---

🔹 Step 2: Create a Self-Signed TLS Certificate
Since we don't have a certificate, we'll generate a self-signed one.

Generate the Certificate & Key
This generates:
- tls.crt (certificate)
- tls.key (private key)
- Create an OpenShift Secret for TLS
```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=nginx.apps.lab.ocp.lan/O=MyOrg"

oc create secret tls nginx-tls --cert=tls.crt --key=tls.key
```
---
🔹 Step 3: Update NGINX to Use the TLS Certificate
We need to update NGINX's configuration to use HTTPS.

Create a ConfigMap for NGINX
Create a file nginx-config.yaml:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;
        server_name localhost;
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }

    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /etc/nginx/tls/tls.crt;
        ssl_certificate_key /etc/nginx/tls/tls.key;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
```
---

🔹 Step 4: Mount TLS Secret & ConfigMap in Deployment
Modify your deployment to use the TLS Secret & ConfigMap.

Create nginx-deployment-updated.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx:latest
          ports:
            - containerPort: 80
            - containerPort: 443
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
            - name: nginx-tls
              mountPath: /etc/nginx/tls
              readOnly: true
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config
        - name: nginx-tls
          secret:
            secretName: nginx-tls
```
---
🔹 Step 5: Create Edge and Passthrough Routes
1️⃣ Edge Route (TLS Termination at OpenShift Router)
- OpenShift terminates TLS, and traffic to NGINX is HTTP (port 80).
```sh
oc create route edge nginx-edge --service=nginx-service \
  --hostname=nginx-edge.apps.lab.ocp.lan
```

2️⃣ Passthrough Route (TLS Termination at Backend)
- TLS termination happens at NGINX (OpenShift forwards raw HTTPS traffic).
```sh
oc create route passthrough nginx-passthrough --service=nginx-service \
  --hostname=nginx-passthrough.apps.lab.ocp.lan
```
----

🔹 Step 6: Validate HTTPS Access
Check Created Routes
```sh
oc get routes
curl -k https://nginx-edge.apps.lab.ocp.lan # OpenShift terminates HTTPS and forwards HTTP to NGINX.
curl -k https://nginx-passthrough.apps.lab.ocp.lan # OpenShift forwards HTTPS traffic directly to NGINX (which serves it via self-signed cert).
```

---

✅ Summary
Route Type	TLS Termination	Backend Port	Command
Edge	OpenShift Router	HTTP (80)	oc create route edge
Passthrough	NGINX Backend	HTTPS (443)	oc create route passthrough
🎯 Next Steps
If using a trusted certificate, replace the self-signed cert (tls.crt, tls.key) with one from a CA.
If browser shows untrusted cert warnings, import tls.crt into trusted certificates.
