# Step-by-Step Setup: Prometheus + Grafana + Loki on AWS EC2

## Target Architecture

```text
Monitoring EC2
│
├── Prometheus (9090)
├── Grafana (3000)
└── Loki (3100)

Application EC2
│
├── Application
├── Node Exporter (9100)
└── Promtail
```

---

# Step 1: Launch EC2 Instances

Create two EC2 instances.

### Monitoring Server

Purpose:

* Prometheus
* Grafana
* Loki

Configuration:

* Ubuntu 24.04
* t3.medium
* 30 GB Storage

Security Group:

| Port | Purpose    |
| ---- | ---------- |
| 22   | SSH        |
| 3000 | Grafana    |
| 9090 | Prometheus |
| 3100 | Loki       |

---

### Application Server

Purpose:

* Application
* Node Exporter
* Promtail

Configuration:

* Ubuntu 24.04
* t3.small

Security Group:

| Port | Purpose       |
| ---- | ------------- |
| 22   | SSH           |
| 9100 | Node Exporter |

---

# Step 2: Install Docker on Monitoring Server

SSH into Monitoring Server.

```bash
sudo apt update

sudo apt install docker.io docker-compose -y

sudo systemctl enable docker

sudo systemctl start docker
```

Verify:

```bash
docker --version
docker-compose --version
```

---

# Step 3: Create Monitoring Directory

```bash
mkdir monitoring

cd monitoring
```

---

# Step 4: Create Docker Compose File

Create:

```bash
nano docker-compose.yml
```

Paste:

```yaml
version: '3.8'

services:

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus

    ports:
      - "9090:9090"

    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    container_name: grafana

    ports:
      - "3000:3000"

  loki:
    image: grafana/loki:latest
    container_name: loki

    ports:
      - "3100:3100"

    command: -config.file=/etc/loki/local-config.yaml
```

Save file.

---

# Step 5: Create Prometheus Configuration

```bash
nano prometheus.yml
```

Replace APP_PRIVATE_IP with Application Server private IP.

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: "node_exporter"

    static_configs:
      - targets:
          - "APP_PRIVATE_IP:9100"
```

Example:

```yaml
- targets:
  - "172.31.10.25:9100"
```

Save file.

---

# Step 6: Start Monitoring Stack

```bash
docker-compose up -d
```

Verify:

```bash
docker ps
```

Expected:

```text
prometheus
grafana
loki
```

---

# Step 7: Install Node Exporter on Application Server

SSH into Application Server.

Download:

```bash
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-1.9.1.linux-amd64.tar.gz

tar -xvf node_exporter-*.tar.gz
```

Copy binary:

```bash
sudo cp node_exporter-*/node_exporter /usr/local/bin/
```

Create service:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste:

```ini
[Unit]
Description=Node Exporter

[Service]
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Start service:

```bash
sudo systemctl daemon-reload

sudo systemctl enable node_exporter

sudo systemctl start node_exporter
```

Verify:

```bash
systemctl status node_exporter
```

Test:

```bash
curl localhost:9100/metrics
```

---

# Step 8: Verify Prometheus

Open browser:

```text
http://MONITORING_PUBLIC_IP:9090
```

Navigate:

Status → Targets

Expected:

```text
node_exporter = UP
```

If UP is green, Prometheus is scraping metrics successfully.

---

# Step 9: Configure Grafana

Open:

```text
http://MONITORING_PUBLIC_IP:3000
```

Login:

```text
admin
admin
```

Change password.

---

# Step 10: Add Prometheus Data Source

Grafana

Connections → Data Sources → Add Data Source

Select:

```text
Prometheus
```

URL:

```text
http://prometheus:9090
```

Save & Test.

Expected:

```text
Data source is working
```

---

# Step 11: Import Dashboard

Grafana

Dashboards → Import

Dashboard ID:

```text
1860
```

Select Prometheus datasource.

Import.

You should now see:

* CPU Usage
* Memory Usage
* Disk Usage
* Network Usage

from Application Server.

---

# Step 12: Install Promtail

SSH into Application Server.

Download:

```bash
wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip

unzip promtail-linux-amd64.zip

chmod +x promtail-linux-amd64
```

---

# Step 13: Configure Promtail

Create:

```bash
nano promtail-config.yml
```

Replace LOKI_PRIVATE_IP.

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://LOKI_PRIVATE_IP:3100/loki/api/v1/push

scrape_configs:

  - job_name: system

    static_configs:

      - targets:
          - localhost

        labels:
          job: varlogs
          host: app-server

          __path__: /var/log/*.log
```

Save file.

---

# Step 14: Start Promtail

```bash
./promtail-linux-amd64 \
-config.file=promtail-config.yml
```

Check logs:

```bash
tail -f /var/log/syslog
```

Promtail should start forwarding logs.

---

# Step 15: Add Loki Data Source

Grafana

Connections → Data Sources → Add Data Source

Select:

```text
Loki
```

URL:

```text
http://loki:3100
```

Save & Test.

---

# Step 16: Verify Logs

Grafana

Explore → Loki

Run:

```logql
{job="varlogs"}
```

You should see Linux logs.

Search errors:

```logql
{job="varlogs"} |= "ERROR"
```

Search exceptions:

```logql
{job="varlogs"} |= "Exception"
```

---

# Final Verification

Metrics:

```text
Node Exporter
    ↓
Prometheus
    ↓
Grafana
```

Logs:

```text
Promtail
    ↓
Loki
    ↓
Grafana
```

At this point you have a complete monitoring and logging solution running on AWS.
