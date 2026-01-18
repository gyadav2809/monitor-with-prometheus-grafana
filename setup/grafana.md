# Grafana Setup (Visualization Layer)

This document explains how Grafana is installed, configured, and connected
to Prometheus in this project.

Grafana is used **only for visualization and exploration of metrics**.
It does not detect failures and does not send alert notifications in this setup.

---

## What Is Grafana?

Grafana is an open-source visualization platform that allows you to:

- Build dashboards using metrics
- Visualize time-series data
- Explore Prometheus metrics interactively
- Understand system behavior during normal operation and failures

Grafana does **not** collect metrics on its own.

---

## What Grafana Is NOT

It is important to clearly understand Grafana’s limitations:

- Grafana does NOT scrape metrics
- Grafana does NOT evaluate alert rules (in this setup)
- Grafana does NOT send alert notifications
- Grafana depends completely on Prometheus for data

Grafana is a **visualization layer**, not a monitoring or alerting engine.

---

## Why Grafana Is Still Important

Even though Grafana is not responsible for alerting here, it is critical because:

- It helps validate that Prometheus is scraping data correctly
- It allows engineers to investigate issues visually
- It provides historical insight into system performance
- It is usually the first interface engineers use during incidents

In real-world environments, Grafana is widely adopted as the standard
observability UI.

---

## Where Grafana Runs

In this project:

- Grafana runs on the **Monitoring EC2 instance**
- Prometheus runs on the same EC2
- Grafana connects to Prometheus using `localhost`
- Grafana listens on port **3000**

---

## Prerequisites

Before installing Grafana, ensure the following:

- Prometheus is already installed and running
- Port **3000** is allowed in the EC2 security group
- Internet access is available for package installation
- You have sudo/root access on the Monitoring EC2

---

## Step 1: Prepare the System

Update the package list and install required dependencies:

```bash
sudo apt update
sudo apt install -y apt-transport-https software-properties-common wget
These packages allow the system to securely fetch Grafana packages.

## Step 2: Add Grafana GPG Key

Add Grafana’s official signing key:
wget -q -O - https://packages.grafana.com/gpg.key | sudo tee /etc/apt/trusted.gpg.d/grafana.asc

## Step 3: Add Grafana Repository

Add the Grafana APT repository:
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update

## Step 4: Install Grafana
sudo apt install -y grafana
Grafana is installed as a systemd-managed service.

## Step 5: Start and Enable Grafana
Enable Grafana to start on boot and start the service:
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

## Step 6: Access Grafana Web Interface

Open a browser and go to:
http://<MONITORING_EC2_PUBLIC_IP>:3000
Default login credentials:
Username: admin
Password: admin

## Step 7: Add Prometheus as a Data Source

In the Grafana UI:
Go to Settings → Data sources
Click Add data source
Select Prometheus
Configure the data source:
URL: http://localhost:9090
Access: Server (default)
Click Save & Test

## Verify Metrics in Grafana - Navigate to Explore in Grafana.

## How Grafana Is Used in This Project
Grafana is used to:
Validate metric availability
Visualize node health
Assist in alert investigation
Provide operational dashboards
Alerting and notifications are handled by Prometheus and Alertmanager.