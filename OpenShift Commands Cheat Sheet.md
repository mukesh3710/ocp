
# OpenShift Commands Cheat Sheet

## Help
```bash
kubectl help # to list and review the available commands for the kubectl command
kubectl create --help # Examine the available subcommands and options for the kubectl create command 
oc help # List and review the available commands for the oc binary
oc create --help # Use the --help option with the oc create command to view the available subcommands and options.
```
---

## Login
```bash
oc login --server=https://api.my.test.com:6443 --username=admin --password=admin
export KUBECONFIG=/Users/mukesh3710/Documents/Study/Open_Shift/SNO/kubeconfig
oc login -u admin -p redhatocp # Log in to the cluster
```
---

## Cluster Information
```bash
oc version # Identify the cluster version 
oc cluster-info # Identify the supported API versions by using the api-versions command.
oc whoami --show-console # Identify the URL for the OpenShift web console
oc get clusteroperator # List cluster operators 
oc get pods -n openshift-apiserver # to list pods in the openshift-api project
oc status -n openshift-authentication # to retrieve the status of resources in the openshift-authentication project.
oc explain services # to list the description and available fields for services resources.
oc get nodes # list cluster nodes.
oc get dns cluster -o jsonpath='{.spec.baseDomain}' # find the base domain
oc get infrastructure cluster -o yaml # To retrieve detailed information about the cluster
oc get route -A # To view routes, including the console and other services
oc get clusterversion # Verifies the OpenShift cluster version and ensures that the cluster components have been installed and are running the correct versions.
oc describe clusterversion version
oc describe clusteroperator authentication # To get more information about why the authentication operator
oc adm upgrade # To see the update history and details
oc get pods -n openshift-cluster-version # Verifies the pods related to the cluster version operator are running correctly.
```

---

## Events & Logs
```bash
oc get events --sort-by='.lastTimestamp' # with timestamp
oc get events -A # Check the cluster-wide events for errors or warnings
oc adm must-gather # Cluster diagnostic information
journalctl -u kubelet # Inspect Node Logs
oc get events -n openshift-authentication-operator # Examine Authentication Operator Events
```

---

## OpenShift Core Components

### API Server
```bash
oc get clusteroperators kube-apiserver # Check Cluster API Health
oc get --raw /readyz # Check API Availability
oc get pods -n openshift-kube-apiserver # Check Pod Status
oc describe pod -n openshift-kube-apiserver <api-server-pod> # Describe the API Server Deployment
oc get pods -A | grep kube-apiserver # Check Running Pods

# Verify etcd Health
oc get pods -n openshift-etcd
oc logs -n openshift-etcd -l app=etcd
oc exec -n openshift-etcd etcd-member-ip-<node> -- etcdctl endpoint status --write-out=table

# Validate Certificates & Authentication
oc get secret -n openshift-kube-apiserver | grep tls
oc whoami
oc whoami --show-token
curl -k https://<api-server>:6443/version

# Check API Server Endpoints
oc whoami --show-server
curl -k -H "Authorization: Bearer $(oc whoami -t)" https://<api-server>/apis

# Check API Server Metrics: Helps identify slow API requests.
oc get --raw /metrics | grep apiserver_request_duration_seconds

# Validate API Server Resource Quotas & Limits
oc get resourcequotas -n openshift-kube-apiserver

# List API Resources
oc api-resources # List the available resource types 
oc api-resources --namespaced # to limit the output of the api-resources command to namespaced resources.
oc api-resources --namespaced -o name
oc api-resources --namespaced=false # Limit the output of the api-resources command to non-namespaced resources.
oc api-resources --api-group '' # to show only resources from the core API group
oc explain pods # to list a description and the available fields for the pods resource type
oc get pods # List all pods 
oc describe pod myapp-54fcdcd9d7-2h5vx #  to view the configuration and events for the pod
oc get pod myapp-54fcdcd9d7-2h5vx -o yaml # Retrieve the details of the pod in a structured format.
```

---

## Infrastructure & Node Management

### Machine Config Operator (MCO)
```bash
oc get machineconfigpool
oc get clusteroperator machine-config
oc get pods -n openshift-machine-config-operator
```

### Cluster Autoscaler
```bash
oc get clusterautoscaler
```

### Node Tuning Operator
```bash
oc get tuneds -n openshift-cluster-node-tuning-operator
```

---

## Networking & Ingress

### Ingress Operator
```bash
oc get ingresscontroller -n openshift-ingress-operator
oc get clusteroperator ingress
```

### Cluster Network Operator (CNO)
```bash
oc get network.operator.openshift.io
```

### Multus
```bash
oc get pods -n openshift-multus
```

---

## Security & Certificates

### Authentication & OAuth
```bash
oc get oauth
oc get clusteroperator authentication
```

### Certificate Management
```bash
oc get certificates -n openshift-ingress-operator
oc get csr
oc describe csr <csr-name>
oc get csr --no-headers | awk '/Pending/ {print $1}' | xargs oc adm certificate approve # Approved all pending certificate 
```

### Compliance Operator
```bash
oc get compliancesuite -n openshift-compliance
```

---

## Storage & Persistent Data

### OpenShift Data Foundation (ODF)
```bash
oc get storagecluster -n openshift-storage
oc get pods -n openshift-storage
```

### Persistent Volumes (PV) & Persistent Volume Claims (PVC)
```bash
oc get pv
oc get pvc -n <namespace>
```

### Storage Classes
```bash
oc get sc
```

---

## Monitoring & Logging

### Cluster Monitoring
```bash
oc get clusteroperator monitoring
oc get pods -n openshift-monitoring
```

### Cluster Logging
```bash
oc get clusterlogging -n openshift-logging
```

---

## Application Management

### Image Registry
```bash
oc get clusteroperator image-registry
oc get pods -n openshift-image-registry
```

### OpenShift Pipelines (Tekton)
```bash
oc get pods -n openshift-pipelines
```

### OpenShift GitOps (ArgoCD)
```bash
oc get pods -n openshift-gitops
```

### OpenShift Serverless (Knative)
```bash
oc get knative-serving -n knative-serving
```

### OpenShift Service Mesh (Istio)
```bash
oc get servicemeshcontrolplane -n istio-system
```

---

## Operator Lifecycle Management

### OperatorHub & OLM
```bash
oc get catalogsource -n openshift-marketplace
oc get subscriptions -n <namespace>
oc get csv -n <namespace>
```

---

## Additional Enterprise Components

### OpenShift Virtualization
```bash
oc get kubevirt -n openshift-cnv
```

### OpenShift Quotas and Limits
```bash
oc get resourcequotas -n <namespace>
```

### OpenShift Cluster API
```bash
oc get cluster
```

