# HomeLab Monitoring Server Setup

A simple monitoring setup using Node Exporter, Prometheus, and Grafana.

## Prerequisites

- Docker & Docker Compose
- Node Exporter
- Prometheus
- Grafana

## Architecture

```
Main Server (App Server)                Monitoring Server
┌──────────────────────┐               ┌──────────────────────┐
│  Node Exporter :9100 │ ◄──scrape──── │  Prometheus :9090    │
└──────────────────────┘               │  Grafana    :3000    │
                                       └──────────────────────┘
```

## Setup

### 1. Main Server: Install Node Exporter

Create a `docker-compose.yml` file:

```yaml
services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    pid: host
    network_mode: host # recommended for accurate host metrics
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
```

Run:

```bash
docker compose up -d
```

> **Important:** Open port `9100` only to the Monitoring Server's IP.

```bash
sudo ufw allow from MONITORING_SERVER_IP to any port 9100
```

### 2. Monitoring Server: Install Grafana + Prometheus

Use the configuration in the `monitoring/` folder.

Make sure `prometheus.yml` includes a target pointing to the Main Server, for example:

```yaml
scrape_configs:
  - job_name: main-server
    static_configs:
      - targets: ['MAIN_SERVER_IP:9100']
```

Run:

```bash
cd monitoring
docker compose up -d
```

## Access & Final Setup

| Service    | URL                                | Notes                               |
|------------|------------------------------------|-------------------------------------|
| Grafana    | `http://MONITORING_SERVER_IP:3000` | Login: `admin` / `changeme`         |
| Prometheus | `http://MONITORING_SERVER_IP:9090` | Check **Status → Targets**          |

> ⚠️ **Change the default Grafana password immediately after your first login.**

Verification: in Prometheus under **Status → Targets**, the `main-server` target should show a status of **UP**.

## Import the Node Exporter Dashboard

1. In Grafana, go to **Dashboards → New → Import**
2. Enter the ID: `1860` (**Node Exporter Full**)
3. Select **Prometheus** as the data source
4. Click **Import**

Alternative dashboards: `11074`, `12486`