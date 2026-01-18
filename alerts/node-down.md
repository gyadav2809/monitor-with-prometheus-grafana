# NodeDown Alert (EC2 Availability)

This document defines and explains the **NodeDown alert** used in this project.

This alert detects when an EC2 instance becomes unreachable by Prometheus and
notifies via Alertmanager.

---

## Purpose of This Alert

The NodeDown alert answers one critical question:

> “Is this node reachable right now?”

This is the most **fundamental and reliable** alert in any monitoring system.

If this alert fires, the node is:
- Shut down
- Crashed
- Network unreachable
- Node Exporter not running

---

## Alert Logic

The alert is based on the Prometheus metric:

```promql
up
up == 1 → target is reachable
up == 0 → target is unreachable

## Alert Rule Definition
sudo vim /etc/prometheus/rules/node_alerts.yml

```yaml

groups:
- name: node-alerts
  rules:
  - alert: NodeDown
    expr: up{job="node_exporter"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Node is down"
      description: "Node {{ $labels.instance }} has been unreachable for more than 1 minute."
      
```

Explanation of Fields

alert
Name of the alert.
expr
PromQL expression used to evaluate the alert.
for
Alert fires only if the condition remains true for 1 minute.
This avoids false positives from short network blips.
labels.severity
Used by Alertmanager for routing and prioritization.
annotations
Human-readable text sent in notifications.


 ## Load the Alert Rule
Ensure Prometheus loads rule files:
rule_files:
  - "/etc/prometheus/rules/*.yml"


## Restart Prometheus:
sudo systemctl restart prometheus

## Test the Alert (Failure Simulation)
Shut down the monitored EC2 instance
Wait 1 minute

## Observe:
Alert becomes FIRING in Prometheus
Alert appears in Alertmanager
Email notification is sent
## Alert Resolution Test
Start the EC2 instance again
Node Exporter becomes reachable
Alert transitions to RESOLVED
Resolved email notification is sent

## Known Limitations
Alert message includes instance IP by default
Human-friendly names require relabeling
EC2 service discovery is not used yet
These improvements are planned later.

## Key Takeaways
NodeDown is the most critical infrastructure alert
up == 0 is a strong failure signal
Alert lifecycle must always be tested
Notifications must be validated end-to-end