# External DNS Troubleshooting Guide

## Common Issues in Kubeadm Clusters

### 1. Immediate Pod Deletion/Restart Loop

**Symptoms:** Pod gets created then immediately deleted or crashes

**Causes & Solutions:**

#### A. Missing AWS Credentials
```bash
# Check if external-dns pod has AWS credentials
kubectl logs -n external-dns deployment/external-dns

# Look for errors like:
# "NoCredentialProviders: no valid providers in chain"
# "UnauthorizedOperation: You are not authorized to perform this operation"
```

**Solution:** Create AWS credentials secret:
```bash
# Method 1: Create secret with your AWS credentials
kubectl create secret generic external-dns-aws-credentials \
  --from-literal=aws-access-key-id=YOUR_ACCESS_KEY \
  --from-literal=aws-secret-access-key=YOUR_SECRET_KEY \
  -n external-dns

# Method 2: Apply the secret template (edit aws-credentials-secret.yaml first)
kubectl apply -f aws-credentials-secret.yaml
```

Then uncomment the env section in application.yaml.

#### B. RBAC Issues
```bash
# Check if service account has proper permissions
kubectl auth can-i list services --as=system:serviceaccount:external-dns:external-dns
kubectl auth can-i list ingresses --as=system:serviceaccount:external-dns:external-dns
kubectl auth can-i list nodes --as=system:serviceaccount:external-dns:external-dns
```

**Solution:** The updated application.yaml includes enhanced RBAC permissions.

#### C. Resource Limits
```bash
# Check if pod is being OOM killed
kubectl describe pod -n external-dns -l app.kubernetes.io/name=external-dns
```

**Solution:** The updated configuration includes resource limits.

### 2. DNS Provider Issues

#### AWS Route53 Setup
1. **IAM Policy Required:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53:ListHostedZones",
                "route53:ListResourceRecordSets"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

2. **Domain Filters:**
Update application.yaml to include your domains:
```yaml
domainFilters:
  - example.com
  - subdomain.example.com
```

### 3. Alternative: Use Different DNS Provider

If AWS is causing issues, try a simpler provider like Cloudflare:

```yaml
provider: cloudflare
env:
  - name: CF_API_TOKEN
    valueFrom:
      secretKeyRef:
        name: cloudflare-api-token
        key: api-token
```

### 4. Debugging Commands

```bash
# Check application status in ArgoCD
kubectl get application external-dns -n argocd -o yaml

# Check external-dns deployment
kubectl get deployment -n external-dns

# Check pods and their status
kubectl get pods -n external-dns

# View detailed pod information
kubectl describe pod -n external-dns -l app.kubernetes.io/name=external-dns

# Check logs (most important for debugging)
kubectl logs -n external-dns deployment/external-dns -f

# Check service account and RBAC
kubectl get serviceaccount -n external-dns
kubectl get clusterrole external-dns
kubectl get clusterrolebinding external-dns

# Check if namespace exists and has proper labels
kubectl get namespace external-dns
```

### 5. Step-by-Step Debugging Process

1. **Check ArgoCD Application:**
```bash
kubectl get application external-dns -n argocd
kubectl describe application external-dns -n argocd
```

2. **Check if namespace is created:**
```bash
kubectl get namespace external-dns
```

3. **Check if deployment is created:**
```bash
kubectl get deployment -n external-dns
```

4. **Check pod status:**
```bash
kubectl get pods -n external-dns
kubectl describe pod -n external-dns -l app.kubernetes.io/name=external-dns
```

5. **Check logs for specific errors:**
```bash
kubectl logs -n external-dns deployment/external-dns --previous
```

### 6. Quick Fix for Testing

For immediate testing without AWS setup, you can use the dry-run mode:

```yaml
policy: sync  # Change from upsert-only
dryRun: true  # Add this line for testing
```

This will show what external-dns would do without actually making changes.

## Next Steps

1. Apply the updated application.yaml
2. If using AWS, create the credentials secret
3. Check the logs using the debugging commands above
4. Share the specific error messages for further assistance