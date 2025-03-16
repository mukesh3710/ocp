## Route in OpenShift

A Route in OpenShift exposes a service to external clients by mapping an external URL to an internal service within the cluster. It allows users to:
- Access applications via HTTP/HTTPS using a hostname (e.g., app.example.com).
- Manage TLS termination for secure connections.
- Apply custom routing rules like path-based routing.
- Routes are implemented using the OpenShift Router, which is based on HAProxy.

---
### MetalLB vs. Ingress vs. Route in OpenShift
| Feature | Route | Ingress | MetalLB |
|---|---|---|---|
| **Purpose** | Exposes HTTP/HTTPS services using hostnames | Exposes HTTP/HTTPS services using hostnames | Provides external IPs for LoadBalancer services |
| **Layer** | Layer 7 (HTTP/HTTPS) | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) |
| **Controller** | OpenShift Router (HAProxy-based) | Ingress Controller (e.g., Nginx, HAProxy) | None (Direct IP allocation) |
| **DNS Requirement** | Required (maps hostname to Router IP) | Required (maps hostname to Ingress IP) | Not required (IP exposed directly) |
| **Security** | Supports TLS termination | Supports TLS termination | No built-in security (relies on service) |
| **Traffic Control** | Path-based, Host-based routing | Path-based, Host-based routing | Basic load balancing (NodePort) |
| **External Access** | Via Router IP (shared IP) | Via Ingress IP (shared IP) | Direct External IP from pool |
| **Custom Headers** | Supports custom HAProxy rules | Limited to Ingress controller capabilities | None |

---
### When to Use Route vs. Ingress vs. MetalLB
| Feature | Route | Ingress | MetalLB |
|---|---|---|---|
| **Use Cases** | Use in OpenShift for HTTP/HTTPS applications requiring host-based routing. | Use for Kubernetes-native HTTP/HTTPS routing. | Use when you need to expose services via external IPs on bare metal clusters. |
| **Key Features/Benefits** | Provides built-in TLS termination and HAProxy customization. | Flexible choice of Ingress Controllers (e.g., Nginx, Traefik). | Ideal for TCP/UDP applications (e.g., databases, legacy apps). Works well with Ingress or Istio for advanced routing. |
| **Ideal For** | Web applications with domain names. | Suitable for hybrid clusters using both OpenShift and vanilla Kubernetes. |  |

---
### How Routes Work in OpenShift:
- Client Request: A user accesses an external URL (e.g., https://app.example.com).
- DNS Resolution: The URL is resolved to the OpenShift Router's IP.
- Routing: The OpenShift Router receives the request and matches it to a Route.
- Service Forwarding: The Router forwards the request to the corresponding Service, which then directs it to a Pod.
- Response: The response is sent back through the Router to the client.

---
### Types of Routes in OpenShift:

Insecure Route:
- Uses HTTP.
- No encryption; traffic is sent in plaintext.
- Example URL: http://app.example.com
```yaml
oc new-app --image=nginx --name=nginx-app
oc expose svc nginx-app --name=nginx-app-route --hostname=myapp.apps.lab.ocp.lan
oc get routes 
curl myapp.apps.lab.ocp.lan
```

Secure Route:
- Uses HTTPS.
- Traffic is encrypted using SSL/TLS.
- Example URL: https://app.example.com
- Supports different types of TLS termination:
  - Edge Termination: TLS is terminated at the Router (traffic between Router and Pod is HTTP).
  - Passthrough: TLS is passed through to the Pod without decryption at the Router.
  - Re-encrypt: TLS is terminated at the Router and re-encrypted before reaching the Pod.

---
### To DO:
Cusom route : my open openshift cluster domain is lab.ocp.la but i want to create new hostname  for my app like my.example.com .. how can we do it?
