# Prometheus Alerting on EC2 (Real-World Setup)

This repository documents a real-world implementation of monitoring and alerting
using Prometheus, Alertmanager, and Grafana on AWS EC2.

The goal of this project is not just to visualize metrics, but to design,
test, and validate a complete alerting pipeline — including failure simulation
and notification delivery.

---

## Architecture Overview

- One EC2 instance runs:
  - Prometheus
  - Alertmanager
  - Grafana
- One or more EC2 instances act as monitored nodes:
  - Node Exporter runs on each node

Alerts are evaluated by Prometheus and routed by Alertmanager.
Notifications are delivered via email.

---

## Implemented Alert

### NodeDown Alert

An alert is triggered when a monitored node becomes unreachable:

- Condition: `up == 0`
- Duration: 1 minute
- Severity: critical

The alert lifecycle is fully tested:
- Inactive → Pending → Firing → Resolved

---

## Notification

Alert notifications are sent via Gmail SMTP using Alertmanager.
Both FIRING and RESOLVED notifications are delivered.

---

## Why This Repository Exists

Most tutorials stop at dashboards.
This project focuses on:

- Alert correctness
- Failure testing (EC2 shutdown)
- Alert resolution
- Notification reliability
- Real-world debugging scenarios

---

## Scope & Limitations

- Static EC2 IPs are used initially
- Email is the only notification channel
- Kubernetes setup is intentionally excluded (planned next)

---

## Next Steps

- Improve alert labels (remove IPs)
- Add multiple EC2 nodes
- Use EC2 service discovery
- Migrate setup to Kubernetes
