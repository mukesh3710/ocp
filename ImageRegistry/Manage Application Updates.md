# Manage Application Updates

## Container Image Identity and Tags

### Key Concepts

- **Image Name**: Includes registry, namespace, name, and tag (e.g., `registry.access.redhat.com/ubi8/nginx-120:1-86`).
- **Tag**: References a specific image version:
  - Floating Tag: Points to the latest version (e.g., `latest`). Can lead to unintended updates.
  - Non-Floating Tag: Points to a specific version (e.g., `1-86`). Provides control over updates.
- **SHA Image ID (Digest)**: Unique identifier for a specific image version (e.g., `sha256:1be2006abd...`). Ensures consistency.

### Best Practices
- Use non-floating tags or SHA IDs for predictable deployments.
- Understand the implications of using floating tags.
- Verify local image availability with:
  ```bash
  oc debug node/<node-name>
  crictl images --digests --no-trunc
  ```

### Image Pull Policy
- **IfNotPresent** (Default): Pulls if not available locally.
- **Always**: Always checks registry for updates.
- **Never**: Expects image to be preloaded.

### Image Pruning
- OpenShift automatically removes unused images to conserve space.
- Triggered when filesystem usage exceeds 85% and stops below 80%.

---

## Update Application Image and Settings

### Key Points

Modern applications separate code, configuration, and data. OpenShift supports:
- Configuration maps and secrets for settings.
- Persistent volumes for data.
- Container images for application code.

### Deployment Strategies
- **Rolling Update** (Default):
  - Replaces instances one by one.
  - Suitable for backward-compatible updates.
  - Configure `maxSurge` and `maxUnavailable` for control.
  ```yaml
  spec:
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
  ```
- **Recreate**:
  - Stops all instances before starting new ones.
  - Use for incompatible updates or data migrations.

### Monitoring and Rollbacks
- Pause rollouts for multiple changes:
  ```bash
  oc rollout pause deployment/<name>
  ```
- Roll back to previous versions:
  ```bash
  oc rollout undo deployment/<name>
  ```
- List history:
  ```bash
  oc rollout history deployment/<name>
  ```

---

## Reproducible Deployments with Image Streams

### Benefits
- **Reproducibility**: Ensures intended image version is deployed.
- **Isolation**: Shields from registry changes.
- **Rollback**: Easily revert to previous versions.

### Using Image Streams
1. Create an image stream in the project.
2. Enable local lookup policy:
   ```bash
   oc set image-lookup <image-stream-name>
   ```
3. Reference the image stream tag in the deployment:
   ```yaml
   image: keycloak:20.0
   ```

---

## Automatic Image Updates with Image Change Triggers

### How It Works
- **Image Stream**: Maintains history and versioning.
- **Trigger Configuration**:
  - Reference the image stream tag in the deployment.
  - Use `oc set triggers` to define triggers for updates:
    ```bash
    oc set triggers deployment/<name> --containers=<container-name> --from-image=<image-stream>:<tag>
    ```
- **Automatic Rollout**: Deployments update when the image stream tag changes.

### Best Practices
- Test changes with aliases or separate tags before rolling out.
- Use `oc describe is` to inspect image streams.
- Automate CI/CD pipelines to build and push images to streams.

---

## Summary

- Use non-floating tags or SHA IDs for consistent image selection.
- Select appropriate deployment strategies to minimize downtime.
- Image streams enable stable, reproducible deployments.
- Image change triggers automate updates and simplify image management.

By adhering to these practices, you can ensure reliable and efficient application updates in production environments.

---

# Managing Container Images and Updates in OpenShift

## Inspecting Container Images and Deploying Applications
### Inspect Container Images
1. Inspect the details of a container image:
   ```bash
   oc describe imagestream <image-stream-name>
   ```
2. List container images running on compute nodes:
   ```bash
   oc get pods -o jsonpath='{.items[*].spec.containers[*].image}'
   ```

### Deploy Applications Using Image Tags or SHA IDs
Deploy an application using a specific image tag or SHA ID:
```bash
oc new-app <image-name>:<tag>
```
Or specify the SHA ID:
```bash
oc new-app <image-name>@<sha-id>
```

## Updating Application Images and Settings
### Pause, Update, and Resume a Deployment
1. Pause the deployment:
   ```bash
   oc rollout pause deployment/<deployment-name>
   ```
2. Update the deployment image:
   ```bash
   oc set image deployment/<deployment-name> <container-name>=<new-image>
   ```
3. Resume the deployment:
   ```bash
   oc rollout resume deployment/<deployment-name>
   ```

### Roll Back a Deployment
1. Check the rollout history:
   ```bash
   oc rollout history deployment/<deployment-name>
   ```
2. Roll back to a previous revision:
   ```bash
   oc rollout undo deployment/<deployment-name> --to-revision=<revision-number>
   ```

## Reproducible Deployments with Image Streams
### Create Image Streams and Image Stream Tags
1. Create an image stream:
   ```bash
   oc create imagestream <image-stream-name>
   ```
2. Add a tag to the image stream:
   ```bash
   oc tag <image-stream>:<tag-name> <new-image>:<tag>
   ```
3. Deploy an application using the image stream tag:
   ```bash
   oc new-app <image-stream>:<tag>
   ```

## Automatic Image Updates with Image Change Triggers
### Add an Image Trigger to a Deployment
1. Add an image trigger to a deployment:
   ```bash
   oc set triggers deployment/<deployment-name> --containers=<container-name> --from-image=<image-stream>:<tag>
   ```
2. Verify the trigger:
   ```bash
   oc describe deployment/<deployment-name>
   ```

### Modify an Image Stream Tag
1. Point an image stream tag to a new image:
   ```bash
   oc tag <new-image>:<tag> <image-stream>:<tag>
   ```
2. Watch the application rollout:
   ```bash
   oc rollout status deployment/<deployment-name>
   ```

### Roll Back a Deployment to a Previous Image
1. Revert the image stream tag to the previous image:
   ```bash
   oc tag <previous-image>:<tag> <image-stream>:<tag>
   ```
2. Monitor the application rollout:
   ```bash
   oc rollout status deployment/<deployment-name>
   ```

## Managing Application Updates
### Reconfigure Deployment to Use Static Tags
1. Update the deployment to use a static tag instead of `latest`:
   ```bash
   oc set image deployment/<deployment-name> <container-name>=<image>:<static-tag>
   ```
2. Verify the updated deployment:
   ```bash
   oc get deployment/<deployment-name> -o yaml
   ```

### Enable Image Triggering
1. Enable image triggering for the `app2` deployment:
   ```bash
   oc set triggers deployment/app2 --containers=app2 --from-image=php-ssl:1
   ```

### Create and Assign Image Stream Tags
1. Create a new image stream tag:
   ```bash
   oc tag <new-image>:<tag> php-ssl:1-234
   ```
2. Move the `php-ssl:1` alias to the new image stream tag:
   ```bash
   oc tag php-ssl:1-234 php-ssl:1 --alias
   ```
3. Verify the redeployment of `app2`:
   ```bash
   oc rollout status deployment/app2
   ```

