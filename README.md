# Self-Hosted CI/CD & Observability Stack

Infrastructure-as-code and documentation for a self-hosted GitLab CI/CD pipeline and full observability stack running on Arch Linux.


## About

This project documents the design, configuration, and operation of a homelab CI/CD and monitoring platform. Everything runs on bare metal via Docker, with GitLab CE handling version control and pipelines, and a complete LGTM stack providing observability across the entire infrastructure.


## CI/CD

- **GitLab CE** — Self-hosted via Docker, source of truth for all repositories
- **GitLab Runner** — Docker executor, instance-level runner handling all project pipelines
- **Mirror Pipeline** — Automated push from GitLab to GitHub on every commit to `main`


## Observability

- **Grafana** — Dashboards and visualization
- **Loki** — Log aggregation
- **Mimir** — Metrics storage (Prometheus-compatible)
- **Tempo** — Distributed tracing
- **Alertmanager** — Alert routing and notifications
- **Alloy** — Telemetry collector (replaces Promtail/Agent)
- **node-exporter** — Host-level metrics


## Supporting Infrastructure

- **Garage** — S3-compatible object storage
- **Tailscale** — Mesh VPN for secure remote access
- **HashiCorp Consul** — Service discovery
- **HashiCorp Nomad** — Workload orchestration
