## Identity & Access Management (IAM) 

IAM in OpenShift is crucial for managing authentication, authorization, and access control. It allows administrators to define roles, permissions, and authentication methods for users and applications.

---

1. HTPasswd Authentication

Workflow:
- Manually create users and groups using an HTPasswd file.
- Store user credentials in a Kubernetes secret.
- Configure OpenShift OAuth to use HTPasswd as an identity provider.
- Users authenticate with username/password.

Real-World Use Case:
- Used in non-production environments where integration with enterprise identity providers is unnecessary.
- Suitable for small development teams with limited user management needs.

Authorization & RBAC:
- Users are manually added to groups.
- Assign permissions using ClusterRoleBindings at the project or cluster level.
- Example roles: admin, edit, view at the project level.
---

2. LDAP/Active Directory Authentication

Workflow:
- Connect OpenShift to an LDAP server or Active Directory (AD).
- OpenShift syncs users and groups from LDAP.
- Users authenticate using corporate credentials.
- Groups are mapped from LDAP groups to OpenShift RBAC roles.

Real-World Use Case:
- Used in enterprise environments where centralized user authentication is required.
- Ideal for organizations using Active Directory for user access management.

Authorization & RBAC:
- Groups are fetched from LDAP and mapped to OpenShift groups.
- Assign ClusterRoles and RoleBindings to LDAP groups.
- Common roles:
  - Cluster-admin (full control)  
  - Project admin (manage projects)
  - Developer (edit/manage apps)
  - Viewer (read-only access)

---
3. OpenID Connect (OIDC) Authentication

Workflow:
- Configure OpenShift to use an OIDC provider (e.g., Keycloak, Okta, Azure AD).
- Users log in via SSO (Single Sign-On).
- OpenShift retrieves user identity and group information from the provider.

Real-World Use Case:
- Used in cloud-native environments with SSO for multiple platforms.
- Common in hybrid/multi-cloud architectures using Okta, Azure AD, or Google IAM.

Authorization & RBAC:
- Users are automatically assigned to groups based on OIDC claims.
- Roles assigned to OIDC groups using ClusterRoleBindings.

Example:
- Developers (edit access to namespaces)
- Admins (manage specific resources at the cluster level)

---

4. GitHub/GitLab OAuth Authentication

Workflow:
- OpenShift connects with GitHub/GitLab OAuth.
- Users authenticate using GitHub/GitLab credentials.
- Team-based access control by mapping GitHub/GitLab teams to OpenShift roles.

Real-World Use Case:
- Used in DevOps environments where GitHub/GitLab authentication is needed for developers.
- Ideal for organizations using OpenShift GitOps (ArgoCD) and CI/CD pipelines.

Authorization & RBAC:
- Assign roles to GitHub/GitLab teams using RoleBindings.
- Common roles:
  - Developers (edit deployment configs)
  - DevOps engineers (manage CI/CD pipelines)
  - Security teams (read-only access)

---

5. Kerberos Authentication

Workflow:
- OpenShift configured to use Kerberos tickets for authentication.
- Users authenticate using Kerberos credentials instead of passwords.
- Typically used with MIT Kerberos or Active Directory Kerberos authentication.

Real-World Use Case:
- Used in high-security environments (government, financial institutions) requiring strong authentication mechanisms.
- Helps in environments where password authentication is restricted.

Authorization & RBAC:
- Users are mapped to Kerberos-based security groups.
- Assign ClusterRoles to Kerberos-authenticated groups.

Example:
- Developers (limited namespace access)
- Security teams (full audit logs access)

---

6. Token-Based Authentication (Service Accounts & API Access)

Workflow:
- OpenShift generates an API token for authentication.
- Used by service accounts or automation tools (Terraform, Ansible, Jenkins, etc.).
- API tokens provide non-interactive authentication.

Real-World Use Case:
- Used for CI/CD pipelines, Infrastructure-as-Code (IaC) deployments, and automated monitoring systems.
- Ideal for machine-to-machine authentication.

Authorization & RBAC:
- Assign roles to service accounts using RoleBindings.

Example:
- Automation bot (edit access to deployments)
- Monitoring tool (read-only access to logs)

---
### RBAC Levels in OpenShift

OpenShift RBAC (Role-Based Access Control) determines what users and groups can do at different levels:

Cluster-Level RBAC (for global admin roles)
- ClusterRole and ClusterRoleBinding
- Example: Cluster-admin (full cluster control)

Project-Level RBAC (for project-specific access)
- Role and RoleBinding
- Example: Project admin, Developer, Viewer

Resource-Level RBAC (for specific resources)
- Assign access to pods, secrets, deployments, etc.
- Example: Allow only read access to a specific resource.

---

### Validating User & Group Permissions

Default Roles & Assign Permissions to Groups:
```yaml
oc get users # list user
oc get groups # List group
oc get rolebindings -n my-project # Check user roles
oc adm policy who-can get pods -n my-project # Verify permissions for a user

oc adm groups new developers # Create a group
Add users to the group:
oc adm groups add-users developers dev-user
oc adm groups add-users cluster-admins admin-user

oc adm policy add-role-to-group edit developers -n my-app # Assign the edit role to the developers group in a project
oc adm policy add-cluster-role-to-group admin cluster-admins # Assign the admin role to the cluster-admins group
```

To Token-Based Authentication (For Automation & API Access):
```yaml
oc create sa automation-bot -n my-project # Create a service account
oc serviceaccounts get-token automation-bot -n my-project # Get the token
curl -k -H "Authorization: Bearer <TOKEN>" https://api.openshift.example.com:6443/apis # Use the token for API access

oc get identity

oc auth can-i create pods -n my-app --as=dev-user # Verify User Access
oc auth can-i list deployments -n my-app --as-group=developers # Verify Group Access
oc get rolebindings -n my-app # Check All RoleBinding
oc get clusterrolebindings # Check All ClusterRoleBindings
```

Assigning Permissions to Users & Groups at Different Levels with role binding:
```yaml
oc adm policy add-role-to-user admin dev-user -n my-app # Give admin access to a user in a specific project
oc adm policy add-role-to-group edit developers -n my-app # Give edit access to a group in a specific project
oc adm policy add-role-to-user pod-manager dev-user -n my-app # Assign Permissions at the Resource Level

oc adm policy add-role-to-user admin project-admin -n my-app # Grant project-wide admin access to a user
oc adm policy add-cluster-role-to-user cluster-admin admin-user # Grant cluster-admin rights (full control over the cluster)
oc adm policy add-cluster-role-to-group view developers # Grant view access to all projects
```
