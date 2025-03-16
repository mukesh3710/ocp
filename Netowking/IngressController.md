## IngressController

To configure an Ingress Controller on a Bare Metal OpenShift Cluster using MetalLB, follow these high-level steps:
- Create a Namespace for MetalLB & Deploy MetalLB
- Configure a MetalLB Address Pool & L2Advertisement
- Deploy a Custom Ingress Controller & Verify the Ingress Controller and Service
- Create and Expose an Application using the New Ingress
- Integrate HAProxy for external access if needed.
- Use cert-manager for TLS certificates.
- Fine-tune MetalLB for better performance.

---
#### Ingress Controller validation
```yaml
oc get pods -n openshift-ingress #  to list the Ingress Controller pods

oc get deployment -n openshift-ingress # the Ingress Controller Deployment
oc describe deployment router-default -n openshift-ingress # For detailed configuration

oc get svc -n openshift-ingress # the Ingress Controller Service

oc get ingresscontroller -n openshift-ingress-operator # the IngressController Resource
oc describe ingresscontroller default -n openshift-ingress-operator # details

oc get routes --all-namespaces # To list all Routes
oc get route <route-name> -n <namespace> # For a specific application
oc describe route <route-name> -n <namespace>
```
---

#### Method to use Ingress

- LoadBalancerService: If your platform supports LoadBalancers (AWS, GCP, Azure, MetalLB).
- HostNetwork (default): Directly accessible via node IPs (requires external DNS setup).
- NodePort: If LoadBalancer is unavailable, expose via worker node ports.

If you need public access, I recommend Option 1 (LoadBalancer). If you're running bare metal, Option 2 (HostNetwork) or Option 3 (NodePort) will work.
```yaml
oc get svc -A | grep LoadBalancer # Check if MetalLB is properly running and assigned an IP to other services.
```

---
#### Create a New IngressController

We alerady have metallb configured as load balancer which can assing external IP to server

Apply the following YAML to create a new IngressController (custom-ingress) in openshift-ingress-operator namespace.
```yaml
cat <<EOF | oc apply -f -
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  name: custom-ingress
  namespace: openshift-ingress-operator
spec:
  domain: custom.apps.lab.ocp.lan  # Change domain if needed
  endpointPublishingStrategy:
    type: LoadBalancerService
  replicas: 2
EOF
```
Check if the new IngressController is created properly
```yaml
oc get ingresscontroller -n openshift-ingress-operator
oc get svc -n openshift-ingress # Check the Service for the New Ingress
```

---

#### Test the New Ingress
```yaml
oc new-project test-app
oc new-app --name=newapp openshift/hello-openshift -n test-app
oc expose svc newapp --name=newapp-route --hostname=newapp.custom.apps.lab.ocp.lan -n test-app
oc get route -n test-app
curl -k http://hello-openshift-test-app.custom.apps.lab.ocp.lan

new-app --name=newapp1 nginx -n test-app
oc expose svc httpd --name=1newapp-route --hostname=1newapp.custom.apps.lab.ocp.lan -n test-app
```
---

#### To do

Confiure external load balacer with HA proxy
