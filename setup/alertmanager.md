````md
# Alertmanager Setup (Alert Routing & Notifications)

This document explains how Alertmanager is installed, configured, and integrated
with Prometheus in this project.

Alertmanager is responsible for **receiving alerts from Prometheus** and
**delivering notifications to humans** through email.

This setup uses Gmail SMTP for alert notifications and is fully tested using
real EC2 shutdown scenarios.

---

## What Is Alertmanager?

Alertmanager is a companion service to Prometheus that handles alerts after they
are fired.

Prometheus decides **when something is wrong**.  
Alertmanager decides **who should be notified and how**.

Alertmanager is responsible for:
- Routing alerts
- Grouping alerts
- Sending notifications
- Sending resolved notifications

Alertmanager does **not** evaluate alert conditions.

---

## Why Alertmanager Is Needed

Without Alertmanager:

- Prometheus can detect problems but cannot notify humans
- Alerts remain visible only in the Prometheus UI
- No email, Slack, or PagerDuty notifications are possible

Alertmanager is the **communication layer** of the monitoring system.

---

## Where Alertmanager Runs

In this project:

- Alertmanager runs on the **Monitoring EC2**
- Prometheus runs on the same EC2
- Alertmanager listens on port **9093**
- Prometheus sends alerts to Alertmanager over HTTP

---

## Prerequisites

Before installing Alertmanager:

- Prometheus must already be running
- Port **9093** must be allowed in the EC2 security group
- A Gmail account with **App Password** enabled
- Internet access to download binaries

---

## Step 1: Download Alertmanager

On the **Monitoring EC2**, run:

wget https://github.com/prometheus/alertmanager/releases/download/v0.30.1/alertmanager-0.30.1.linux-amd64.tar.gz

## Step 2: Extract Alertmanager
tar xvf alertmanager-0.30.1.linux-amd64.tar.gz
sudo mv alertmanager-0.30.1.linux-amd64 /etc/alertmanager

## Step 3: Create Directories
sudo mkdir -p /etc/alertmanager
sudo mkdir -p /var/lib/alertmanager

Directory purpose:
/etc/alertmanager → configuration
/var/lib/alertmanager → runtime data

## Step 4: Create Alertmanager User
sudo useradd --no-create-home --shell /usr/sbin/nologin alertmanager
This improves security by avoiding root execution.

## Step 5: Set Permissions
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chown -R alertmanager:alertmanager /var/lib/alertmanager

## Step 6: Create Alertmanager Configuration
Create the configuration file:
sudo vim /etc/alertmanager/alertmanager.yml

Paste the following complete configuration:
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'YOUR_GMAIL@gmail.com'
  smtp_auth_username: 'YOUR_GMAIL@gmail.com'
  smtp_auth_password: 'GMAIL_APP_PASSWORD'
  smtp_require_tls: true

route:
  receiver: 'email-alerts'
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

receivers:
- name: 'email-alerts'
  email_configs:
  - to: 'YOUR_GMAIL@gmail.com'
    send_resolved: true

Replace:

YOUR_GMAIL@gmail.com with your Gmail address

GMAIL_APP_PASSWORD with your Gmail App Password (no spaces)

## Step 7: Create systemd Service
Create the service file:
sudo nano /etc/systemd/system/alertmanager.service

Paste the following in the file.
```yaml

[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/opt/alertmanager/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
## Step 8: Start and Enable Alertmanager
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager

## Step 9: Verify Alertmanager
sudo systemctl status alertmanager
ss -lntp | grep 9093
http://<MONITORING_EC2_PUBLIC_IP>:9093


## Step 10: Integrate Alertmanager with Prometheus

Ensure Prometheus config includes: the file we create in STEP-7 of prometheus installation - sudo vim /etc/prometheus/prometheus.yml
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"
```
and restart prometheus - sudo systemctl restart prometheus


## Step 11: Verify Alerts

Shut down the monitored EC2 instance.
Expected behavior:
Prometheus fires NodeDown
Alert appears in Prometheus
Alert appears in Alertmanager
Email notification is sent (FIRING)
When EC2 starts again → RESOLVED email is sent
