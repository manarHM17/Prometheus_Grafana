# Exporters Setup Guide

This guide covers the setup and configuration of various exporters used in our monitoring stack.

## Node Exporter (Linux Systems)

### Docker Installation
```bash
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

### Systemd Installation
1. Create user:
```bash
sudo useradd -rs /bin/false node_exporter
```

2. Download and install:
```bash
VERSION="1.6.1"
wget https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz
tar xvf node_exporter-${VERSION}.linux-amd64.tar.gz
sudo mv node_exporter-${VERSION}.linux-amd64/node_exporter /usr/local/bin/
```

3. Create systemd service:
```bash
sudo tee /etc/systemd/system/node_exporter.service <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter --collector.systemd

[Install]
WantedBy=multi-user.target
EOF
```

4. Start service:
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

### Verification
```bash
curl http://localhost:9100/metrics
```

## Windows Exporter

### Installation Options

#### MSI Installer
1. Download latest release from [Windows Exporter Releases](https://github.com/prometheus-community/windows_exporter/releases)
2. Run MSI installer:
```powershell
msiexec /i windows_exporter.msi ENABLED_COLLECTORS="cpu,memory,disk,network"
```

#### Docker Installation (Windows Containers)
```powershell
docker run -d --name windows-exporter `
  --restart unless-stopped `
  -p 9182:9182 `
  ghcr.io/prometheus-community/windows-exporter:latest
```

### Configuration
1. Enable required collectors in `windows_exporter.yml`:
```yaml
collectors:
  enabled: cpu,memory,disk,network,iis,mssql
```

2. Configure Windows Firewall:
```powershell
New-NetFirewallRule -Name "Windows Exporter" `
  -DisplayName "Windows Exporter" `
  -Direction Inbound `
  -Action Allow `
  -Protocol TCP `
  -LocalPort 9182
```

### Verification
```powershell
Invoke-WebRequest http://localhost:9182/metrics
```

## Blackbox Exporter

### Configuration
Create `blackbox.yml`:
```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      preferred_ip_protocol: "ip4"
      valid_status_codes: [200]

  http_post_2xx:
    prober: http
    timeout: 5s
    http:
      method: POST
      preferred_ip_protocol: "ip4"

  tcp_connect:
    prober: tcp
    timeout: 5s
```

### Docker Installation
```bash
docker run -d \
  --name blackbox-exporter \
  -p 9115:9115 \
  -v `pwd`/blackbox.yml:/config/blackbox.yml \
  prom/blackbox-exporter:latest \
  --config.file=/config/blackbox.yml
```

### Prometheus Configuration
Add to `prometheus.yml`:
```yaml
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://${APP_IP}
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

### Verification
```bash
curl "http://localhost:9115/probe?target=http://${APP_IP}&module=http_2xx"
```

## Custom Exporters

### Adding New Exporters
1. Create Docker Compose service
2. Add Prometheus scrape config
3. Configure alerting rules
4. Create Grafana dashboard

### Example Custom Exporter
```yaml
  custom-exporter:
    image: custom-exporter:latest
    ports:
      - "${BIND_ADDRESS}:9999:9999"
    volumes:
      - ./custom-exporter/config:/etc/custom-exporter
    restart: unless-stopped
```

## Troubleshooting

### Common Issues
1. Port conflicts
2. Permission issues
3. Network connectivity
4. Resource constraints

### Debugging Commands
```bash
# Check exporter status
curl -v http://localhost:<port>/metrics

# View exporter logs
docker logs <exporter-container>

# Test network connectivity
telnet <host> <port>
```

## Best Practices

### Security
1. Use non-root users
2. Implement authentication
3. Use network isolation
4. Regular updates

### Performance
1. Optimize scrape intervals
2. Monitor resource usage
3. Use appropriate collectors

### Maintenance
1. Regular updates
2. Backup configurations
3. Monitor exporter health