# Troubleshooting Guide

This guide helps diagnose and resolve common issues in the monitoring stack.

## General Troubleshooting Steps

1. Check service status:
```bash
docker compose -f docker-compose.full-stack.yml ps
```

2. View service logs:
```bash
docker compose -f docker-compose.full-stack.yml logs [service-name]
```

3. Verify network connectivity:
```bash
docker network inspect monitoring-network
```

## Common Issues

### Prometheus

#### Unable to Scrape Targets

1. Check target status in Prometheus UI (/targets)
2. Verify network connectivity to target
3. Check target configuration in file_sd
4. Verify firewall rules

Solution:
```bash
# Test target connectivity
curl http://target-host:9100/metrics

# Check Prometheus configuration
docker compose -f docker-compose.full-stack.yml exec prometheus promtool check config /etc/prometheus/prometheus.yml
```

#### High Memory Usage

1. Review retention period
2. Check storage.tsdb settings
3. Monitor TSDB stats

Solution:
- Adjust retention in prometheus.yml
- Consider reducing scrape interval
- Clean up unused metrics

### Grafana

#### Can't Connect to Prometheus

1. Check Prometheus datasource configuration
2. Verify network connectivity
3. Check Prometheus health

Solution:
- Test Prometheus connection in Grafana UI
- Verify prometheus:9090 is accessible from Grafana container
- Check Grafana logs for connection errors

#### Dashboard Loading Issues

1. Check JSON model
2. Verify datasource variables
3. Review panel queries

Solution:
- Reload dashboard
- Clear browser cache
- Check console for JavaScript errors

### AlertManager

#### Alerts Not Firing

1. Check alerting rules
2. Verify AlertManager configuration
3. Check Slack webhook

Solution:
```bash
# Test alerting rules
docker compose -f docker-compose.full-stack.yml exec prometheus promtool check rules /etc/prometheus/alerts.yml

# Verify Slack integration
curl -X POST -H "Content-type: application/json" --data '{"text":"Test message"}' $(cat secrets/alertmanager/slack_webhook_url)
```

### Node Exporter

#### Metrics Not Available

1. Check service status
2. Verify port accessibility
3. Review permissions

Solution:
```bash
# Check Node Exporter status
systemctl status node_exporter

# Test metrics endpoint
curl http://${TARGET_IP}:9100/metrics  # Replace ${TARGET_IP} with the IP of the machine running node_exporter
```

### WMI Exporter

#### Installation Issues

1. Verify MSI installation
2. Check Windows services
3. Test port availability

Solution:
- Reinstall using elevated privileges
- Check Windows Event Viewer
- Verify firewall rules

## Log Collection

### Collecting Logs for Support

```bash
# Create log bundle
./scripts/collect-logs.sh

# View specific service logs
docker compose -f docker-compose.full-stack.yml logs --tail=100 [service-name]
```

## Security Issues

### Rotating Compromised Secrets

1. Generate new secrets
2. Update secret files
3. Restart affected services

```bash
# Update Grafana admin password
echo "new-secure-password" > secrets/grafana_admin_password.txt
docker compose -f docker-compose.full-stack.yml restart grafana
```

## Performance Optimization

### High Resource Usage

1. Monitor container stats
2. Review scrape intervals
3. Check retention settings

```bash
# Monitor container resources
docker stats

# Review Prometheus storage
docker compose -f docker-compose.full-stack.yml exec prometheus promtool tsdb stats /prometheus
```

## Getting Help

If you can't resolve an issue:

1. Collect logs and configurations
2. Check GitHub issues
3. Create a new issue with:
   - Error messages
   - Log outputs
   - Configuration files (sanitized)
   - Steps to reproduce