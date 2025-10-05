# Online Boutique ArgoCD Application

This directory contains the ArgoCD application configuration for deploying Google's Online Boutique microservices demo.

## Overview

Online Boutique is a cloud-native microservices demo application consisting of:

- **Frontend** - Web UI for the e-commerce site
- **Cart Service** - Shopping cart functionality
- **Checkout Service** - Order processing
- **Currency Service** - Currency conversion
- **Email Service** - Email notifications
- **Payment Service** - Payment processing
- **Product Catalog Service** - Product information
- **Recommendation Service** - Product recommendations
- **Shipping Service** - Shipping calculations
- **Ad Service** - Advertisement serving
- **Redis** - Session storage

## Architecture

The application demonstrates:
- Microservices architecture
- gRPC and HTTP communication
- Service mesh compatibility
- Observability patterns
- Cloud-native deployment

## Configuration

The application is configured with:
- LoadBalancer service for frontend (external access)
- Resource limits for all services
- Automated sync with pruning and self-healing
- Deployment to `onlineboutique` namespace

## Customization

### Service Type Options

**LoadBalancer (Default):**
```yaml
frontend:
  service:
    type: LoadBalancer
```

**NodePort:**
```yaml
frontend:
  service:
    type: NodePort
    nodePort: 30080
```

**ClusterIP with Ingress:**
```yaml
frontend:
  service:
    type: ClusterIP
  ingress:
    enabled: true
    hosts:
      - host: boutique.example.com
        paths:
          - path: /
            pathType: Prefix
```

### Resource Scaling

For larger clusters, increase resources:
```yaml
recommendationService:
  resources:
    requests:
      cpu: 200m
      memory: 440Mi
    limits:
      cpu: 500m
      memory: 900Mi
```

### Disable Services

To disable specific services for testing:
```yaml
adService:
  enabled: false
```

## Deployment

Apply the ArgoCD application:
```bash
kubectl apply -f application.yaml
```

## Access the Application

### Via LoadBalancer
```bash
# Get the external IP
kubectl get service frontend-external -n onlineboutique

# Wait for external IP to be assigned
kubectl wait --for=jsonpath='{.status.loadBalancer.ingress}' service/frontend-external -n onlineboutique
```

### Via Port Forward
```bash
kubectl port-forward -n onlineboutique service/frontend 8080:80
```
Access at: http://localhost:8080

### Via NodePort
```bash
# Get the node IP and port
kubectl get nodes -o wide
kubectl get service frontend-external -n onlineboutique
```

## Monitoring

Check application status:
```bash
kubectl get application onlineboutique -n argocd
```

View all services:
```bash
kubectl get all -n onlineboutique
```

Check service mesh integration:
```bash
kubectl get pods -n onlineboutique -o wide
```

## Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n onlineboutique
kubectl describe pod <pod-name> -n onlineboutique
```

### View Logs
```bash
# Frontend logs
kubectl logs -n onlineboutique deployment/frontend

# All service logs
kubectl logs -n onlineboutique -l app.kubernetes.io/name=onlineboutique
```

### Resource Issues
```bash
# Check resource usage
kubectl top pods -n onlineboutique

# Check resource limits
kubectl describe pods -n onlineboutique
```

### Service Communication
```bash
# Test internal service connectivity
kubectl exec -n onlineboutique deployment/frontend -- wget -qO- http://productcatalogservice:3550/products
```

## Integration with Other Services

### With External DNS
The frontend LoadBalancer will automatically get DNS records if external-dns is configured with the right annotations:

```yaml
frontend:
  service:
    annotations:
      external-dns.alpha.kubernetes.io/hostname: boutique.geekops.dev
```

### With Prometheus Stack
The services expose metrics that can be scraped by Prometheus. Add ServiceMonitor resources:

```yaml
serviceMonitor:
  enabled: true
  labels:
    app: onlineboutique
```

## Performance Notes

- The recommendation service is the most resource-intensive
- Redis is used for session storage and can be scaled
- Services communicate via gRPC for better performance
- Consider using a service mesh for advanced traffic management

## Security Considerations

- Services run as non-root users
- Network policies can be applied for micro-segmentation
- Consider using Pod Security Standards
- External access is only through the frontend service