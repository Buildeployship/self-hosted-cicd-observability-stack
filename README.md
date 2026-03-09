# Self-Hosted CI/CD & Observability Stack

Infrastructure-as-code and documentation for a self-hosted GitLab CI/CD pipeline and full observability stack running on Arch Linux.


## About

This project documents the design, configuration, and operation of a homelab CI/CD and monitoring platform. Everything runs on bare metal via Docker, with GitLab CE handling version control and pipelines, and a complete LGTM stack providing observability across the entire infrastructure.


## CI/CD

- **GitLab CE** — Self-hosted via Docker, source of truth for all repositories
- **GitLab Runner** — Docker executor, instance-level runner handling all project pipelines
- **Mirror Pipeline** — Automated push from GitLab to GitHub on every commit to `main`


## Monitoring/Observability

- **Grafana** — Dashboards and visualization
- **Loki** — Log aggregation
- **Mimir** — Metrics storage (Prometheus-compatible)
- **Tempo** — Distributed tracing
- **OTel Collector** — OTLP ingestion (traces/metrics/logs)
- **Alloy** — Telemetry collector (replaces Promtail/Agent)
- **node-exporter** — Host-level metrics
- **Alertmanager** — Alert routing and notifications


## Supporting Infrastructure

- **Garage** — S3-compatible object storage
- **Tailscale** — Mesh VPN for secure remote access
- **HashiCorp Consul** — Service discovery
- **HashiCorp Nomad** — Workload orchestration

## Architecture

```
┌─────────────────────────────────────────────────────────—┐
│                    LOCAL MACHINE                         │
│                                                          │
│   git push gitlab main                                   │
│   remote:      ssh://git@localhost:2222/buildeployship/  │
│   commit email:        noreply@users.noreply.github.com  │
│   signing: SSH key        auth: SSH key pair (port 2222) │
│                                                          │
└────────────────────────────┬─────────────────────────────┘
                             │
                    SSH Push (Port 2222)
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│                  GITLAB CE (DOCKER)                      │
│             localhost:8081 → container:80                │
│                                                          │
│ ┌──────────────────────────────────────────────────────┐ │
│ │  SERVER CONFIG                                       │ │
│ │                                                      │ │
│ │  external_url: http://localhost:8081                 │ │
│ │  nginx listen_port: 80                               │ │
│ │  ports: 8081:80 | 2222:22                            │ │
│ └──────────────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────────────┐ │
│ │  CI/CD PIPELINE — .gitlab-ci.yml                     │ │
│ │                                                      │ │
│ │  stages:                                             │ │
│ │    - mirror                                          │ │
│ │                                                      │ │
│ │  mirror-to-github:                                   │ │
│ │    stage: mirror                                     │ │
│ │    image: alpine:latest                              │ │
│ │    before_script:                                    │ │
│ │      - apk add --no-cache git                        │ │
│ │    script:                                           │ │
│ │      - git remote add github                         │ │
│ │          https://$GITHUB_TOKEN@github.com/           │ │
│ │          Buildeployship/$CI_PROJECT_NAME.git         │ │
│ │      - git push github HEAD:refs/heads/main --force  │ │
│ │    only:                                             │ │
│ │      - main                                          │ │
│ └──────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌─────────────────────┐  ┌────────────────────────────┐ │
│  │  GITLAB RUNNER      │  │  CI/CD VARIABLES           │ │
│  │                     │  │                            │ │
│  │  Executor: Docker   │  │  GITHUB_TOKEN              │ │
│  │  Image: alpine      │  │  ✓ Masked                 │ │
│  │  Tags: docker,      │  │  ✓ Protected              │ │
│  │        homelab      │  │                            │ │
│  └─────────────────────┘  └────────────────────────────┘ │
│                                                          │
└────────────────────────────┬─────────────────────────────┘
                             │
                   HTTPS Push + PAT
                             │
                             ▼
┌──────────────────────────────────────────────────────────┐
│                         GITHUB                           │
│                github.com/Buildeployship                 │
│                                                          │
│  • Repo auto-detected via $CI_PROJECT_NAME               │
│  • Email privacy: noreply address                        │
│  • Push blocks: enabled                                  │
│  • Visibility: Public                                    │
│                                                          │
└──────────────────────────────────────────────────────────┘

git push → GitLab receives → Pipeline triggers → Job queued → Runner executes → GitHub mirrors
```

### Full Chain

```
git push gitlab main
    │
    │  You push the main branch to the gitlab remote
    │
    ▼
GitLab receives the push on main
    │
    │  Reads .gitlab-ci.yml → only: [main] matches
    │
    ▼
Pipeline created
    │
    │  One stage: mirror
    │
    ▼
Job queued (mirror-to-github)
    │
    │  Waits for an available Runner
    │
    ▼
Runner executes
    │
    │  Spins up Alpine container
    │  Installs git
    │  Adds github remote using $GITHUB_TOKEN
    │  Pushes HEAD:refs/heads/main --force
    │
    ▼
GitHub mirrors
    │
    │  Code appears at github.com/Buildeployship/$CI_PROJECT_NAME
    │
    Done
```
