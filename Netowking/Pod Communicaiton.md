## Pod Communicaiton

### Pod IP Assignment
- Every pod in Kubernetes gets a unique IP address.
- The Container Network Interface (CNI) plugin (like Flannel, Calico, Cilium) assigns these IPs.
- The CNI configures network routes so the pod can communicate with others. The pod keeps this IP until it is deleted. Pods can talk to each other directly using their IPs.
---

### Service IP Assignment
- Services act as stable entry points for pods.
- Kubernetes assigns a ClusterIP from the service network range
- The service does not change its IP, even if backend pods restart.

Types of Kubernetes Services & IP Handling:
- ClusterIP (Default) - Only accessible inside the cluster.
- NodePort - Exposes the service on each node’s IP and a fixed port (30000-32767). External users access via NodeIP:NodePort.
- LoadBalancer - Assigns an external IP (cloud provider assigns this). Directs traffic to the right pods.
- ExternalName - No IP; maps to an external domain like db.example.com
---

### Mapping Pods to Services
- Kubernetes maps services to pods dynamically using label selectors.
- The Kube-proxy component ensures requests reach the correct pods.
- Kubernetes load balances traffic across all matching pods. Selects one of the pod replicas (round-robin, random, or least connections based on backend).
---

### Exposing a Kubernetes Service for External Network Access

To expose a Kubernetes service externally, we can use several methods like Port Forwarding, NodePort, LoadBalancer, or Ingress with HAProxy. Below, I'll explain the setup where we use HAProxy as an external load balancer to manage traffic flow.

 Network Flow: External Client → HAProxy → Node → Service → Pod
 - Client Requests a DNS-Resolved Domain. The DNS resolves app.example.com to the HAProxy external IP.
 - HAProxy is set up externally or within the Kubernetes cluster. It forwards traffic to worker nodes on a specific port (NodeIP:Port). It uses L4 (TCP) or L7 (HTTP) routing.
 - The worker node has the Kubernetes Service (NodePort or ClusterIP) running. HAProxy forwards traffic to NodeIP:30080 (NodePort) or directly to ServiceIP.
 - The service (10.100.1.10:80) uses kube-proxy (iptables/IPVS) to forward traffic to an available pod. One of the pod replicas handles the request.
