# Network Policy

In OpenShift, Network Policies are used to control the traffic flow between pods at the network level. These policies define Ingress (incoming) and Egress (outgoing) traffic rules.

Types of Network Policies:
Ingress Policy
- Controls incoming traffic to a pod.
- Defines which sources (pods, namespaces, or IP ranges) are allowed to send traffic to the target pods.

Egress Policy
- Controls outgoing traffic from a pod.
- Defines which destinations (pods, namespaces, or IP ranges) are allowed to receive traffic from the source pods.

---

Key Components of Network Policy:

Pod Selector (Pod Label)
- Specifies which pods the policy applies to using labels.
- Example: matchLabels: {app: web}

Namespace Selector
- Allows traffic from/to specific namespaces.
- Example: namespaceSelector: {matchLabels: {project: dev}}

IPBlock (Subnet)
- Defines allowed/denied IP ranges (CIDR format).
- Example: ipBlock: {cidr: 192.168.1.0/24}

Rules (Ingress/Egress)
- Allow: Explicitly allows traffic.
- Deny: Implicitly denies all traffic if no allow rules exist.

AND vs OR in Rules
- AND: Multiple match conditions must all be true (e.g., podSelector + namespaceSelector).
- OR: Multiple rules within ingress or egress apply as OR conditions (if any match, traffic is allowed).

---

Example YAML Network Policies:

Allow Ingress from a Specific Namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-dev
  namespace: production
spec:
  podSelector: # Apply policy to pods with label "app: web"
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector: # Allow traffic from "dev" namespace
        matchLabels:
          project: dev
    ports:
    - protocol: TCP
      port: 80
```

Allow Egress to a Specific Subnet
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-internet
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.1.0/24 # Allow access to this subnet
    ports:
    - protocol: TCP
      port: 443
  policyTypes:
  - Egress
```

Deny All Ingress Traffic (Default Deny)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
---

Applying Network Policies:

Since OpenShift does not provide a CLI command to create network policies, you must apply them using YAML manifests.

```bash
oc apply -f policy.yaml # Apply Policy
oc get networkpolicy -n <namespace> # Check Existing Policies
oc describe networkpolicy <policy-name> -n <namespace> #  Describe a Specific Policy
```
---

Applying Policy Within & Across Namespaces:
- Within the Same Namespace: Use podSelector to apply policies between pods in the same namespace.
- Across Namespaces: Use namespaceSelector to allow/deny traffic between namespaces.
