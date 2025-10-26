# Grafana Dashboards Guide

This guide provides information about available dashboards and how to configure them.

## Dashboard Overview

### Node Exporter Dashboard
- Purpose: Monitor Linux system metrics
- Data Source: Prometheus
- Panels:
  - CPU Usage
  - Memory Usage
  - Disk I/O
  - Network Traffic
  - System Load
  - File System Usage

### Windows Exporter Dashboard
- Purpose: Monitor Windows system metrics
- Data Source: Prometheus
- Panels:
  - CPU Performance
  - Memory Utilization
  - Disk Activity
  - Network Interface
  - Service Status
  - Event Logs

### Application Dashboard
- Purpose: Monitor application health and performance
- Data Source: Prometheus
- Panels:
  - Response Time
  - Error Rate
  - Request Count
  - Active Users
  - Resource Usage

### Alert Overview Dashboard
- Purpose: View and manage alerts
- Data Source: Prometheus
- Panels:
  - Active Alerts
  - Alert History
  - Alert Groups
  - Silenced Alerts

## Dashboard Installation

### Using Grafana UI
1. Click '+' icon > Import
2. Enter dashboard ID or upload JSON file
3. Select data source
4. Click Import

### Using API
```bash
curl -X POST -H "Content-Type: application/json" \
  -d @dashboard.json \
  http://${GRAFANA_IP}:3000/api/dashboards/db
```

## Dashboard Configuration

### Data Sources
1. Add Prometheus data source:
   - URL: http://prometheus:9090
   - Access: Server (default)
   - Scrape interval: 15s

### Variables
Common dashboard variables:
```json
{
  "instance": {
    "type": "query",
    "query": "label_values(node_uname_info, instance)",
    "refresh": 2
  },
  "job": {
    "type": "query",
    "query": "label_values(job)",
    "refresh": 2
  }
}
```

### Annotations
Example annotation configuration:
```json
{
  "alerting": {
    "tags": ["alerting"],
    "enable": true,
    "query": "ALERTS{alertstate=\"firing\"}"
  }
}
```

## Custom Dashboards

### Creating New Dashboards
1. Start with empty dashboard
2. Add panels
3. Configure data sources
4. Set variables
5. Add annotations
6. Save dashboard

### Panel Types
1. Graph
2. Gauge
3. Stat
4. Table
5. Heatmap
6. Bar Gauge

### Query Examples

#### CPU Usage
```
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

#### Memory Usage
```
100 * (1 - ((avg_over_time(node_memory_MemFree_bytes[5m]) + avg_over_time(node_memory_Cached_bytes[5m]) + avg_over_time(node_memory_Buffers_bytes[5m])) / avg_over_time(node_memory_MemTotal_bytes[5m])))
```

#### Disk Usage
```
100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)
```

## Dashboard Export/Import

### Export Dashboard
1. Dashboard settings
2. JSON Model
3. Copy JSON or download file

### Import Dashboard
1. Dashboard import
2. Upload JSON file
3. Configure variables
4. Select data source

## Dashboard Organization

### Folder Structure
- System Monitoring
  - Node Exporter
  - Windows Exporter
- Application Monitoring
  - Services
  - APIs
- Infrastructure
  - Network
  - Storage
- Alerts
  - Overview
  - History

### Permissions
1. Viewer: Can view dashboards
2. Editor: Can edit dashboards
3. Admin: Full access

## Best Practices

### Dashboard Design
1. Consistent naming
2. Clear descriptions
3. Appropriate time ranges
4. Useful variables
5. Meaningful units

### Performance
1. Optimize queries
2. Use appropriate intervals
3. Limit panel count
4. Cache results

### Maintenance
1. Regular updates
2. Version control
3. Documentation
4. Backup/restore

## Troubleshooting

### Common Issues
1. No data
2. Query errors
3. Variable problems
4. Permission issues

### Solutions
1. Check data source
2. Verify metrics
3. Review permissions
4. Check variables
5. Validate JSON