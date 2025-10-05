# Verify IAM Role for External DNS

Since you have an IAM role attached to your nodes, external-dns should work automatically. Here's how to verify and troubleshoot:

## 1. Check Node IAM Role Permissions

Your node's IAM role needs these Route53 permissions:

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

## 2. Test IAM Role from a Pod

Create a test pod to verify the role works:

```bash
kubectl run aws-cli-test --image=amazon/aws-cli:latest -it --rm --restart=Never -- /bin/bash
```

Inside the pod, test AWS access:
```bash
aws sts get-caller-identity
aws route53 list-hosted-zones
```

## 3. Check External DNS Logs

After deploying, check the logs for authentication issues:

```bash
kubectl logs -n external-dns deployment/external-dns -f
```

Look for:
- ✅ `Successfully authenticated with AWS`
- ✅ `Found hosted zones: [...]`
- ❌ `NoCredentialProviders: no valid providers in chain`
- ❌ `UnauthorizedOperation`

## 4. Common Issues with Node IAM Roles

### Issue: Pod can't assume the role
**Solution:** Ensure the node's IAM role has the necessary Route53 permissions attached.

### Issue: Wrong AWS region
**Solution:** Set the AWS region in the application.yaml:
```yaml
env:
  - name: AWS_DEFAULT_REGION
    value: "your-region"
```

### Issue: No hosted zones found
**Solution:** Make sure you have Route53 hosted zones and set domain filters:
```yaml
domainFilters:
  - your-domain.com
```

## 5. Quick Deployment Test

1. Apply the application:
```bash
kubectl apply -f application.yaml
```

2. Wait for deployment:
```bash
kubectl wait --for=condition=available --timeout=300s deployment/external-dns -n external-dns
```

3. Check logs immediately:
```bash
kubectl logs -n external-dns deployment/external-dns
```

4. Look for successful initialization messages like:
```
time="..." level=info msg="Connected to cluster at https://kubernetes.default.svc"
time="..." level=info msg="Found hosted zones: [Z1234567890ABC example.com.]"
```

## 6. If Still Having Issues

The most common remaining issues are:

1. **Domain filters not set** - Add your actual domains to `domainFilters`
2. **Wrong AWS region** - Set the correct region in env vars
3. **IAM role missing Route53 permissions** - Attach the Route53 policy to your node role

Let me know what you see in the logs and I can help debug further!