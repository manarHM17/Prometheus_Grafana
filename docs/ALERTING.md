# Alerting Configuration Guide

This guide covers the setup and configuration of alerting in our monitoring stack.

## AlertManager Setup

### Configuration File
Create `alertmanager.yml`:
```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'email-notifications'

receivers:
- name: 'email-notifications'
  email_configs:
  - to: '${ALERT_EMAIL}'
    from: '${SMTP_FROM}'
    smarthost: '${SMTP_SERVER}:587'
    auth_username: '${SMTP_USER}'
    auth_password: '${SMTP_PASSWORD}'
    send_resolved: true

- name: 'slack-notifications'
  slack_configs:
  - api_url: '${SLACK_WEBHOOK_URL}'
    channel: '#alerts'
    send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

### Docker Configuration
```yaml
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "${BIND_ADDRESS}:9093:9093"
    volumes:
      - ./alertmanager:/etc/alertmanager
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    restart: unless-stopped
```

## Alert Rules

### Prometheus Rules
Create `prometheus/rules/alerts.yml`:
```yaml
groups:
- name: node
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High CPU usage on {{ $labels.instance }}
      description: CPU usage is above 85% for 5 minutes

  - alert: HighMemoryUsage
    expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Cached_bytes + node_memory_Buffers_bytes)) / node_memory_MemTotal_bytes * 100 > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High memory usage on {{ $labels.instance }}
      description: Memory usage is above 85% for 5 minutes

  - alert: DiskSpaceLow
    expr: 100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes) > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Low disk space on {{ $labels.instance }}
      description: Disk usage is above 85% for 5 minutes

- name: blackbox
  rules:
  - alert: EndpointDown
    expr: probe_success == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Endpoint {{ $labels.instance }} down
      description: Endpoint has been down for more than 1 minute

- name: services
  rules:
  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Service {{ $labels.job }} is down
      description: Service has been down for more than 1 minute
```

### Prometheus Configuration
Update `prometheus.yml`:
```yaml
rule_files:
  - "rules/alerts.yml"

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager:9093
```

## Notification Channels

### Email Configuration
1. Configure SMTP settings in `alertmanager.yml`
2. Test email delivery:
```bash
curl -H "Content-Type: application/json" -d '{
  "receiver": "email-notifications",
  "status": "firing",
  "alerts": [{
    "status": "firing",
    "labels": {
      "alertname": "TestAlert",
      "service": "test",
      "severity": "warning"
    },
    "annotations": {
      "summary": "Test alert"
    }
  }]
}' http://localhost:9093/api/v1/alerts
```

### Slack Configuration
1. Create Slack webhook
2. Add webhook URL to `alertmanager.yml`
3. Test Slack notification:
```bash
curl -H "Content-Type: application/json" -d '{
  "receiver": "slack-notifications",
  "status": "firing",
  "alerts": [{
    "status": "firing",
    "labels": {
      "alertname": "TestAlert",
      "service": "test",
      "severity": "warning"
    },
    "annotations": {
      "summary": "Test alert"
    }
  }]
}' http://localhost:9093/api/v1/alerts
```

## Alert Management

### Silencing Alerts
1. Access AlertManager UI (http://${ALERT_MANAGER_IP}:9093)
2. Create silence:
   - Matchers: alertname=~".*"
   - Duration: 1h
   - Comment: Maintenance window

### Alert Grouping
Configure in `alertmanager.yml`:
```yaml
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
```

## Alert Templates

### Email Template
```html
{{ define "email.default.html" }}
<!DOCTYPE html>
<html>
<body>
  <h2>{{ .GroupLabels.alertname }}</h2>
  <p><strong>Status:</strong> {{ .Status }}</p>
  <p><strong>Labels:</strong></p>
  <ul>
    {{ range .Labels.SortedPairs }}
    <li>{{ .Name }}: {{ .Value }}</li>
    {{ end }}
  </ul>
  <p><strong>Annotations:</strong></p>
  <ul>
    {{ range .Annotations.SortedPairs }}
    <li>{{ .Name }}: {{ .Value }}</li>
    {{ end }}
  </ul>
</body>
</html>
{{ end }}
```

### Slack Template
```yaml
slack_configs:
  - channel: '#alerts'
    username: 'AlertManager'
    title: '{{ .GroupLabels.alertname }}'
    text: >-
      {{ range .Alerts }}
      *Alert:* {{ .Annotations.summary }}
      *Description:* {{ .Annotations.description }}
      *Severity:* {{ .Labels.severity }}
      {{ end }}
```

## Best Practices

### Alert Design
1. Clear naming convention
2. Appropriate thresholds
3. Meaningful descriptions
4. Correct severity levels

### Alert Routing
1. Use labels for routing
2. Configure proper intervals
3. Group related alerts
4. Set up inhibition rules

### Maintenance
1. Regular testing
2. Update documentation
3. Review alert history
4. Adjust thresholds

## Troubleshooting

### Common Issues
1. Alert not firing
2. Notification failures
3. Template errors
4. Configuration issues

### Debugging
1. Check AlertManager logs
2. Verify alert expressions
3. Test notification channels
4. Review configuration