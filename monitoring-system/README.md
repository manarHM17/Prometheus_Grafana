# Monitoring System Stack

A complete monitoring solution using Prometheus, Grafana, and various exporters for system monitoring, with secure configuration management and Docker deployment.

![Architecture Overview](docs/Introduction (2).png)

## Features

- Complete monitoring stack with Prometheus, Grafana, and various exporters
- Secure secrets management and configuration
- Docker-based deployment for easy setup
- Support for multiple exporter types:
  - Node Exporter (Linux systems)
  - WMI Exporter (Windows systems)
  - Blackbox Exporter (endpoint monitoring)
- Automated alerting via Slack
- Grafana dashboards for visualization
- Secure configuration with secrets management

## Quick Start

1. Clone this repository:
```bash
git clone https://github.com/manarHM17/Prometheus_Grafana.git
cd Prometheus_Grafana
```

2. Set up environment variables:
```bash
cp .env.example .env
# Edit .env with your configurations
```

3. Deploy the full stack:
```bash
docker compose -f docker-compose.full-stack.yml up -d
```

4. Access the interfaces:
- Prometheus: http://${SERVER_IP}:9090
- Grafana: http://${SERVER_IP}:3000 (default credentials in docs)
- AlertManager: http://${SERVER_IP}:9093

Note: Replace ${SERVER_IP} with your actual server IP address.

## Documentation

- [Installation Guide](docs/INSTALLATION.md)
- [Configuration Guide](docs/CONFIGURATION.md)
- [Architecture Overview](docs/ARCHITECTURE.md)
- [Troubleshooting Guide](docs/TROUBLESHOOTING.md)

### Component Documentation

- [Prometheus Setup](prometheus/README.md)
- [Grafana Configuration](grafana/README.md)
- [Exporters Setup](exporters/README.md)
- [Alerting Configuration](alerting/README.md)

## Security

All sensitive information (passwords, API keys, endpoints) should be stored in environment variables or Docker secrets. Never commit sensitive data to the repository. See [Configuration Guide](docs/CONFIGURATION.md) for details on secure configuration.

