# Monitoring Tools Setup Guide

## Tools Used:

- **Grafana**: Visualization dashboard.
- **Prometheus**: Monitoring and alerting system.
- **Node Exporter**: Exposes system metrics.

---

## Prerequisites:

- Docker and Docker Compose must be installed.
- Basic understanding of YAML files and Linux commands.

---

## Step 1: Configure Docker Compose File

### 1. Create the `docker-compose.yml` file:

```bash
vim docker-compose.yml
```

### 2. Add the following content:

```yaml
networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}

services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'

    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'

    networks:
      - monitoring

    ports:
      - "9090:9090"
```

---

## Step 2: Create Prometheus Configuration File

### 1. Create the `prometheus.yml` file:

```bash
vim prometheus.yml
```

### 2. Add the following configuration:

```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'aws-server-1'
    static_configs:
      - targets: ['node-exporter:9100']
```

---

## Step 3: Start the Services

### 1. Start services using Docker Compose:

```bash
docker-compose up -d --build
```

### 2. Verify services are running:

```bash
docker ps
```

---

## Step 4: Install Grafana

Run the following command:

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

---

## Step 5: Configure Grafana with Prometheus

### 1. Access Grafana:

- Open a browser and navigate to `http://<server-ip>:3000`.
- Log in with default credentials: `admin/admin`.

### 2. Add Prometheus as a Data Source:

- Navigate to **Home > Connections > Data sources**.
- Select **Prometheus**.
- Set the URL to `http://<server-ip>:9090/`.
- Click **Save & Test**.

---

## Step 6: Extracting Grafana Dashboard ID

If provided with a Grafana dashboard link like:

```
https://grafana.com/grafana/dashboards/1860-node-exporter-full/
```

- Identify the Dashboard ID (`1860`).

---

## Step 7: Import Grafana Dashboard

### 1. Import the Dashboard:

- Go to Grafana Dashboards.
- Click **Import**.
- Enter Dashboard ID (`1860`) and click **Load**.
- Select Prometheus as the data source.
- Click **Import**.

---

## Step 8: Useful Linux Commands

- Monitor processes: `top` or `ps aux`
- Check memory usage: `free -h`
- View CPU info: `cat /proc/cpuinfo`
- View memory info: `cat /proc/meminfo`
- Filter CPU processes: `cat /proc/cpuinfo | grep process`

---

## Verification

- Access Prometheus: `http://<server-ip>:9090`
- Access Grafana: `http://<server-ip>:3000`
- Check the Grafana dashboard for metrics.

---

## Notes

- Replace `<server-ip>` with the server’s IP.
- Adjust the `scrape_interval` or add more `targets` in `prometheus.yml` as needed.

![](https://github.com/abrahimcse/devops-resources/blob/main/Monitoring-Tools/Images/Grafana.png)https://github.com/abrahimcse/devops-resources/blob/main/Monitoring-Tools/Images/Grafana.png
