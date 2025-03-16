## CoreDNS in Kubernetes

CoreDNS is the default DNS server in Kubernetes, responsible for service discovery and internal DNS resolution within the cluster. It runs as a Kubernetes Deployment and serves as a DNS server for all pods, resolving both internal (ClusterIP services, pod names) and external domain names (e.g., google.com).
- Uses plugins for flexible configuration.
- Can forward DNS queries to external resolvers (e.g., Cloudflare, Google DNS).
---

### How CoreDNS Works Internally

CoreDNS Architecture in Kubernetes:
- Runs as a Deployment (coredns) in the kube-system namespace.
- Uses a ConfigMap (coredns) to define rules for DNS resolution.
- Pods inside Kubernetes query CoreDNS when resolving internal/external domains.

Internal DNS Resolution:
- Pod makes a DNS request.
- Kubernetes sends the request to CoreDNS (10.96.0.10 by default).
- CoreDNS checks its cache/config. If it's a Kubernetes Service, it resolves it using service records. If it's an external domain, it forwards the request to external resolvers (Google DNS, etc.).
- Response is returned to the pod.
---

### Validate CoreDNS in Kubernetes

```yaml

kubectl get pods -n kube-system | grep coredns # Check if CoreDNS is Running
kubectl logs -n kube-system -l k8s-app=kube-dns # Check CoreDNS Logs for Errors
kubectl run -it --rm test-pod --image=busybox --restart=Never -- nslookup my-service.default # Validate DNS Resolution from a Pod
kubectl get cm coredns -n kube-system -o yaml # Check CoreDNS Configuration
```
---

## CoreDNS in OpenShift

OpenShift (built on Kubernetes) also uses CoreDNS for service discovery and DNS resolution. However, OpenShift integrates CoreDNS with its Cluster Network Operator (CNO) and DNS Operator to manage configurations dynamically.

### Feature Comparison: Kubernetes CoreDNS vs. OpenShift CoreDNS

| Feature | Kubernetes CoreDNS | OpenShift CoreDNS |
|---|---|---|
| Default DNS Service | coredns Deployment in kube-system | Managed by DNS Operator in openshift-dns |
| Config Management | coredns ConfigMap | Managed via OpenShift DNS CRD |
| Custom DNS Forwarding | Modify coredns ConfigMap | Use DNS CR (customdns) |
| Service Discovery | `svc.cluster.local` | `svc.cluster.local` (same as Kubernetes) |
---

### Components of DNS in OpenShift
- DNS Operator (openshift-dns-operator): Manages DNS configuration dynamically.
- DNS DaemonSet (openshift-dns): Runs CoreDNS on each node.
- CoreDNS Config (/etc/resolv.conf): Points pods to the internal CoreDNS IP.

```yaml
oc get pods -n openshift-dns # Check CoreDNS Pods in OpenShift
oc get deployment -n openshift-dns-operator # Check the DNS Operator
oc logs -n openshift-dns -l dns.operator.openshift.io/daemonset=dns # Check CoreDNS Logs
oc run test-pod --image=registry.access.redhat.com/ubi8/ubi-minimal -it -- /bin/sh # Validate Internal DNS Resolution
```
