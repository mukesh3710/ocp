## Secrets and ConfigMaps

Secrets: Used to store sensitive data like passwords, API keys, and SSH keys. Secrets are base64-encoded and can be mounted as files or injected as environment variables.

Types of Secrets (Most Commonly Used)
Opaque (Default) – User-defined key-value pairs.
TLS Secret – Stores TLS certificates and keys.
Docker Registry Secret – Used for pulling images from private registries.
Basic Authentication Secret – Stores credentials like username/password.
SSH Key Secret – Stores SSH private/public keys.
Service Account Token Secret – Provides access to API resources.


ConfigMaps: Used to store non-sensitive configuration data such as application properties, environment settings, and other configuration files.

---

### Example of deploying an NGINX app on OpenShift/Kubernetes that:
- Uses a Secret for sensitive data
- Uses a ConfigMap for configuration
- Injects both into an NGINX Deployment as environment variables & volume mount 

---
#### Create the Secret (For Sensitive Data):
- We'll store a username and password for basic authentication.
- Create the Secret YAML (nginx-secret.yaml) and apply kubectl apply -f nginx-secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # Base64 encoded "username"
  password: cGFzc3dvcmQ=  # Base64 encoded "password"
```
---
#### Create the ConfigMap (For Non-Sensitive Config):
- We'll store an nginx.conf file and a message.
- Create the ConfigMap YAML (nginx-configmap.yaml) and apply kubectl apply -f nginx-configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html;
            auth_basic "Restricted Content";
            auth_basic_user_file /etc/nginx/.htpasswd;
        }
    }
  welcome-message: "Welcome to my NGINX app!"
```
---
Create the Deployment:
- NGINX_USERNAME comes from the Secret as an environment variable
- WELCOME_MESSAGE comes from the ConfigMap as an environment variable
- NGINX_PASSWORD is mounted as a file at /etc/secrets/password
- nginx.conf is mounted as a file at /etc/nginx/nginx.conf from the ConfigMap
- Use htpasswd to generate a password file inside the container
- Apply the Deployment - kubectl apply -f nginx-deployment.yaml
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
          env:
            - name: NGINX_USERNAME
              valueFrom:
                secretKeyRef:
                  name: nginx-secret
                  key: username
            - name: WELCOME_MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: nginx-config
                  key: welcome-message
          volumeMounts:
            - name: nginx-secret-volume
              mountPath: "/etc/secrets"
              readOnly: true
            - name: nginx-config-volume
              mountPath: "/etc/nginx/conf.d/default.conf"
              subPath: nginx.conf
      volumes:
        - name: nginx-secret-volume
          secret:
            secretName: nginx-secret
        - name: nginx-config-volume
          configMap:
            name: nginx-config
```
---
