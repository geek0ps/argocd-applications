# Prometheus Stack ArgoCD Application

This directory contains the ArgoCD application configuration for deploying the kube-prometheus-stack.

## Overview

The kube-prometheus-stack includes:
- **Prometheus** - Metrics collection and storage
- **Grafana** - Visualization and dashboards
- **Alertmanager** - Alert handling and routing
- **Node Exporter** - Hardware and OS metrics
- **Kube State Metrics** - Kubernetes object metrics
- **Prometheus Operator** - Manages Prometheus instances

## Configuration

The application is configured with:
- 30-day retention for Prometheus metrics
- Persistent storage for all components
- Resource limits and requests set
- All default alerting rules enabled
- Automated sync with pruning and self-healing

## Storage Requirements

- **Prometheus**: 50Gi (configurable based on cluster size)
- **Grafana**: 10Gi
- **Alertmanager**: 10Gi

## Default Credentials

- **Grafana**: admin/admin (change after first login)

## Customization

### Storage Classes
Update storage class names if not using `gp2`:

```yaml
prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: your-storage-class
```

### Resource Allocation
Adjust based on cluster size:

```yaml
prometheus:
  prometheusSpec:
    resources:
      requests:
        memory: 4Gi  # Increase for larger clusters
        cpu: 2000m
```

### Enable Ingress for Grafana
```yaml
grafana:
  ingress:
    enabled: true
    hosts:
      - grafana.example.com
    tls:
      - secretName: grafana-tls
        hosts:
          - grafana.example.com
```

### Custom Alerting Rules
Add custom PrometheusRule resources:

```yaml
additionalPrometheusRulesMap:
  custom-rules:
    groups:
    - name: custom.rules
      rules:
      - alert: HighCPUUsage
        expr: cpu_usage > 80
        for: 5m
```

## Deployment

Apply the ArgoCD application:

```bash
kubectl apply -f application.yaml
```

## Access Services

### Port Forward to Grafana
```bash
kubectl port-forward -n monitoring svc/prom-stack-grafana 3000:80
```
Access at: http://localhost:3000

### Port Forward to Prometheus
```bash
kubectl port-forward -n monitoring svc/prom-stack-kube-prom-prometheus 9090:9090
```
Access at: http://localhost:9090

### Port Forward to Alertmanager
```bash
kubectl port-forward -n monitoring svc/prom-stack-kube-prom-alertmanager 9093:9093
```
Access at: http://localhost:9093

## Monitoring

Check application status:
```bash
kubectl get application prom-stack -n argocd
```

View all monitoring resources:
```bash
kubectl get all -n monitoring
```

Check Prometheus targets:
```bash
kubectl get servicemonitor -n monitoring
```

## Troubleshooting

### Check Prometheus Configuration
```bash
kubectl get prometheus -n monitoring -o yaml
```

### View Operator Logs
```bash
kubectl logs -n monitoring deployment/prom-stack-kube-prom-operator
```

### Check Storage
```bash
kubectl get pvc -n monitoring
```