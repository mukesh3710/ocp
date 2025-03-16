# Deploying Packaged Applications

## OpenShift Templates

**What Are Templates?**
- Kubernetes custom resources that define a set of configurations.
- Include parameters for customization during deployment.

### Benefits of Templates
- **Simplified Deployment**: Easily deploy applications with predefined configurations.
- **Flexibility**: Adapt templates by adjusting parameters.
- **Maintainability**: Centralized management of configuration changes.

### Discovering and Using Templates
- **Discovering Templates**:
  - Cluster Samples Operator provides templates in the `openshift` namespace.
  - List templates: `oc get templates -n openshift`.
  - View template details: `oc describe template <template-name> -n openshift`.

- **Using Templates**:
  1. Deploy directly: `oc new-app --template=<template-name> -p <param-name>=<value>`.
  2. Generate manifests: `oc process <template-name> -p <param-name>=<value> > manifest.yaml`.
  3. Apply manifests: `oc apply -f manifest.yaml`.

### Managing Templates in Production
1. Create a customized copy of the template.
2. Modify default parameter values as needed.
3. (Optional) Remove the `namespace` field.
4. Upload to your project: `oc create -f <template-file>`.

**Notes:**
- Version templates in a source control system.
- For complex deployments, consider Helm charts.
- Use `oc process --param-file` to provide parameters from a file.

---

## Helm Charts

**What Are Helm Charts?**
- Packages for deploying Kubernetes applications.
- Include resource definitions (e.g., deployments, services) and metadata.
- Use Go templates for customization via values.

### Benefits of Helm Charts
- **Simplified Deployments**: Package pre-configured resources.
- **Flexibility**: Override defaults (e.g., image tags) during deployment.
- **Version Control**: Manage different chart versions.
- **Distribution**: Share charts via repositories.

### Helm Chart Structure
- `Chart.yaml`: Metadata (name, version, maintainers).
- `templates/`: Resource definitions (e.g., deployments, services).
- `values.yaml`: Default values for customization.
- Optional hooks: Automate tasks during deployment or upgrades.

### Using Helm Charts
1. **Install Helm**:
   - Deploy a chart: `helm install <release-name> <chart-reference>`.
   - List releases: `helm list`.
   - Rollback release: `helm rollback <release-name> <revision>`.
2. **Customize Deployments**:
   - Override values in `values.yaml` during deployment.
   - Pass a separate YAML file or command-line arguments for overrides.
3. **Manage Repositories**:
   - Add repository: `helm repo add <repo-name> <repo-url>`.
   - Update repository: `helm repo update`.
   - Search charts: `helm search repo <keyword>`.

### Important Notes
- Review chart documentation before installation.
- Test rollbacks before production upgrades.
- Avoid modifying Helm release secrets directly.

---

## Summary

- **Templates**:
  - Use templates for parameterized deployments.
  - Deploy templates with `oc new-app`, `oc process`, and `oc apply`.
  - Customize parameters with `-p` or `--param-file`.

- **Helm Charts**:
  - View chart metadata: `helm show chart <chart-reference>`.
  - View default values: `helm show values <chart-reference>`.
  - Install releases: `helm install <release-name> <chart-reference>`.
  - Manage repositories with `helm repo add` and `helm repo update`.

By leveraging OpenShift templates and Helm charts, you can streamline deployments, improve maintainability, and manage application lifecycles effectively.
