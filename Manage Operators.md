# Manage Kubernetes Operators

## Kubernetes Operators and Operator Lifecycle Manager (OLM)

### Operator Pattern
- **Complex Workloads**: Operators manage complex applications with multiple components and automated tasks (e.g., databases, messaging systems).
- **Custom Resources (CRs)**: Define the desired state of the application.
- **Operator Workload**: Watches for CR changes and creates/updates Kubernetes resources to achieve the desired state.

### Operator Deployment and Management
1. **Cluster Operators**: Core platform services managed by the Cluster Version Operator (CVO).
2. **Add-on Operators**: Installed and managed using the Operator Lifecycle Manager (OLM).
3. **Custom Operators**: Developed using frameworks like Operator SDK or Java Operator SDK.

### Operator Lifecycle Manager (OLM)
- Manages the lifecycle of add-on operators.
- Uses operator catalogs to discover and install operators.
- Provides a user-friendly interface for operator management in the web console.

#### Key Concepts
- **PackageManifest**: Metadata and configuration for an operator.
- **Subscription**: Installs an operator from a specific channel in a catalog.
- **CatalogSource**: Points to a catalog of operators (e.g., Red Hat, Certified, Community, Marketplace).

### Operator Development
- Use **Operator SDK** for Go-based operators.
- Use **Java Operator SDK** for Java-based operators.
- Leverage frameworks like **Quarkus** for efficient development.

### Benefits of Operators
- **Automation**: Simplify complex application management tasks.
- **Consistency**: Ensure consistent deployment and configuration.
- **Self-Healing**: Automatically recover from failures.
- **Extensibility**: Create custom operators for specific needs.

---

## Installing and Managing Operators

### Using the Web Console
#### OperatorHub
- Discover and install operators from various sources (Red Hat, Certified, Community, Marketplace).
- Filter operators by category, source, or provider.
- Review operator details and documentation before installation.

#### Install Operator Wizard
- **Update Channel**: Choose the desired update channel.
- **Installation Mode**: Select "All namespaces" or "Installed namespace".
- **Update Approval**: Configure updates as automatic or manual.
- **Monitoring**: Enable monitoring for supported operators.

#### Installed Operators
- View operator status, details, subscriptions, and custom resource instances.
- Manage updates and uninstallations.

#### Using Operators
- Create custom resource instances via the web console’s form or YAML editors.
- Consult operator documentation for specific configurations.

#### Troubleshooting Operators
- Check statuses of CSV, subscription, and install plan resources.
- Review operator workloads (deployments, stateful sets).
- Analyze pod logs and events.

### Tips
- Always review operator documentation before installation.
- Use monitoring tools to track operator health.

---

### Using the CLI
#### Examining Available Operators
- `oc get catalogsource`: List available catalog sources.
- `oc get packagemanifests`: List available operators.
- `oc describe packagemanifest`: View operator details (channels, install modes).

#### Installing Operators
1. Review operator documentation.
2. Create required namespaces or operator groups.
3. Create a subscription specifying:
   - Channel (update path).
   - Package manifest.
   - Source catalog.
   - Install plan approval mode (automatic/manual).

#### Managing Resources
- **Install Plan**: Represents the installation/update process.
- **Operator Resource**: Operator-specific configuration.
- **Cluster Service Version (CSV)**: Defines operator version and installation details.
- View resources with `oc describe` commands.

#### Using Operators
- Operators create custom resource definitions.
- Use `oc get csv` to list associated custom resource definitions.
- Use `oc explain` to view details of custom resources.

#### Troubleshooting Operators
- Review operator documentation for troubleshooting steps.
- Examine statuses of operator, install plan, and CSV resources.
- Debug operator workloads and associated custom resource workloads.

### Additional Tips
- Leverage the CLI for detailed exploration and management.
- Understand subscription options and resource structure for efficient management.

---

## Summary
- Operators extend Kubernetes functionality, automating complex workloads.
- Cluster operators provide core platform services like the OpenShift web console.
- The Operator Lifecycle Manager (OLM) simplifies add-on operator management.
- Operators use custom resources to manage applications declaratively.
- Users can manage operators via the web console, CLI, or API.
- Use tools like package manifests, subscriptions, and install plans for detailed management.

By mastering operator patterns, OLM, and CLI/web management tools, you can effectively automate and streamline Kubernetes application management.
