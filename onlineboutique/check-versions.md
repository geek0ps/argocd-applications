# Online Boutique Version Check

## Issue with OCI Registry

The error indicates that ArgoCD cannot resolve the "*" version constraint for the OCI chart. This is common with OCI registries.

## Solutions

### Option 1: Use Specific Version (Updated application.yaml)
I've updated the application.yaml to use version `0.10.0`. If this doesn't work, try these commands to find available versions:

```bash
# Check available versions using Helm
helm show chart oci://us-docker.pkg.dev/online-boutique-ci/charts/onlineboutique

# Or try to pull and inspect
helm pull oci://us-docker.pkg.dev/online-boutique-ci/charts/onlineboutique --version 0.10.0
```

### Option 2: Use Git Repository (application-git.yaml)
I've created an alternative `application-git.yaml` that uses the GitHub repository directly with Kustomize. This is often more reliable.

```bash
kubectl apply -f onlineboutique/application-git.yaml
```

### Option 3: Manual Helm Installation First
Test the chart manually to find the correct version:

```bash
# List available versions
helm search repo onlineboutique --versions

# Or try different versions
helm install onlineboutique oci://us-docker.pkg.dev/online-boutique-ci/charts/onlineboutique \
  --version 0.8.0 \
  --namespace onlineboutique \
  --create-namespace \
  --dry-run
```

## Common Version Numbers to Try

If 0.10.0 doesn't work, try these versions in application.yaml:

- `"0.8.0"`
- `"0.9.0"`
- `"1.0.0"`
- `"latest"`

## Recommended Approach

1. **First, try the updated application.yaml** with version 0.10.0
2. **If that fails, use application-git.yaml** (more reliable)
3. **If you need the Helm chart specifically**, test manually first to find the right version

## Deploy Commands

```bash
# Option 1: Try the fixed OCI version
kubectl apply -f onlineboutique/application.yaml

# Option 2: Use the Git-based version (recommended)
kubectl apply -f onlineboutique/application-git.yaml
```

The Git-based approach is often more reliable and gives you the same microservices demo application.