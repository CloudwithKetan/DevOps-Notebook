# Prometheus + Grafana + Loki Monitoring Setup (AWS EC2)

## Introduction

In modern DevOps environments, we need visibility into two things:

1. Infrastructure health (CPU, Memory, Disk, Network)
2. Application logs and errors

To achieve this, we use:

* Prometheus for metrics collection
* Grafana for visualization
* Loki for centralized log management
* Node Exporter for server metrics
* Promtail for log collection

This combination provides a complete monitoring and observability solution.

---

# Why Do We Need These Tools?

Imagine an application is running on an EC2 instance.

Suddenly users report:

> "The application is slow."

As a DevOps Engineer, the first questions are:

* Is CPU utilization high?
* Is memory exhausted?
* Is disk space full?
* Are there application errors?
* Did a service crash?

Without monitoring tools, finding the root cause takes a lot of time.

Prometheus, Grafana, and Loki help us answer these questions quickly.

---

# Architecture Overview

We maintain two servers:

### Monitoring Server

This server hosts:

* Prometheus
* Grafana
* Loki

### Application Server

This server hosts:

* Application
* Node Exporter
* Promtail

Data flow:

Metrics:

Application Server → Node Exporter → Prometheus → Grafana

Logs:

Application Logs → Promtail → Loki → Grafana

---

# Prometheus

Prometheus is responsible for collecting metrics from servers.

It does not push data.

Instead, Prometheus continuously pulls data from configured targets.

Example:

Node Exporter exposes metrics on:

http://server-ip:9100/metrics

Prometheus accesses this endpoint every few seconds and stores the collected data.

Examples of collected metrics:

* CPU utilization
* Memory utilization
* Disk usage
* Network traffic
* System load
* Uptime

Prometheus stores all information as time-series data.

---

# Node Exporter

Node Exporter is installed on every Linux server we want to monitor.

Its job is very simple:

It reads operating system metrics and exposes them to Prometheus.

Without Node Exporter, Prometheus cannot directly collect Linux server metrics.

Think of Node Exporter as a bridge between the Linux operating system and Prometheus.

---

# Grafana

Grafana provides dashboards and visualization.

Prometheus stores data but the user experience is limited.

Grafana makes the data easy to understand.

Using Grafana we can visualize:

* CPU Usage
* Memory Usage
* Disk Utilization
* Network Traffic
* Server Health

Most organizations use Grafana because of its powerful dashboard capabilities.

Common dashboard:

Node Exporter Full (Dashboard ID: 1860)

This dashboard provides a complete server health overview.

---

# Loki

Prometheus works very well for metrics.

However, Prometheus is not designed to store application logs.

For logs we use Loki.

Loki is a lightweight log aggregation platform developed by Grafana Labs.

Its purpose is:

* Collect logs
* Store logs
* Search logs
* Analyze errors

Example:

Application log:

ERROR Database Connection Failed

Instead of logging into multiple servers and checking log files manually, Loki centralizes everything.

---

# Promtail

Promtail is the log collection agent.

It reads log files from Linux servers and forwards them to Loki.

Example log locations:

/var/log/messages

/var/log/syslog

/var/log/application.log

Promtail continuously watches these files and pushes new log entries to Loki.

---

# AWS Deployment

## Monitoring EC2

Services:

* Prometheus
* Grafana
* Loki

Recommended Instance:

t3.medium

Security Group:

22 - SSH

3000 - Grafana

9090 - Prometheus

3100 - Loki

---

## Application EC2

Services:

* Application
* Node Exporter
* Promtail

Security Group:

22 - SSH

9100 - Node Exporter

---

# Troubleshooting Example

Suppose users report:

> Application is responding slowly.

Step 1:

Open Grafana dashboard.

Check:

* CPU
* Memory
* Disk

Step 2:

If CPU is high, identify the process consuming resources.

Step 3:

Open Grafana Explore.

Select Loki.

Search:

ERROR

or

Exception

Step 4:

Review logs.

Example:

ERROR Database Connection Timeout

Now we know the slowdown is caused by database connectivity issues.

Without centralized monitoring this investigation would take much longer.

---

# Real Interview Explanation

"We deployed Prometheus, Grafana, and Loki on a dedicated monitoring server. Node Exporter was installed on application servers to expose infrastructure metrics, while Promtail collected logs and forwarded them to Loki. Prometheus scraped metrics from Node Exporter, Grafana visualized both metrics and logs, and Loki provided centralized log storage and search capabilities. This setup enabled proactive monitoring, troubleshooting, and performance analysis across our AWS environment."

---

# Key Learning Points

Prometheus = Metrics Collection

Node Exporter = Linux Metrics Provider

Grafana = Dashboards & Visualization

Promtail = Log Collector

Loki = Log Storage & Search

Together they provide a complete monitoring and observability platform for AWS infrastructure and applications.
