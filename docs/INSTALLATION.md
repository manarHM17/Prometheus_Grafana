# Installation Guide

This guide provides step-by-step instructions for setting up the complete monitoring stack.

## Prerequisites

- Docker Engine (version 20.10.0 or higher)
- Docker Compose (version 2.0.0 or higher)
- Git
- 4GB RAM minimum (8GB recommended)
- 20GB free disk space
- Network access to monitored hosts

## Basic Installation

1. Clone the repository:
```bash
git clone https://github.com/manarHM17/Prometheus_Grafana.git
cd Prometheus_Grafana
```

2. Create required directories for secrets:
```bash
./scripts/setup-secrets.sh
```

3. Set up environment variables:
```bash
cp .env.example .env
# Edit .env with your configurations
```

4. Configure secrets:
- Create Grafana admin password:
```bash
echo "your-secure-password" > secrets/grafana_admin_password.txt
```
- Add Slack webhook URL:
```bash
echo "https://hooks.slack.com/services/YOUR/WEBHOOK/URL" > secrets/alertmanager/slack_webhook_url
```

5. Deploy the stack:
```bash
docker compose -f docker-compose.full-stack.yml up -d
```

## Component Setup

### Node Exporter Setup (Linux)

1. On each Linux host:
```bash
cd exporters/node-exporter/linux
./install.sh
```

### WMI Exporter Setup (Windows)

1. On each Windows host:
- Follow instructions in `exporters/wmi-exporter/install-guide.md`

### Blackbox Exporter Setup

1. Configure endpoints in `exporters/blackbox-exporter/blackbox.yml.template`
2. Update target list in Prometheus file_sd configuration

## Verification

1. Check services are running:
```bash
docker compose -f docker-compose.full-stack.yml ps
```

2. Verify endpoints:
- Prometheus: http://${SERVER_IP}:9090
- Grafana: http://${SERVER_IP}:3000
- AlertManager: http://${SERVER_IP}:9093

Note: Replace ${SERVER_IP} with your actual server IP address (e.g., 192.168.1.100)

3. Check Prometheus targets:
- Visit http://localhost:9090/targets
- All targets should show as "UP"

## Next Steps

- Configure additional alerts in `prometheus/alerts.yml`
- Import dashboards in Grafana
- Set up additional exporters as needed
- Configure Slack notifications

## Troubleshooting

See [Troubleshooting Guide](TROUBLESHOOTING.md) for common issues and solutions.

## Security Considerations

- All sensitive information should be stored in the `secrets/` directory
- Never commit secrets to the repository
- Use environment variables for configuration
- Follow the principle of least privilege for service accounts
- Regularly rotate secrets and credentials

## Backup and Maintenance

See the [Configuration Guide](CONFIGURATION.md) for details on:
- Backing up data
- Updating components
- Managing configurations
- Rotating secrets