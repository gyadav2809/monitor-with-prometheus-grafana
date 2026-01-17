# Node Exporter Setup (Monitored EC2)

This document explains what Node Exporter is, why it is needed, and how to install,
configure, and verify it on an EC2 instance that will be monitored by Prometheus.

Node Exporter is the foundation of system-level monitoring in this project.

---

## What Is Node Exporter?

Node Exporter is a Prometheus exporter that exposes system metrics such as:

- CPU usage
- Memory usage
- Disk usage
- Network statistics
- Load averages

It runs as a small HTTP service and exposes metrics at:

http://<node-ip>:9100/metrics


Prometheus periodically scrapes this endpoint.

---

## Why Node Exporter Is Required

Prometheus does not collect system metrics by itself.

Without Node Exporter:
- Prometheus has nothing to scrape
- No CPU, memory, or disk metrics exist
- No reliable way to detect node-level failures

Node Exporter acts as the **sensor** on each monitored EC2 instance.

---

## Where Node Exporter Runs

- Installed on **every EC2 instance that needs monitoring**
- Runs independently of Prometheus
- Does not store data
- Does not send alerts

In this project:
- Node Exporter runs on the **Monitored EC2**
- Prometheus runs on a **separate Monitoring EC2**

---

## Prerequisites

Before installing Node Exporter:

- EC2 instance running Linux (Ubuntu used here)
- SSH access to the instance
- Port **9100** allowed from the Prometheus server
- Internet access to download binaries

---

## Step 1: Download Node Exporter

Connect to the **Node EC2** and run:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz


You can also refer to official node_exporter Github repo for latest release and choose according your requirement.

https://github.com/prometheus/node_exporter/releases

## Step 2: Extract the Binary

tar xvf node_exporter-1.10.2.linux-amd64.tar.gz
cd node_exporter-1.10.2.linux-amd64

## Step 3: Move Binary to a Standard Location
sudo mv node_exporter /usr/local/bin/


-- Why /usr/local/bin:
Standard location for manually installed binaries
Accessible system-wide
Keeps filesystem clean

## Step 4: Create a Dedicated System User
sudo useradd --no-create-home --shell /usr/sbin/nologin node_exporter


--Why this matters:
Node Exporter does not need root access
Reduces security risk
Follows production best practices

## Step 5: Create systemd Service
Node Exporter must run continuously and survive reboots. If system reboots or you shuydown then again you have to start node_exporter via running ## sudo systemctl start node_exporter

Create a service file:
sudo nano /etc/systemd/system/node_exporter.service

 ## paste the following in service file -
--------------------------------------------------------------------------------------------

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

-------------------------------------------------------------------------------------------


## step 6: Start and Enable Node Exporter
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

---This ensures:
Node Exporter starts on boot
Automatically restarts if it crashes

## Step 7: Verify Node Exporter Locally
Check service status run - 

sudo systemctl status node_exporter

## step 8
Now verify the node_exporter locally - sudo systemctl status node_exporter
Check the listing port of node_exporter - ss -lntp | grep 9100

### Step 9: Allow Prometheus to Access Node Exporter
Ensure the monitored EC2 security group allows:
Inbound  TCP      Port: 9100

Source: Prometheus EC2 private IP or security group
Node Exporter should not be publicly exposed unless required.

## Step 10: How Prometheus Connects to Node Exporter

Node Exporter does not initiate any connection.
http://<node-private-ip>:9100/metrics

This is configured later in the Prometheus setup.

##  Key Takeaways
** Node Exporter is mandatory for system monitoring
** It runs on every monitored node
** It exposes metrics, but does not store or analyze them
** Prometheus depends on Node Exporter for node-level visibility

