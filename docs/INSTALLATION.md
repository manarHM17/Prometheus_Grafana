# Complete Installation Runbook

## 1. Ubuntu Server Setup

### 1.1 Ubuntu Server Installation
1. Download Ubuntu Server 22.04 LTS
2. Create VM with minimum specifications:
   - 4GB RAM
   - 2 CPU cores
   - 50GB disk space
   - Network adapter in bridge mode

3. Install Ubuntu Server:
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    git
```

### 1.2 Network Configuration
```bash
# Set static IP (edit netplan configuration)
sudo nano /etc/netplan/00-installer-config.yaml
```

Example netplan configuration:
```yaml
network:
  ethernets:
    ens33:
      addresses:
        - ${IP_ADDRESS}/24
      gateway4: ${GATEWAY_IP}
      nameservers:
          addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

Apply network configuration:
```bash
sudo netplan apply
```

## 2. Docker Installation

### 2.1 Install Docker Engine
```bash
# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### 2.2 Install Docker Compose
```bash
# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 3. Monitoring Stack Setup

### 3.1 Clone Repository
```bash
git clone https://github.com/manarHM17/Prometheus_Grafana.git
cd Prometheus_Grafana
```

### 3.2 Create Directory Structure
```bash
# Create necessary directories
mkdir -p {prometheus,grafana,alertmanager}/{config,data}
mkdir -p prometheus/rules
mkdir -p grafana/provisioning/{dashboards,datasources}
```

### 3.3 Configure Environment Variables
```bash
cp .env.example .env
```

Edit .env file with your configurations:
```env
# Core Services
PROMETHEUS_VERSION=v2.47.0
GRAFANA_VERSION=10.1.2
ALERTMANAGER_VERSION=v0.25.0

# Ports
PROMETHEUS_PORT=9090
GRAFANA_PORT=3000
ALERTMANAGER_PORT=9093
NODE_EXPORTER_PORT=9100
BLACKBOX_EXPORTER_PORT=9115

# Credentials
GF_SECURITY_ADMIN_PASSWORD=your_secure_password
GF_SECURITY_ADMIN_USER=admin

# Network
DOCKER_NETWORK=monitoring
BIND_ADDRESS=${IP_ADDRESS}
```

## 4. Configuration Files

### 4.1 Docker Compose Configuration
Create docker-compose.yml:

```yaml
version: '3.8'

networks:
  monitoring:
    name: ${DOCKER_NETWORK}

volumes:
  prometheus_data:
  grafana_data:
  alertmanager_data:

services:
  prometheus:
    image: prom/prometheus:${PROMETHEUS_VERSION}
    container_name: prometheus
    volumes:
      - ./prometheus/config:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - "${BIND_ADDRESS}:${PROMETHEUS_PORT}:9090"
    networks:
      - monitoring
    restart: unless-stopped

  grafana:
    image: grafana/grafana:${GRAFANA_VERSION}
    container_name: grafana
    volumes:
      - ./grafana/provisioning:/etc/grafana/provisioning
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=${GF_SECURITY_ADMIN_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GF_SECURITY_ADMIN_PASSWORD}
    ports:
      - "${BIND_ADDRESS}:${GRAFANA_PORT}:3000"
    networks:
      - monitoring
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:${ALERTMANAGER_VERSION}
    container_name: alertmanager
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    ports:
      - "${BIND_ADDRESS}:${ALERTMANAGER_PORT}:9093"
    networks:
      - monitoring
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "${BIND_ADDRESS}:${NODE_EXPORTER_PORT}:9100"
    networks:
      - monitoring
    restart: unless-stopped

  blackbox-exporter:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-exporter
    volumes:
      - ./blackbox/config:/config
    command:
      - '--config.file=/config/blackbox.yml'
    ports:
      - "${BIND_ADDRESS}:${BLACKBOX_EXPORTER_PORT}:9115"
    networks:
      - monitoring
    restart: unless-stopped
```

### 4.2 Prometheus Configuration
Create prometheus/config/prometheus.yml:

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
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'windows'
    static_configs:
      - targets:
        - '${WINDOWS_IP}:9182'  # WMI Exporter

  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - 'http://${APP_IP}'    # Application endpoints
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

### 4.3 AlertManager Configuration
Create alertmanager/config/alertmanager.yml:

```yaml
global:
  slack_api_url_file: '/etc/alertmanager/slack_webhook_url'

route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#monitoring-alerts'
        send_resolved: true
```

## 5. Start Services

```bash
# Start all services
docker-compose up -d

# Verify services are running
docker-compose ps

# Check logs if needed
docker-compose logs -f [service_name]
```

## 6. Configure Grafana

### 6.1 Add Prometheus Data Source
Create grafana/provisioning/datasources/prometheus.yml:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

### 6.2 Import Dashboards
1. Access Grafana (http://${IP_ADDRESS}:3000)
2. Log in with configured credentials
3. Import recommended dashboards:
   - Node Exporter Full (ID: 1860)
   - Windows Exporter (ID: 14694)
   - Blackbox Exporter (ID: 7587)

## 7. Verify Installation

1. Check Prometheus targets:
   - Visit http://${IP_ADDRESS}:9090/targets
   - All targets should show as "UP"

2. Check Grafana dashboards:
   - Verify metrics are being displayed
   - Check time range data availability

3. Test AlertManager:
   - Verify Slack integration
   - Test alert notification delivery

## 8. Maintenance

### Backup Configuration
```bash
# Backup all configuration files
tar -czf monitoring-backup-$(date +%F).tar.gz \
    prometheus/config \
    grafana/provisioning \
    alertmanager/config
```

### Update Services
```bash
# Pull latest images
docker-compose pull

# Restart services
docker-compose up -d
```

## 9. Troubleshooting

### Common Issues

1. **Prometheus Can't Scrape Targets**
   ```bash
   # Check Prometheus configuration
   docker-compose exec prometheus promtool check config /etc/prometheus/prometheus.yml
   ```

2. **Grafana Can't Connect to Prometheus**
   ```bash
   # Verify network connectivity
   docker network inspect monitoring
   ```

3. **Node Exporter Access Issues**
   ```bash
   # Check port accessibility
   curl http://${IP_ADDRESS}:9100/metrics
   ```

For more troubleshooting information, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md)