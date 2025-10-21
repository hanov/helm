# Monitoring Stack Helm Chart

A comprehensive monitoring and alerting stack for Kubernetes featuring Prometheus, Grafana, and Alertmanager.

## Components

- **Prometheus**: Time-series database and monitoring system
- **Grafana**: Metrics visualization and dashboarding
- **Alertmanager**: Alert routing and management
- **Node Exporter**: Hardware and OS metrics exporter
- **Kube State Metrics**: Kubernetes cluster state metrics

## Quick Start

### Installation

```bash
# Install with default values
helm install monitoring ./monitoring-stack

# Install with custom values
helm install monitoring ./monitoring-stack -f custom-values.yaml

# Install in a specific namespace
helm install monitoring ./monitoring-stack --namespace monitoring --create-namespace
```

### Accessing the UIs

#### Grafana (Default)
```bash
# Port forward to access Grafana
kubectl port-forward -n monitoring svc/grafana 3000:3000

# Open http://localhost:3000
# Default credentials: admin/admin (change immediately!)
```

#### Prometheus
```bash
kubectl port-forward -n monitoring svc/prometheus 9090:9090
# Open http://localhost:9090
```

#### Alertmanager
```bash
kubectl port-forward -n monitoring svc/alertmanager 9093:9093
# Open http://localhost:9093
```

## Configuration

### Storage

By default, persistent storage is enabled. To disable:

```yaml
prometheus:
  persistence:
    enabled: false

grafana:
  persistence:
    enabled: false

alertmanager:
  persistence:
    enabled: false
```

To specify a storage class:

```yaml
prometheus:
  persistence:
    storageClass: "fast-ssd"
    size: 100Gi
```

### Resource Limits

Customize resource requests and limits:

```yaml
prometheus:
  resources:
    requests:
      memory: "1Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "4000m"
```

### Alert Rules

Alert rules are configured in `values.yaml`. Example:

```yaml
alertRules:
  enabled: true
  groups:
    - name: custom-alerts
      rules:
        - alert: ServiceDown
          expr: up{job="my-service"} == 0
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "Service is down"
            description: "Service {{ $labels.instance }} is down"
```

### Alertmanager Configuration

Configure notification channels in `values.yaml`:

```yaml
alertmanager:
  config:
    route:
      receiver: 'slack'
      group_by: ['alertname']
    receivers:
      - name: 'slack'
        slack_configs:
          - api_url: 'YOUR_SLACK_WEBHOOK_URL'
            channel: '#alerts'
```

### Grafana Datasources

Prometheus is configured as the default datasource. To add more:

```yaml
grafana:
  datasources:
    prometheus:
      enabled: true
      url: http://prometheus:9090
      isDefault: true
```

## Monitoring Targets

### Automatic Discovery

Prometheus automatically discovers:
- Kubernetes nodes
- Kubernetes pods (with annotation `prometheus.io/scrape: "true"`)
- Kubernetes services
- Kubernetes API servers

### Manual Targets

Add custom scrape configs by modifying the Prometheus ConfigMap template.

## Alert Examples

The chart includes common alert rules:

- **HighCPUUsage**: Triggers when CPU usage > 80%
- **HighMemoryUsage**: Triggers when memory usage > 80%
- **DiskSpaceLow**: Triggers when disk space < 20%
- **PodCrashLooping**: Triggers when pods are restarting
- **PodNotReady**: Triggers when pods are not in Running state

## Upgrading

```bash
helm upgrade monitoring ./monitoring-stack -f values.yaml
```

## Uninstalling

```bash
helm uninstall monitoring -n monitoring

# If you want to delete the namespace
kubectl delete namespace monitoring
```

## Advanced Configuration

### High Availability

To run multiple replicas of Prometheus:

```yaml
prometheus:
  replicas: 2
```

### External Access

Expose Grafana via LoadBalancer:

```yaml
grafana:
  service:
    type: LoadBalancer
```

Or create an Ingress:

```yaml
# Create a separate ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  rules:
  - host: grafana.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
```

### Retention Period

Configure how long Prometheus retains data:

```yaml
prometheus:
  retention: 30d  # Keep data for 30 days
```

## Troubleshooting

### Check Pod Status
```bash
kubectl get pods -n monitoring
kubectl logs -n monitoring <pod-name>
```

### Check PVC Status
```bash
kubectl get pvc -n monitoring
```

### Verify Prometheus Targets
```bash
# Port-forward to Prometheus
kubectl port-forward -n monitoring svc/prometheus 9090:9090

# Navigate to http://localhost:9090/targets
```

### Common Issues

1. **Pods not starting**: Check if PVCs are bound
   ```bash
   kubectl get pvc -n monitoring
   ```

2. **No metrics showing**: Verify ServiceAccount permissions
   ```bash
   kubectl get clusterrolebinding prometheus
   ```

3. **High memory usage**: Reduce retention period or increase limits
   ```yaml
   prometheus:
     retention: 7d
   ```

## Security Considerations

1. **Change default passwords**: Update Grafana admin password immediately
2. **Use secrets**: Store sensitive data in Kubernetes secrets
3. **RBAC**: Review and restrict ServiceAccount permissions as needed
4. **Network policies**: Consider implementing network policies to restrict access

## Values Reference

See `values.yaml` for all available configuration options.

## Contributing

Feel free to submit issues and pull requests.

## License

This chart is provided as-is for use in Kubernetes environments.
