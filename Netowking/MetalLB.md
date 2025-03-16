## MetalLB LoadBalancer

MetalLB is a LoadBalancer implementation for bare metal Kubernetes and OpenShift clusters. It provides a way to expose services with LoadBalancer type, similar to how cloud providers do, but on on-premises or bare metal infrastructure. It enables high availability and load balancing without requiring external hardware load balancers.

---
### How MetalLB Works
MetalLB operates in two main modes:

#### Layer 2 (L2) Mode:
- MetalLB uses ARP (Address Resolution Protocol) or NDP (Neighbor Discovery Protocol) to announce IP addresses.
- It broadcasts the IP to the local network, making the node that runs the service respond to traffic for that IP.
- This is simpler but requires all nodes to be on the same Layer 2 network (same subnet).

#### BGP (Border Gateway Protocol) Mode:
- MetalLB uses BGP to announce service IPs to external routers.
- The router then routes the traffic to the node hosting the service.
- This mode is more complex but allows multi-subnet or more advanced routing scenarios.

---
### MetalLB Architecture
MetalLB has two main components:

#### Controller
- Runs as a deployment.
- Monitors LoadBalancer type services and allocates IPs from the configured IP pool.
- Updates the service's status with the assigned external IP.

#### Speaker
- Runs as a DaemonSet on each node.
- Advertises the external IPs via ARP (L2 mode) or BGP (BGP mode).
- Ensures that traffic to the external IP is routed to the correct node.

---
### How MetalLB Assigns IPs:

IP Address Pool:
- You define an IP pool in a IPAddressPool resource. (Assigning a Persistent IP to a Service) (We can also create a Service with Static IP by defining in the svc deployment file)
- MetalLB allocates IPs from this pool for LoadBalancer services.

Service Exposure:
- When you expose a service with type: LoadBalancer, MetalLB:
- Allocates an IP from the pool.
- Updates the service's status with the external IP.
- Advertises the IP using ARP (L2) or BGP.

Traffic Flow:
- Incoming traffic to the LoadBalancer IP is routed to one of the OpenShift nodes.
- kube-proxy then load-balances this traffic to the service's pods.

---

### Prerequisites

OpenShift Cluster: Ensure your 5-node on-prem OpenShift cluster is up and running.

Network Requirements:
- A dedicated pool of IP addresses for MetalLB to use.
- Ensure that these IPs are within the same subnet as your OpenShift nodes and are not managed by DHCP.
- No other services (like DHCP) should allocate these IPs.

Add static route & firwwall rules:
- Add static route on each openshift node
- Add routing in switch/routeer # sudo route -n add -net 192.168.22.0/24 192.168.0.200
- Add firewall rules

---
### Create a deployment and LoadBalancer Service:
```yaml
Create deployment and expose with loadbalancer service:
kubectl create deploy helloworld-yellow --image=docker.io/rhay/helloworld:v1 -n helloworld
kubectl expose deploy helloworld-yellow --type LoadBalancer --port 80 --target-port 3000 -n helloworld

You will the External IP is pending at this time, check if you can access container with internal IP & one of nodeIP with nodeport:
kubectl get svc -n helloworld  
kuberkubectl run curl -n helloworld -it --rm --image curlimages/curl -- /bin/sh -n helloworld
curl 172.30.209.22 # Curl internal IP
curl helloworld-blue
http://192.168.22.201:32408 # Access from outside with one of node IP and nodeport.
```

---
### Install MetalLB

Install MetalLB Operator
```yaml
Install MetalLB Operator:
Go to the OpenShift Administrator Console.
Navigate to Operators > OperatorHub.
Search for MetalLB Operator and install it:
Set the installation namespace to metallb-system.
Set the installation mode to All namespaces.
```

If you prefer using the CLI follow 
- https://metallb.io/installation -> Installation by manifest
- The metallb-system/controller deployment. This is the cluster-wide controller that handles IP address assignments.
- The metallb-system/speaker daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.
- Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.
```yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
# This will deploy MetalLB to your cluster, under the metallb-system namespace
```

---
### Configure MetalLB Custom Resource (IPAddressPool)

- Follow with https://metallb.io/configuration
  
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: my-ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.22.160-192.168.22.190  # Persistent IP

oc apply -f <filename>.yaml
```

- Create a Service with Static IP: Create a service with type: LoadBalancer and specify the IP in loadBalancerIP
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: default
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.1.200  # Persistent IP from pool
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: my-app

oc apply -f <filename>.yaml
```
---
### When to Use L2 vs. BGP Mode:

L2 Mode:
- Simpler to set up.
- Ideal if all nodes are on the same subnet.
- Recommended for small to medium-sized clusters.

BGP Mode:
- Needed for multi-subnet networks.
- Suitable for advanced routing and larger clusters.
- Requires a BGP-capable router or switch.

---
### Create L2 Advertisement (or Create BGP Mode for advanced netwoking)
- This enables Layer 2 mode for IP advertising:
```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: my-l2-adv
  namespace: metallb-system

oc apply -f <filename>.yaml
```

---
### Cloud Compatibility (MetalLB on OpenShift OCP)
- Check cloud compatibility here - https://metallb.io/installation/clouds/
- To run MetalLB on OpenShift, two changes are required: changing the pod UIDs, and granting MetalLB additional networking privileges.
- Pods get UIDs automatically assigned based on an OpenShift-managed UID range, so you have to remove the hardcoded unprivileged UID from the MetalLB manifests. You can do this by removing the spec.template.spec.securityContext.runAsUser field from both the controller Deployment and the speaker DaemonSet. Also, as the allowed group ID range in Openshift is 5000 through 5999, you have to remove spec.template.spec.securityContext.fsGroup field as well.
- Additionally, you have to grant the speaker DaemonSet elevated privileges, so that it can do the raw networking required to make load balancers work. You can do this with:
```yaml
oc adm policy add-scc-to-user privileged -n metallb-system -z speaker
```
- After that, MetalLB should work normally.

---
### Verify the Installation

```yaml
oc get all -n metallb-system # List all resources, including Pods, Services, ConfigMaps, and Custom Resources

If you're unsure about the available MetalLB CRDs, list all of them using:
oc api-resources | grep metallb
oc get crd | grep metallb # To see all CRDs installed by MetalLB
oc explain <crd-name> # For detailed information about a specific CRD

List MetalLB Custom Resources:m
oc get ipaddresspool -n metallb-system
oc get l2advertisement -n metallb-system
oc get bgpadvertisement -n metallb-system
oc get configmap -n metallb-system

Export All Configurations to YAML:
oc get all -n metallb-system -o yaml > metallb-all-config.yaml
oc get ipaddresspool,l2advertisement,bgpadvertisement -n metallb-system -o yaml > metallb-crds.yaml
```
---
### Validation with new app
It will create a new app and assigned cluster IP , we can patch service and change the type: LoadBalancer from ClusterIP
```
oc new-app --name=newapp openshift/hello-openshift -n metallb                      
oc patch svc newapp -n metallb --type='merge' -p '{"spec": {"type": "LoadBalancer"}}'
```
