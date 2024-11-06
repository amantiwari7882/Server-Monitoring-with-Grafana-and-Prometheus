Here's a more polished and formatted version of your GitHub README:

---

# Server Monitoring with Grafana and Prometheus

## Table of Contents
- [Description](#description)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Installation Steps](#installation-steps)
  - [Prerequisites](#prerequisites)
  - [1. Set Up Prometheus on Monitoring Server](#1-set-up-prometheus-on-monitoring-server)
  - [2. Set Up Node Exporter on Target Server](#2-set-up-node-exporter-on-target-server)
  - [3. Set Up Grafana on Monitoring Server](#3-set-up-grafana-on-monitoring-server)
  - [4. Configure Grafana](#4-configure-grafana)
- [Usage](#usage)
  - [1. Access Prometheus](#1-access-prometheus)
  - [2. Access Grafana](#2-access-grafana)
  - [3. Viewing System Metrics on Grafana](#3-viewing-system-metrics-on-grafana)
  - [4. Customizing Prometheus Alerts (Optional)](#4-customizing-prometheus-alerts-optional)
- [Project Structure](#project-structure)

## Description
This project demonstrates how to set up **Prometheus** and **Grafana** using Docker to monitor system metrics on a target server. By following this setup, users can collect and visualize real-time server data such as CPU, memory, disk, and network usage, enabling efficient monitoring and troubleshooting.

The process includes:

- **Setting up Prometheus** to collect metrics from the target server.
- **Using Node Exporter** on the target server to expose system metrics.
- **Integrating with Grafana** to visualize data via customizable dashboards.

### Technologies Used:
- **Prometheus**: For collecting and storing metrics.
- **Node Exporter**: To expose system metrics for Prometheus.
- **Grafana**: For visualizing metrics with customizable dashboards.
- **Docker**: To containerize all components, ensuring portability and scalability.
- **Docker Compose**: For orchestrating multiple containers.
- **Linux Firewall**: For securing the monitoring environment.

This setup is ideal for users looking to proactively monitor server health and performance in real time.

---

## Features

- **Containerized Setup**: Prometheus, Node Exporter, and Grafana run in Docker containers for easy setup, portability, and isolation.
- **Real-Time Monitoring**: Prometheus collects real-time server metrics (CPU, memory, disk usage, network traffic) from Node Exporter.
- **Customizable Dashboards**: Grafana provides interactive, customizable dashboards for detailed visualizations.
- **Multiple Data Sources**: Easily extendable to add more data sources in Grafana.
- **Scalability**: Supports monitoring of multiple servers by simply adding new targets in Prometheus.
- **Security**: Configured with firewall rules to ensure secure communication between servers.
- **Pre-configured Dashboards**: Utilizes Grafana’s **Node Exporter Full Dashboard** template (ID: 1860) for comprehensive visualizations.
- **Data Persistence**: Docker volumes persist data, ensuring continuity in metrics collection and visualization even after restarts.

---

## Tech Stack

- **Prometheus**: Metrics collection and storage.
- **Node Exporter**: Exposes system metrics from target servers.
- **Grafana**: Visualizes Prometheus data through dashboards.
- **Docker**: Containerizes Prometheus, Node Exporter, and Grafana.
- **Docker Compose**: Manages multi-container setup.
- **Linux Firewall**: Configures network security.
- **Rocky Linux 9**: Base operating system for the setup.

---

## Installation Steps

### Prerequisites

- Two Linux servers (e.g., Rocky Linux 9) — one for Prometheus/Grafana and one for the target server with Node Exporter.
- **Docker** and **Docker Compose** installed on both servers.

### 1. Set Up Prometheus on the Monitoring Server

1. **SSH into the Monitoring Server**:
   ```bash
   ssh user@monitoring-server-ip
   ```

2. **Create Required Directories**:
   ```bash
   sudo mkdir -p /opt/docker/prometheus
   cd /opt/docker/prometheus
   ```

3. **Create Docker Compose File** (`docker-compose.yml`):
   ```yaml
   version: '3'
   services:
     prometheus:
       image: prom/prometheus:latest
       container_name: prometheus
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus:/etc/prometheus
         - ./prometheus-data:/prometheus
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'
       restart: unless-stopped
   ```

4. **Create Prometheus Configuration File** (`prometheus.yml`):
   In `/opt/docker/prometheus/prometheus`, create `prometheus.yml` with the following content:
   ```yaml
   global:
     scrape_interval: 15s
   scrape_configs:
     - job_name: 'node_exporter'
       scrape_interval: 5s
       static_configs:
         - targets: ['target-server-ip:9100']
   ```

5. **Start Prometheus**:
   ```bash
   docker-compose up -d
   ```

### 2. Set Up Node Exporter on the Target Server

1. **SSH into the Target Server**:
   ```bash
   ssh user@target-server-ip
   ```

2. **Create Required Directories**:
   ```bash
   sudo mkdir -p /opt/docker/node_exporter
   cd /opt/docker/node_exporter
   ```

3. **Create Docker Compose File** (`docker-compose.yml`):
   ```yaml
   version: '3'
   services:
     node_exporter:
       image: prom/node-exporter:latest
       container_name: node_exporter
       network_mode: host
       restart: unless-stopped
   ```

4. **Open Firewall Port for Node Exporter**:
   ```bash
   sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent
   sudo firewall-cmd --reload
   ```

5. **Start Node Exporter**:
   ```bash
   docker-compose up -d
   ```

### 3. Set Up Grafana on the Monitoring Server

1. **Navigate to Grafana Directory**:
   ```bash
   cd /opt/docker && sudo mkdir grafana && cd grafana
   ```

2. **Create Docker Compose File** (`docker-compose.yml`):
   ```yaml
   version: '3'
   services:
     grafana:
       image: grafana/grafana:latest
       container_name: grafana
       ports:
         - "3000:3000"
       volumes:
         - ./grafana-data:/var/lib/grafana
       restart: unless-stopped
   ```

3. **Start Grafana**:
   ```bash
   docker-compose up -d
   ```

### 4. Configure Grafana

1. **Access Grafana**: Open a browser and navigate to `http://monitoring-server-ip:3000`.
2. **Login**: Use the default credentials (`admin` / `admin`) and change the password when prompted.
3. **Add Prometheus as a Data Source**:
   - Go to **Settings** > **Data Sources** > **Add Data Source**.
   - Choose **Prometheus** and enter the Prometheus URL (`http://monitoring-server-ip:9090`).
   - Click **Save & Test**.
4. **Import Node Exporter Dashboard**:
   - Go to **Create** > **Import**.
   - Enter the dashboard ID `1860` for the **Node Exporter Full Dashboard** and select Prometheus as the data source.

---

## Usage

### 1. Access Prometheus

- **Prometheus Web UI**: Open your browser and navigate to:
  ```url
  http://monitoring-server-ip:9090
  ```
- **Querying Metrics**: Use the Prometheus Web UI to query and view specific metrics. Examples:
  - `node_cpu_seconds_total` for CPU usage.
  - `node_memory_MemAvailable_bytes` for available memory.
  - `node_network_receive_bytes_total` and `node_network_transmit_bytes_total` for network usage.

### 2. Access Grafana

- **Grafana Web UI**: Open your browser and go to:
  ```url
  http://monitoring-server-ip:3000
  ```
- **Login Credentials**: Use the credentials set during installation (default: `admin` / `admin`).

### 3. Viewing System Metrics on Grafana

1. **Open the Dashboard**: Navigate to **Dashboards** > **Manage** and select the **Node Exporter Full** dashboard (ID: 1860).
2. **Visualize Metrics**: The dashboard will display various system metrics like CPU usage, memory utilization, disk I/O, and network traffic in real-time.
3. **Adjust Time Range**: Use the time picker in the top-right corner to select a specific time range (e.g., last 5 minutes, last hour).

### 4. Customizing Prometheus Alerts (Optional)

To set up alerts based on specific conditions:

1. **Edit Prometheus Configuration**: Open the `prometheus.yml` file in `/opt/docker/prometheus/`.
2. **Define Alerting Rules**: Add custom alert rules for thresholds like high CPU usage, low memory, etc.
3. **Reload Prometheus**: Restart the Prometheus container to apply changes:
   ```bash
   cd /opt/docker/prometheus
   docker-compose restart prometheus
   ```

---

## Project Structure

```bash
.
├── docker-compose.yml               # Main Docker Compose file for all services
├── prometheus/
│   ├── docker-compose.yml           # Docker Compose for Prometheus
│   ├── prometheus.yml               # Prometheus configuration (scrape intervals, targets)
│   ├── prometheus/                  # Persistent data for Prometheus
│   └── prometheus-data/             # Data storage for Prometheus
├── node_exporter/
│   ├── docker-compose.yml           # Docker Compose for Node Exporter
│   └── node_exporter/               # Persistent data for Node Exporter
└── grafana/
    ├── docker-compose.yml           # Docker Compose for Grafana
    ├── grafana/                     # Persistent data for Grafana
    └── grafana-data/                # Data storage for Grafana
```
