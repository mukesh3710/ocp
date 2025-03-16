## Declarative vs. Imperative Approach in Kubernetes/OpenShift

### Imperative Approach:
- Directly issues commands to the cluster to create, update, or delete resources.
- Quick and easy for one-time tasks.
- Hard to version control and maintain.
```yaml
kubectl create deployment my-app --image=nginx

Imperative Commands to Generate Base YAML:
kubectl run my-pod --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment my-app --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl expose deployment my-app --port=80 --target-port=80 --type=ClusterIP --dry-run=client -o yaml > service.yaml
```

---
### Declarative Approach:
- Defines the desired state of resources using YAML or JSON files.
- Managed using kubectl apply to create or update resources.
- Version control-friendly, reusable, and more maintainable.
- A Kubernetes YAML configuration file typically consists of 5 basic parts:

---
### apiVersion
- Defines the version of the Kubernetes API to use.
- Determines the schema and behavior of the object.
```yaml
oc api-resources| kubectl api-resources
```

---
### kind
- Specifies the type of resource (e.g., Pod, Deployment, Service).
```yaml
kubectl explain pod
```

---
### metadata
- Identifies the resource with a name, namespace, labels, and annotations.
- name: Unique name for the resource within the namespace.
- namespace: (Optional) Namespace where the resource is created.
- labels: Key-value pairs for organization, selection, and grouping.
- annotations: Key-value pairs for non-identifying information.

---
### spec
- Describes the desired state of the resource.
- Varies depending on the resource type.
- spec defines the desired state of the resource.
- Common Fields:
  - replicas: Number of Pod replicas for a Deployment.
  - selector: Matches Pods using labels.
  - template: Defines the Pod specification.
  - containers: Defines the container image, ports, and environment variables.
  - annotations: Used for custom configurations or metadata.

---
### status
- Describes the current state of the resource.
- Managed by Kubernetes (not specified in YAML).

---
### Example YAML file
- Here’s a complete Kubernetes/OpenShift YAML file that includes a Pod, Deployment, and Service, with comments explaining each section:
```yaml
# Pod Definition
apiVersion: v1  # API version for Kubernetes resources
kind: Pod  # Defines the type of resource (Pod)
metadata:
  name: my-pod  # Name of the Pod
  labels:
    app: my-app  # Labels to categorize the Pod
spec:
  containers:
    - name: nginx-container  # Container name
      image: nginx:latest  # Image to use
      ports:
        - containerPort: 80  # Expose port 80 inside the container
---
# Deployment Definition
apiVersion: apps/v1  # API version for Deployments
kind: Deployment  # Defines the type of resource (Deployment)
metadata:
  name: my-deployment  # Name of the Deployment
  labels:
    app: my-app  # Labels to categorize the Deployment
spec:
  replicas: 3  # Number of desired Pod replicas
  selector:
    matchLabels:
      app: my-app  # Ensures Pods with this label are managed by the Deployment
  template:
    metadata:
      labels:
        app: my-app  # Labels assigned to Pods created by this Deployment
      annotations:
        description: "This is an example deployment"  # Optional metadata
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80  # Expose port 80 inside the container
---
# Service Definition
apiVersion: v1  # API version for Services
kind: Service  # Defines the type of resource (Service)
metadata:
  name: my-service  # Name of the Service
  labels:
    app: my-app  # Labels to categorize the Service
spec:
  selector:
    app: my-app  # Matches Pods with this label
  ports:
    - protocol: TCP  # Protocol type
      port: 80  # Exposed port on the service
      targetPort: 80  # Port on the Pod to forward traffic to
  type: ClusterIP  # Service type (ClusterIP for internal access)
```
---
---

## Updating Resources
- **Apply Changes:**
  ```bash
  kubectl apply -f manifest.yaml
  ```
  Tracks changes via the `kubectl.kubernetes.io/last-applied-configuration` annotation.

- **Dry-Run and Validation:**
  Use `--dry-run=server --validate=true` to preview changes before applying.

- **Force Pod Restart:**
  Some changes (e.g., ConfigMap updates) require a manual restart:
  ```bash
  oc rollout restart deployment <deployment-name>
  ```

---

## Comparing and Patching Resources
- **Compare Manifests and Live State:**
  ```bash
  kubectl diff -f manifest.yaml
  ```

- **Patch Resources:**
  Apply quick fixes with:
  ```bash
  oc patch <resource> -p '{"spec": {"replicas": 3}}'
  ```

---

## Kustomize Overlays
Kustomize simplifies environment-specific customizations for Kubernetes deployments. Instead of duplicating manifests, Kustomize applies overlays to a single base configuration.

### Structure:
1. **Base Directory:**
   Contains `kustomization.yaml` and resource files for the application.
2. **Overlay Directories:**
   Customize the base with environment-specific changes (e.g., dev, test, prod).

### Key Features:
- **Name Modifiers:** Add prefixes or suffixes to resource names.
- **Namespace Settings:** Set namespaces for resources.
- **Common Labels/Annotations:** Apply labels or annotations universally.
- **Resource Generators:**
  - `configMapGenerator` and `secretGenerator` create ConfigMaps and Secrets.
  - Example:
    ```yaml
    configMapGenerator:
      - name: app-config
        files:
          - config.properties
    ```

### Benefits:
- Simplifies multi-environment deployments.
- Reduces risk by avoiding duplicated manifests.
- Enhances maintainability and clarity.

---

## Summary
- **Declarative Approach:** Use manifests for version-controlled, reproducible deployments.
- **Kustomize Integration:** Leverage Kustomize for environment-specific configurations.
- **Resource Management Tools:** Use commands like `kubectl diff`, `oc patch`, and `kubectl apply` to manage changes effectively.

Declarative resource management ensures consistent, reliable, and scalable Kubernetes deployments, ideal for production environments.

