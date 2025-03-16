# Configuring Applications for Reliability in OCP

## Ensuring High Availability (HA)

### Key HA Concepts
- **High Availability (HA):** Applications remain functional despite failures.
- **Kubernetes HA Techniques:**
  - **Pod Restarting:** Automatic restarts for failed pods.
  - **Health Probes:** Detect and recover from unhealthy states.
  - **Horizontal Scaling:** Adjust replicas dynamically based on load.

### Application Design Considerations
- **Reliable Code:** Minimize crashes and errors.
- **Health Checks:** Use probes to signal health status.
- **Resource Efficiency:** Avoid resource overuse.
- **Graceful Termination:** Handle termination signals appropriately.

### Implementing HA with Kubernetes

#### Pod Restarting
- Use a **restart policy** (`Always`, `OnFailure`, or `Never`).
- Kubernetes automatically restarts pods that fail or fail health checks.

#### Health Probes
- **Liveness Probe:** Verifies if the container is responsive.
- **Readiness Probe:** Ensures the container is ready to handle traffic.
- **Startup Probe:** Validates successful startup for applications with long initialization times.

#### Horizontal Scaling
- Dynamically adjust replicas to handle fluctuating load.
- Helps maintain availability and performance under varying traffic.

#### Additional Considerations
- **Resource Limits and Requests:** Prevent resource contention.
- **Network Reliability:** Ensure stable pod-to-service communication.
- **Persistent Storage:** Use for data that must survive pod restarts.
- **Security:** Harden applications against vulnerabilities.

## Using Application Health Probes

### Probe Types
- **Liveness Probe:** Checks container responsiveness.
- **Readiness Probe:** Verifies if the container can serve requests.
- **Startup Probe:** Confirms successful application startup.

### Configuring Probes
- **PeriodSeconds:** Frequency of checks.
- **FailureThreshold:** Consecutive failures before action.
- Probe Types:
  - **HTTP GET:** Sends a request to a specified path.
  - **TCP Socket:** Opens a socket to a port.
  - **Exec:** Executes a command in the container.

### Benefits of Probes
- Automatically restart unhealthy containers.
- Direct traffic only to healthy containers.
- Ensure resource efficiency and application reliability.

### Best Practices
- Use simple probes to avoid performance issues.
- Test probe configurations in different scenarios.
- Configure **Startup Probes** for applications with slow initialization.
- Monitor logs to diagnose probe failures.

## Resource Management for Compute Capacity

### Pod Scheduling
- **Node Filtering:**
  - Match labels between pods and nodes.
  - Ensure sufficient CPU, memory, and storage availability.
- **Node Prioritization:**
  - Assign weighted scores to nodes for optimal placement.
  - Highest score determines pod placement.

### Resource Requests
- Define minimum guaranteed compute resources (e.g., `cpu: 100m`, `memory: 1Gi`).
- Used by the scheduler for placement decisions.

### Resource Inspection
- View cluster and pod resource usage with:
  - `oc describe node`
  - `oc adm top pods`
  - `oc adm top nodes`

### Best Practices
- Set realistic requests and limits.
- Monitor usage to identify bottlenecks.

## Limiting Compute Capacity

### Memory and CPU Limits
- **Memory Limits:** Prevent excessive memory use to avoid crashes.
- **CPU Limits:** Restrict CPU usage to prevent impact on other pods.

### Viewing Resource Usage
- Inspect limits and usage using CLI tools or the web console.

### Best Practices
- Avoid over- or under-provisioning.
- Use resource limits to prevent resource exhaustion.

## Application Autoscaling

### Horizontal Pod Autoscaler (HPA)
- Automatically adjusts the number of pods based on metrics.
- Metrics can include:
  - **CPU Utilization:** Respond to CPU demand.
  - **Memory Utilization:** Handle memory load.
  - **Custom Metrics:** For application-specific needs.

### Configuring HPA
- Define `minReplicas` and `maxReplicas`.
- Set target utilization (e.g., 80% CPU).

### Creating an HPA
- Use `oc autoscale` for quick setup.
- Use YAML for complex configurations.

### Best Practices
- Ensure accurate resource requests and limits.
- Monitor and adjust metrics and thresholds.
- Use custom metrics for specialized applications.

### Additional Considerations
- Choose metrics that align with application behavior.
- Account for scaling delays and stability.

## Summary
- Kubernetes provides tools like health probes and autoscalers to ensure application reliability.
- Resource requests and limits maintain balanced resource usage.
- Horizontal Pod Autoscalers dynamically scale applications to meet demand, ensuring efficient resource utilization and availability.

---

# Configuring and Managing Reliable Applications in OpenShift

## Exploring Pod Restart Policies

1. **Observe Restart Behavior:**
   - Deploy a sample pod with a `restartPolicy` of `Always`, `OnFailure`, and `Never` to understand the differences.
   - Example YAML for a crashing pod:

     ```yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: crash-pod
     spec:
       containers:
       - name: crash-container
         image: busybox
         command: ["/bin/sh", "-c", "exit 1"]
       restartPolicy: OnFailure
     ```

   - Deploy the pod:
     ```bash
     oc apply -f crash-pod.yaml
     ```

   - Observe pod restarts using:
     ```bash
     oc get pods
     ```

2. **Key Observations:**
   - Pods with `restartPolicy: Always` will restart indefinitely.
   - `OnFailure` restarts only on non-zero exit codes.
   - `Never` does not restart the pod regardless of failure.

---

## Managing Slow-Starting Applications Without Probes

1. **Deploy a Slow-Starting Application:**
   - Example Deployment:

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: slow-app
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: slow-app
       template:
         metadata:
           labels:
             app: slow-app
         spec:
           containers:
           - name: slow-container
             image: busybox
             command: ["/bin/sh", "-c", "sleep 300"]
     ```

   - Deploy the application:
     ```bash
     oc apply -f slow-app.yaml
     ```

2. **Monitor Pod Status:**
   - Use `oc get pods` to observe behavior during startup.

3. **Add Probes to Ensure Reliability:**
   - Update the deployment to include probes:

     ```yaml
     readinessProbe:
       httpGet:
         path: /healthz
         port: 8080
       initialDelaySeconds: 10
       periodSeconds: 5
     livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
       initialDelaySeconds: 30
       periodSeconds: 10
     ```

---

## Configuring Resource Requests and Limits

1. **Set Resource Requests and Limits:**
   - Define CPU and memory limits in your deployment:

     ```yaml
     resources:
       requests:
         memory: "128Mi"
         cpu: "500m"
       limits:
         memory: "256Mi"
         cpu: "1"
     ```

   - Deploy the application:
     ```bash
     oc apply -f resource-config.yaml
     ```

2. **Monitor Resource Utilization:**
   - Use metrics to monitor usage:
     ```bash
     oc adm top pods
     ```

3. **Impact of Resource Requests:**
   - Simulate scaling to understand scheduling constraints by modifying `replicas`.
   - Example:
     ```bash
     oc scale deployment slow-app --replicas=5
     ```

---

## Scaling with Horizontal Pod Autoscaler

1. **Enable Autoscaling:**
   - Create an HPA to scale pods based on CPU usage:

     ```bash
     oc autoscale deployment slow-app --min=1 --max=10 --cpu-percent=80
     ```

2. **Test Autoscaling Behavior:**
   - Generate load using tools like `siege` or custom scripts.
   - Monitor scaling activity:
     ```bash
     oc get hpa
     ```

3. **Ensure N Instances Per Node:**
   - Use `nodeSelector` or `affinity` rules to distribute pods:

     ```yaml
     affinity:
       podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
         - labelSelector:
             matchExpressions:
             - key: app
               operator: In
               values:
               - slow-app
           topologyKey: "kubernetes.io/hostname"
     ```

---

## Deploy and Troubleshoot a Reliable Application

1. **Deploy the Final Application:**
   - Define health probes, resource requests, and autoscaler in a single deployment YAML.

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: reliable-app
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: reliable-app
       template:
         metadata:
           labels:
             app: reliable-app
         spec:
           containers:
           - name: reliable-container
             image: your-app-image
             ports:
             - containerPort: 8080
             resources:
               requests:
                 memory: "128Mi"
                 cpu: "500m"
               limits:
                 memory: "256Mi"
                 cpu: "1"
             readinessProbe:
               httpGet:
                 path: /ready
                 port: 8080
               initialDelaySeconds: 5
               periodSeconds: 10
             livenessProbe:
               httpGet:
                 path: /healthz
                 port: 8080
               initialDelaySeconds: 10
               periodSeconds: 10
     ```

2. **Monitor and Troubleshoot:**
   - Use `oc logs`, `oc describe pod`, and `oc adm top` to debug and optimize performance.

3. **Test Application Behavior:**
   - Simulate load and observe autoscaling to ensure smooth scaling and robust performance.

