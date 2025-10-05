# External DNS ArgoCD Application

This directory contains the ArgoCD application configuration for deploying external-dns.

## Overview

External DNS automatically manages DNS records for Kubernetes services and ingresses, making them accessible from outside the cluster.

## Configuration

The application is configured to:
- Use the official external-dns Helm chart
- Deploy to the `external-dns` namespace (auto-created)
- Use AWS as the DNS provider
- Monitor services and ingresses for DNS record creation
- Use automated sync with pruning and self-healing enabled

## Customization

To customize the deployment, modify the `helm.values` section in `application.yaml`:

### Common Configuration Options

```yaml
# Change DNS provider
provider: cloudflare  # aws, azure, gcp, cloudflare, etc.

# Set domain filters
domainFilters:
  - example.com
  - subdomain.example.com

# Configure AWS settings
aws:
  zoneType: public  # or private
  region: us-east-1

# Adjust sync interval
interval: 5m

# Change log level
logLevel: debug  # debug, info, warn, error
```

## Deployment

Apply the ArgoCD application:

```bash
kubectl apply -f application.yaml
```

Or use ArgoCD CLI:

```bash
argocd app create -f application.yaml
```

## Monitoring

Check the application status:

```bash
kubectl get application external-dns -n argocd
```

View external-dns logs:

```bash
kubectl logs -n external-dns deployment/external-dns
```