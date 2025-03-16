# OpenShift API Server


```yaml
oc get clusteroperators kube-apiserver # Check Cluster API Health
oc get --raw /readyz # Check API Availability
oc get pods -n openshift-kube-apiserver # Check Pod Status
oc describe pod -n openshift-kube-apiserver <api-server-pod> # Describe the API Server Deployment
oc get pods -A | grep kube-apiserver # Check Running Pods

Verify etcd Health:
oc get pods -n openshift-etcd
oc logs -n openshift-etcd -l app=etcd
oc exec -n openshift-etcd etcd-member-ip-<node> -- etcdctl endpoint status --write-out=table

Validate Certificates & Authentication:
oc get secret -n openshift-kube-apiserver | grep tls
oc whoami
oc whoami --show-token
curl -k https://<api-server>:6443/version

Check API Server Endpoints:
oc whoami --show-server
curl -k -H "Authorization: Bearer $(oc whoami -t)" https://<api-server>/apis

Check API Server Metrics: Helps identify slow API requests.
oc get --raw /metrics | grep apiserver_request_duration_seconds

Validate API Server Resource Quotas & Limits:
oc get resourcequotas -n openshift-kube-apiserver

api-resources:
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

The OpenShift API Server (openshift-kube-apiserver) is a core control plane component that exposes the Kubernetes and OpenShift-specific APIs. It acts as the primary interface for cluster operations, authenticating and validating requests before updating etcd (the cluster's key-value store).

Key Responsibilities:
- Exposes OpenShift and Kubernetes APIs
- Validates and processes API requests
- Handles authentication and RBAC authorization
- Manages OpenShift-specific resources (e.g., Routes, Builds, Projects)
- Communicates with etcd for persistent state storage

Key Differences between kube-apiserver & openshift-apiserver:
- Functionality: kube-apiserver handles core Kubernetes functionalities, while openshift-apiserver adds OpenShift-specific features.
- Resource Types: kube-apiserver manages Kubernetes resources, and openshift-apiserver manages OpenShift resources.
- API Extensions: openshift-apiserver extends the Kubernetes API to include OpenShift-specific APIs.
- The API group often gives a clue: If the group is apps, v1, or similar, it's likely a core Kubernetes resource handled by kube-apiserver. If the group is apps.openshift.io, route.openshift.io, or similar, it's an OpenShift resource handled by openshift-apiserver.

Think of it this way: 
- openshift-kube-apiserver is a specific instance of kube-apiserver that's part of the OpenShift platform and managed by its dedicated operator
- kube-apiserver is the core Kubernetes API server.
- openshift-apiserver extends the Kubernetes API with OpenShift-specific resources.
- openshift-kube-apiserver is simply another way to refer to the Kubernetes API server within the OpenShift environment, emphasizing its management by the cluster-kube-apiserver-operator.

Kubernetes-Native Resources:
- Pods	Running containers
- Deployments	Manage ReplicaSets
- ReplicaSets	Maintain replica count
- ConfigMaps	Store non-sensitive data
- Secrets	Store sensitive data
- Services	Expose applications
- Nodes	Cluster worker nodes
- PersistentVolumes (PV)	Cluster-wide storage
- PersistentVolumeClaims (PVC)	Storage request

OpenShift-Specific Resources:
- Routes	OpenShift Ingress resource
- Projects	Namespaces with OpenShift-specific settings
- Builds	OpenShift build automation
- ImageStreams	Manage container images
- DeploymentConfigs	OpenShift-specific deployment mechanism
- NetworkPolicies	Control inter-pod communicatio

How API Server Manages Resources:
- Validation: Ensures API requests comply with schema.
- Authentication & Authorization: Uses OAuth, RBAC, and certificates.
- Persistence: Stores resource state in etcd.
- Controllers: Communicates with controllers to maintain desired state.

