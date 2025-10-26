# Operational Runbook: Prometheus & Grafana Stack

## Overview
This runbook provides step-by-step procedures for installing, operating, and maintaining the Prometheus and Grafana monitoring stack using Docker.

### Important Note
Throughout this document, replace `${SERVER_IP}` with your actual server's IP address. For example, if your server's IP is 192.168.1.100, you would use:
```bash
# Example with actual IP
curl http://192.168.1.100:9090/-/healthy
```

You can set this as an environment variable for convenience:
```bash
export SERVER_IP="your.server.ip.address"
# Then use as shown in the commands below
```

## 1. Prerequisites Installation

### 1.1 Docker Installation (Ubuntu)
```bash
# Update package index
sudo apt-get update

# Install dependencies
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Add current user to docker group
sudo usermod -aG docker $USER
```

### 1.2 System Requirements
- Minimum 4GB RAM
- 2 CPU cores
- 50GB disk space
- Ports required:
  - 9090 (Prometheus)
  - 3000 (Grafana)
  - 9093 (AlertManager)
  - 9100 (Node Exporter)
  - 9115 (Blackbox Exporter)

## 2. Initial Setup

### 2.1 Create Directory Structure
```bash
# Create base directory
mkdir -p monitoring-stack
cd monitoring-stack

# Create required directories
mkdir -p {prometheus,grafana,alertmanager}/config
mkdir -p data/{prometheus,grafana,alertmanager}
mkdir -p secrets
```

### 2.2 Set Up Secrets
```bash
# Create secrets directory with restricted permissions
mkdir -p secrets
chmod 700 secrets

# Create secret files
echo "your-grafana-admin-password" > secrets/grafana_admin_password
echo "https://hooks.slack.com/services/YOUR/WEBHOOK/URL" > secrets/slack_webhook_url
chmod 600 secrets/*
```

### 2.3 Configure Environment
```bash
# Copy example environment file
cp .env.example .env

# Edit environment variables
nano .env
```

## 3. Component Installation

### 3.1 Deploy Stack
```bash
# Pull images
docker-compose -f docker-compose.full-stack.yml pull

# Start services
docker-compose -f docker-compose.full-stack.yml up -d
```

### 3.2 Verify Installation
```bash
# Check service status
docker-compose -f docker-compose.full-stack.yml ps

# Check Prometheus
curl http://${SERVER_IP}:9090/-/healthy

# Check Grafana
curl http://${SERVER_IP}:3000/api/health

# Check AlertManager
curl http://${SERVER_IP}:9093/-/healthy
```

## 4. Post-Installation Configuration

### 4.1 Configure Prometheus Targets
1. Edit target files in `prometheus/file_sd/`:
```yaml
- targets:
  - 'node-exporter:9100'
  labels:
    env: 'production'
```

### 4.2 Configure Grafana
1. Access Grafana at http://${SERVER_IP}:3000
2. Default login: admin / (password from secrets)
3. Import dashboards:
   - Node Exporter Full (ID: 1860)
   - Prometheus 2.0 (ID: 3662)

### 4.3 Verify Alerts
1. Access AlertManager at http://${SERVER_IP}:9093
2. Check Slack integration:
```bash
# Test Slack notification
curl -X POST -H "Content-type: application/json" \
  --data '{"status":"firing","alerts":[{"status":"firing","labels":{"alertname":"TestAlert","severity":"critical"},"annotations":{"description":"This is a test alert"}}]}' \
  http://${SERVER_IP}:9093/api/v1/alerts
```

## 5. Maintenance Procedures

### 5.1 Backup Procedures
```bash
# Backup configurations
./scripts/backup-config.sh

# Backup data volumes
./scripts/backup-data.sh
```

### 5.2 Update Procedures
```bash
# Pull new images
docker-compose -f docker-compose.full-stack.yml pull

# Update services
docker-compose -f docker-compose.full-stack.yml up -d

# Verify update
docker-compose -f docker-compose.full-stack.yml ps
```

### 5.3 Scaling
- Prometheus storage scaling:
  1. Update retention in prometheus.yml
  2. Restart Prometheus

### 5.4 Log Management
```bash
# View logs
docker-compose -f docker-compose.full-stack.yml logs -f [service_name]

# Rotate logs
docker-compose -f docker-compose.full-stack.yml exec prometheus promtool tsdb clean
```

## 6. Troubleshooting

### 6.1 Common Issues
1. Prometheus not scraping targets:
```bash
# Check target status
curl http://${SERVER_IP}:9090/api/v1/targets

# Validate config
docker-compose -f docker-compose.full-stack.yml exec prometheus promtool check config /etc/prometheus/prometheus.yml
```

2. Grafana can't connect to Prometheus:
```bash
# Check Prometheus connection
curl http://prometheus:9090/api/v1/query?query=up

# Verify network
docker network inspect monitoring-network
```

### 6.2 Recovery Procedures
```bash
# Restore from backup
./scripts/restore-backup.sh [backup_date]

# Reset to last known good configuration
git checkout [last_working_commit]
docker-compose -f docker-compose.full-stack.yml up -d
```

## 7. Security Procedures

### 7.1 Secret Rotation
```bash
# Rotate Grafana admin password
echo "new-secure-password" > secrets/grafana_admin_password
docker-compose -f docker-compose.full-stack.yml restart grafana

# Rotate Slack webhook
echo "new-webhook-url" > secrets/slack_webhook_url
docker-compose -f docker-compose.full-stack.yml restart alertmanager
```

### 7.2 Security Checks
```bash
# Check container security
docker scan prometheus
docker scan grafana

# Check exposed ports
sudo netstat -tulpn | grep -E '9090|3000|9093'
```

## 8. Contact Information

- Primary Contact: [Your Name] (email/slack)
- Secondary Contact: [Backup Name] (email/slack)
- Slack Channel: #monitoring-alerts

## 9. Related Documents

- [Architecture Documentation](ARCHITECTURE.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)
- [Security Guidelines](SECURITY.md)
- [Backup and Recovery Procedures](BACKUP.md)