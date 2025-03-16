# Exposing Non-HTTP/SNI Applications in Kubernetes

This document explains how to expose non-HTTP/SNI applications in Kubernetes clusters using LoadBalancer services and secondary networks. These methods are essential for providing external access to applications that do not rely on HTTP protocols, such as databases or proprietary communication protocols.

---

## Exposing Applications with LoadBalancer Services

### Limitations of Ingress/Routes
- Designed primarily for HTTP services with virtual hosting.
- Unsuitable for non-HTTP protocols lacking routing mechanisms.

### Kubernetes Service Types
Kubernetes services provide access to pods:
- **ClusterIP**: Internal communication within the cluster.
- **NodePort & LoadBalancer**: Expose services externally.
- **External IP (ClusterIP)**: Limited external exposure (not recommended).

### LoadBalancer Services
- Use external load balancers (cloud providers or MetalLB for bare-metal environments).
- Distribute traffic across pods running the service.
- Require environment-specific network configuration.

#### Steps to Use LoadBalancer Services:
1. **Create a Service**:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-loadbalancer-service
    spec:
      type: LoadBalancer
      ports:
        - port: 12345
          targetPort: 12345
      selector:
        app: my-app
    ```
2. **Verify Service Availability**:
    - Use a client for the protocol to test connectivity.
    - Confirm load balancing works using tools like `ping` or `traceroute`.

#### Important Notes:
- Prefer ingress/routes for HTTP services.
- Load balancer services may require additional security and configuration.
- Always verify after deployment.

---

## Multus Secondary Networks

Secondary networks enhance performance, security, and control for specific traffic.

### Benefits of Secondary Networks
- Improved performance with dedicated networks.
- Enhanced security through network isolation.
- Simplified control over pod traffic.

### Multus CNI Plug-in
- Enables attaching pods to custom networks.
- Configured through **NetworkAttachmentDefinition** resources.

#### Configuring Secondary Networks
1. **Prepare Existing Networks**:
   - Use tools like **NMState** or **SR-IOV** to configure cluster node networks.
   - SR-IOV offers high bandwidth and low latency.

2. **Create NetworkAttachmentDefinition**:
    ```yaml
    apiVersion: k8s.cni.cncf.io/v1
    kind: NetworkAttachmentDefinition
    metadata:
      name: custom-network
    spec:
      config: '{"cniVersion": "0.3.1", "type": "macvlan", "master": "eth0", "ipam": {"type": "host-local", "subnet": "192.168.1.0/24"}}'
    ```

3. **Annotate Pods**:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
      annotations:
        k8s.v1.cni.cncf.io/networks: custom-network
    spec:
      containers:
        - name: app-container
          image: my-image
    ```

4. **Monitor Status**:
   - Use the `k8s.v1.cni.cncf.io/network-status` annotation for real-time status updates.

### Types of NetworkAttachmentDefinitions
- **Host Device**: Attaches a single network interface to a pod.
- **Bridge**: Connects pods to each other and external networks.
- **IPVLAN/MACVLAN**: Optimized Linux drivers for container environments.

#### IP Address Management (IPAM):
- Handles IP assignments using DHCP or static configurations.

---

## Summary
- **LoadBalancer Services** are essential for exposing non-HTTP applications with external access.
- **Multus CNI** enables attaching pods to custom networks for enhanced performance and security.
- Use **NetworkAttachmentDefinitions** to configure secondary networks with flexibility.
- Always validate configurations with appropriate tools and ensure security measures are in place.
