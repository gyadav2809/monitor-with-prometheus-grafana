# Prometheus Setup (Monitoring EC2)

This document explains how Prometheus is installed, configured, and verified
in this project.

Prometheus is the core component responsible for collecting metrics and
evaluating alert rules. It acts as the “brain” of the monitoring system.

---

## What Is Prometheus?

Prometheus is a time-series monitoring system that:

- Scrapes metrics from targets over HTTP
- Stores metrics as time-series data
- Evaluates alert rules based on metrics
- Sends alerts to Alertmanager

Prometheus does **not** send notifications directly.
It only decides **when something is wrong**.

---

## Why Prometheus Is Needed

Without Prometheus:

- Metrics from Node Exporter are not collected
- No historical data exists
- No alert logic can be evaluated
- Grafana has no data source

Prometheus is the central decision-making component.

---

## Where Prometheus Runs

In this setup:

- Prometheus runs on a **dedicated Monitoring EC2**
- It scrapes Node Exporter from other EC2 instances
- It sends alerts to Alertmanager running on the same server

This separation ensures monitoring continues even if a node fails.

---

## Prerequisites

Before installing Prometheus:

- Monitoring EC2 with Linux (Ubuntu used here)
- SSH access
- Internet access to download binaries
- Port **9090** allowed (for Prometheus UI)
- Node Exporter already running on monitored nodes

---

## Step 1: Download Prometheus

Connect to the **Monitoring EC2** and run:


wget https://github.com/prometheus/prometheus/releases/download/v3.9.1/prometheus-3.9.1.linux-amd64.tar.gz

or checkout latest release https://github.com/prometheus/prometheus/releases

## Step 2: Extract the Archive
tar xvf prometheus-3.9.1.linux-amd64.tar.gz
cd prometheus-3.9.1.linux-amd64

## Step 3: Create Directories
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus

## Step 4: Move Binaries
sudo mv prometheus promtool /usr/local/bin/
This makes Prometheus available system-wide.

## Step 5: Create Prometheus System User
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus

## Step 6: Set Permissions
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus

## Step 7: Create Prometheus Configuration
sudo vim /etc/prometheus/prometheus.yml

paste the following in prometheus file - 

------------------------------------------------------------------------------------------------
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "<NODE_PRIVATE_IP>:9100"
```
-----------------------------------------------------------------------------------

Replace: <NODE_PRIVATE_IP> with the private IP of the Node EC2


## Step 8: Add Alert Rules Directory
sudo mkdir -p /etc/prometheus/rules  - Prometheus loads alert rules from this directory.

## Step 9: Configure Rule Files - Edit Prometheus config again:
sudo nano /etc/prometheus/prometheus.yml
Add this at top level (not inside scrape_configs):

rule_files:
  - "/etc/prometheus/rules/*.yml"

## Step 10: Create Prometheus systemd Service
sudo vim /etc/systemd/system/prometheus.service

paste following in the service file - 
----------------------------------------------------------------------------------------------------
**********************************
```yaml
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
***********************************************
----------------------------------------------------------------------------------------------------

## Step 11: Start and Enable Prometheus
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

## Step 12: Open Prometheus UI and Verify Node Exporter Target

Open Prometheus UI: http://<MONITORING_EC2_PUBLIC_IP>:9090
verify node target - In Prometheus UI: Status → Targets

 ## Key Takeaways
Prometheus is the decision-maker
It scrapes metrics and evaluates rules
It does not send notifications directly
Alerting depends on correct integration with Alertmanager




