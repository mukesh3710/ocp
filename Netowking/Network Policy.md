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
