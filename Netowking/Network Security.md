# Network Security in OpenShift (OCP)

This guide focuses on securing both external and internal communication in OpenShift using TLS encryption, network policies, and service certificates. These tools help ensure secure communication, enforce traffic isolation, and comply with zero-trust security principles.

---

## Securing External Traffic with TLS in OpenShift

### OpenShift Routes for External Access
- **Routes** expose applications with unique hostnames for external access.
- **Routers** redirect traffic from public IPs to application pods.

### TLS Termination Options for Routes

#### 1. Edge Termination
- TLS terminates at the router.
- Router serves certificates (custom or OpenShift-provided).
- Internal traffic (router to pods) remains unencrypted.

#### 2. Passthrough Termination
- Encrypted traffic reaches the application directly.
- Application serves its own certificates.
- Enables mutual authentication between client and application.

#### 3. Re-encryption Termination
- TLS terminates at the router with a client-trusted certificate.
- Router re-encrypts traffic for pods using service certificates.
- Ensures end-to-end encryption.

### Securing Applications with Routes
- **Edge Routes**: Use `oc create route edge` with custom TLS certificates. Suitable when internal traffic encryption isn't critical.
- **Passthrough Routes**: Applications handle their certificates. Use OpenShift secrets to store certificates.
- **Re-encrypt Routes**: Offers full encryption with trusted certificates for external and internal communication.

---

## Securing Communication with Network Policies

### Network Policies for Pod Isolation
- Define granular rules for allowed traffic to/from pods.
- Use pod labels and selectors for logical network zoning.

### Example: Network Policy Structure
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-qa-to-product-catalog
  namespace: network-1
spec:
  podSelector:
    matchLabels:
      deployment: product-catalog
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          network: network-2
      podSelector:
        matchLabels:
          role: qa
    ports:
    - protocol: TCP
      port: 8080
```

### Real-World Examples
1. **Restricting Traffic**: Only allow QA pods in `network-2` to access product catalog pods in `network-1`.
2. **Deny-All Policies**: Start with a default deny-all policy and explicitly allow required traffic.

---

## Securing Internal Traffic with TLS in OpenShift

### Zero-Trust Security with Service Certificates
- OpenShift encrypts control plane and node communication by default.
- Applications verify identities using service certificates provided by the service-ca controller.

### Configuring Service Certificates
1. Annotate a service:
   ```bash
   kubectl annotate service <service-name> service.beta.openshift.io/serving-cert-secret-name=<secret-name>
   ```
2. The service-ca controller generates a secret containing the certificate and key.
3. Mount the secret in application pods.

#### Example Deployment with Mounted Certificates
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-tls
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: tls-secret
      mountPath: /etc/tls
      readOnly: true
  volumes:
  - name: tls-secret
    secret:
      secretName: hello-secret
```

### Client-Side Configuration
- Inject CA bundles into client applications for certificate verification.
  ```bash
  kubectl annotate configmap <configmap-name> service.beta.openshift.io/inject-cabundle=true
  ```

---

## Alternatives for Internal TLS
- **Service Mesh**: Provides advanced features like traffic management and observability.
- **Cert-Manager**: Automates certificate management, including signing and renewal.

---

## Summary
- Secure external traffic with TLS termination (edge, passthrough, or re-encryption).
- Use network policies for granular traffic control between pods.
- Leverage the OpenShift service-ca controller for internal traffic encryption.
- Incorporate CA bundles and service certificates to ensure secure communication in production environments.

By implementing these best practices, you can achieve robust network security and adhere to zero-trust principles in your OpenShift cluster.
