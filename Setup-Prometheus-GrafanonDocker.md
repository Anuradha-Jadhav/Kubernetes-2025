# Prometheus & Grafana Monitoring Setup

## Overview

This guide provides step-by-step instructions to set up Prometheus and Grafana for monitoring Docker containers using cAdvisor. It also covers setting up Google Sheets as a data source in Grafana.

## Prerequisites

- Docker installed on the host machine
- Internet access to pull required images
- Google API key for Google Sheets integration (optional)

## Network Setup

Create a dedicated Docker network for Prometheus and Grafana:

```sh
sudo docker network create pro-network
```

## Running cAdvisor (Container Metrics Collector)

```sh
docker run -d --name=cadvisor --network pro-network --restart unless-stopped \
  -p 8081:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker/:/var/lib/docker:ro \
  gcr.io/cadvisor/cadvisor
```

## Running Prometheus

```sh
sudo docker run -d -p 9090:9090 --network pro-network \
  -v prometheus-data:/prometheus prom/prometheus
```

## Prometheus Configuration

Create a `prometheus.yml` configuration file:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: []

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "docker"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]
```

Copy the configuration file into the Prometheus container:

```sh
sudo docker cp prometheus.yml <prometheus_container_id>:/etc/prometheus/prometheus.yml
```

## Alerting Configuration

Create `alerts.yml`:

```yaml
groups:
  - name: docker_alerts
    interval: 15s
    rules:
      - alert: DockerContainerDown
        expr: count(container_last_seen) by (name) == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Docker Container Down"
          description: "Container {{ $labels.name }} has been down for more than 1 minute."

      - alert: DockerDaemonDown
        expr: up{job="docker"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Docker Daemon Down"
          description: "Docker daemon is not reachable."

      - alert: cAdvisorDown
        expr: up{job="cadvisor"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "cAdvisor Down"
          description: "cAdvisor service is not reachable."
```

Copy the alert configuration into the Prometheus container:

```sh
sudo docker cp alerts.yml <prometheus_container_id>:/etc/prometheus/
```

## Testing Metrics Collection

To check cAdvisor container metrics:

```sh
curl -s http://localhost:8081/metrics | grep jenkins
```

## Setting Up Grafana

Run Grafana as a Docker container:

```sh
docker run -d -p 3000:3000 --name=grafanaserver \
  -e "GF_SECURITY_ADMIN_USER=admin" \
  -e "GF_SECURITY_ADMIN_PASSWORD=password" \
  grafana/grafana:latest
```

## Installing Grafana Plugin (Google Sheets Data Source)

Enter the Grafana container:

```sh
docker exec -it grafanaserver bash
```

Install the Google Sheets plugin:

```sh
grafana-cli plugins install grafana-googlesheets-datasource
```

Alternatively, start Grafana with the plugin pre-installed:

```sh
docker run -d -p 3000:3000 --name=grafanaserver \
  -e "GF_INSTALL_PLUGINS=grafana-googlesheets-datasource" \
  -e "GF_SECURITY_ADMIN_USER=admin" \
  -e "GF_SECURITY_ADMIN_PASSWORD=password" \
  grafana/grafana:latest
```

## Google API Key & Sheet ID Setup

To configure Google Sheets as a data source in Grafana, follow these steps:

1. Go to [Google Cloud Console](https://console.cloud.google.com/home/)
2. Enable the Google Sheets API
3. Generate an API key and use it in Grafana
4. Copy your Google Sheet ID from the URL:
   - Example URL: `https://docs.google.com/spreadsheets/d/GOOGLE_SHEET_ID/edit#gid=0`
   - The `GOOGLE_SHEET_ID` is the string between `/d/` and `/edit`
5. Access your Google Sheets at: [Google Sheets](https://docs.google.com/spreadsheets/)

## Conclusion

You have successfully set up Prometheus and Grafana for monitoring Docker containers. You can now explore Grafana dashboards and configure alerts for better observability.

---

Reference Video for Grafana:

[https://www.youtube.com/watch?v=GnWZsHjM5To](https://www.youtube.com/watch?v=GnWZsHjM5To)

---

###
