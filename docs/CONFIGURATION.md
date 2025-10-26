# Configuration Guide

This guide provides detailed information about configuring each component of the monitoring stack.

## Environment Variables Configuration

The `.env` file contains all configurable parameters. Here's a detailed explanation of each:

```bash
# Core Components Versions
PROMETHEUS_VERSION=v2.47.0
GRAFANA_VERSION=10.1.2
ALERTMANAGER_VERSION=v0.25.0

# Network Configuration
BIND_ADDRESS=${IP_ADDRESS}  # Your server's IP address
PROMETHEUS_PORT=9090
GRAFANA_PORT=3000
ALERTMANAGER_PORT=9093
NODE_EXPORTER_PORT=9100
BLACKBOX_EXPORTER_PORT=9115

# Grafana Settings
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=your_secure_password
GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel

# Retention Settings
PROMETHEUS_RETENTION_TIME=15d
PROMETHEUS_STORAGE_SIZE=50GB
```

## Prometheus Configuration

### Main Configuration (prometheus.yml)
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Configuration examples in prometheus/config/prometheus.yml
```

### Storage Configuration
- Default storage path: `/prometheus`
- Retention configuration in docker-compose.yml:
```yaml
command:
  - '--storage.tsdb.path=/prometheus'
  - '--storage.tsdb.retention.time=${PROMETHEUS_RETENTION_TIME}'
```

## Grafana Configuration

### Data Sources
- Prometheus data source is auto-configured via provisioning
- Additional data sources can be added in `grafana/provisioning/datasources/`

### Dashboard Provisioning
- Place dashboard JSON files in `grafana/provisioning/dashboards/`
- Dashboard configuration in `grafana/provisioning/dashboards/dashboard.yml`

### Environment Variables
```bash
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=your_secure_password
GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel
```

## AlertManager Configuration

### Basic Configuration (alertmanager.yml)
```yaml
global:
  slack_api_url_file: '/etc/alertmanager/slack_webhook_url'
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
```

### Alert Routes
- Default route sends to Slack
- Custom routes can be added for different teams/severities

### Notification Templates
- Located in `alertmanager/templates/`
- Custom templates for different notification types

## Security Configuration

### Network Security
1. Use internal Docker network for inter-service communication
2. Expose only necessary ports
3. Bind to specific IP using BIND_ADDRESS

### Authentication
1. Grafana authentication required
2. Basic auth for endpoints (optional)
3. TLS configuration (recommended for production)

### Secrets Management
1. Use Docker secrets for sensitive data
2. Environment variables for configuration
3. File-based secrets for credentials

## Advanced Configuration

### High Availability
- AlertManager cluster configuration
- Prometheus federation
- Multiple Grafana instances

### Remote Storage
- Configure remote write/read for long-term storage
- Integration with time-series databases

### Service Discovery
- File-based SD configuration
- Kubernetes service discovery (if applicable)
- Cloud provider SD options

## Production Considerations

### Resource Requirements
- Prometheus storage calculation
- Grafana resource allocation
- AlertManager scaling

### Backup Configuration
- Regular configuration backups
- Data retention policies
- Disaster recovery procedures

### Monitoring the Monitors
- Self-monitoring configuration
- Meta-monitoring alerts
- Health check endpoints

## Troubleshooting Configuration

### Common Issues
1. Prometheus target configuration
2. Grafana data source connection
3. AlertManager notification delivery

### Validation Commands
```bash
# Validate Prometheus config
docker-compose exec prometheus promtool check config /etc/prometheus/prometheus.yml

# Test AlertManager config
docker-compose exec alertmanager amtool check-config /etc/alertmanager/alertmanager.yml

# Verify Grafana health
curl http://${IP_ADDRESS}:3000/api/health
```

## Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/prometheus/latest/)
- [Grafana Documentation](https://grafana.com/docs/)
- [AlertManager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)