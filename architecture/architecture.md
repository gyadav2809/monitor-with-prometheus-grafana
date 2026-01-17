# Architecture Overview

This document explains the overall architecture of the monitoring and alerting
setup implemented in this project.

The goal of this architecture is not just to collect metrics, but to reliably
detect failures, generate alerts, and notify humans in a real-world scenario.

This setup is intentionally designed to be simple, explicit, and easy to reason
about.

---

## High-Level Architecture

The system consists of two types of EC2 instances:

1. **Monitoring Server**
2. **Monitored Node(s)**

Each component has a single responsibility.

---

## Components and Their Responsibilities

### 1. Monitoring Server (EC2)

This EC2 instance runs all monitoring and alerting services.

It hosts:

- **Prometheus**
- **Alertmanager**
- **Grafana**

This server acts as the *central brain* of the system.

#### Why a dedicated monitoring server?

- Keeps monitoring separate from application workloads
- Makes alerting independent of node failures
- Matches real-world production designs
- Easier to secure, scale, and troubleshoot

---

### 2. Monitored Node(s) (EC2)

These EC2 instances represent application servers or infrastructure nodes.

Each monitored node runs:

- **Node Exporter**

Node Exporter exposes system-level metrics such as:
- CPU usage
- Memory usage
- Disk usage
- Network statistics

---

## Component Roles Explained Simply

### Prometheus (The Brain)

Prometheus is responsible for:

- Pulling (scraping) metrics from Node Exporter
- Storing time-series data
- Evaluating alert rules

Prometheus does **not** send emails or notifications directly.

Its only job is to decide:
> “Is something wrong, based on the data?”

---

### Alertmanager (The Messenger)

Alertmanager receives alerts from Prometheus and decides:

- Where alerts should go (email, Slack, PagerDuty, etc.)
- When alerts should be grouped
- When alerts should be repeated
- When resolved alerts should be sent

Alertmanager does **not** detect problems.
It only handles alerts **after Prometheus fires them**.

---

### Grafana (Visualization Only)

Grafana is used for:

- Visualizing metrics
- Building dashboards
- Exploring system behavior

Grafana:
- Does NOT detect failures
- Does NOT send alerts in this setup
- Depends entirely on Prometheus as a data source

Grafana is optional for alerting but essential for observability.

---

### Node Exporter (The Sensor)

Node Exporter runs on each monitored EC2 instance.

It:
- Exposes system metrics at `/metrics`
- Does not store data
- Does not evaluate rules
- Does not send alerts

If Node Exporter stops or the node becomes unreachable,
Prometheus detects this via scrape failures.

---

## Data Flow (End-to-End)

The complete data and alert flow looks like this:

1. Node Exporter exposes metrics on the monitored node
2. Prometheus periodically scrapes those metrics
3. Prometheus evaluates alert rules
4. If a rule condition is met:
   - Alert moves from `inactive` → `pending` → `firing`
5. Prometheus sends the alert to Alertmanager
6. Alertmanager sends a notification (email)
7. When the issue is fixed:
   - Alert resolves
   - Alertmanager sends a resolved notification

---

## Failure Detection Philosophy

This setup focuses on **absence of signal**, not dashboards.

The primary alert used is:

```promql
up == 0
