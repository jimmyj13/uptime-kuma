# Uptime Kuma DevOps Project

A complete DevOps pipeline built around [Uptime Kuma](https://github.com/louislam/uptime-kuma), an open-source monitoring tool. This project demonstrates containerization, CI/CD automation, Kubernetes orchestration, and a full monitoring/alerting stack.

---

## Architecture Overview
```
GitHub Push
    ↓
GitHub Actions (CI/CD)
    ↓ builds & pushes
Docker Hub
    ↓ image pulled by
Kubernetes (Minikube)
    ├── uptime-kuma    → monitors services
    ├── prometheus     → scrapes metrics
    ├── grafana        → visualizes metrics
    └── alertmanager   → sends Slack alerts
         ↓
    Nginx Ingress
    ├── uptime-kuma.local
    ├── grafana.local
    └── prometheus.local
```

---

## Project Structure
```
uptime-kuma/
├── Dockerfile.custom              # Multi-stage Docker build
├── docker-compose.custom.yml      # Local development setup
├── .github/
│   └── workflows/
│       └── ci.yml                 # GitHub Actions CI/CD pipeline
├── k8s/
│   ├── configmap.yaml             # App environment config
│   ├── deployment.yaml            # Kubernetes deployment
│   ├── service.yaml               # NodePort service
│   ├── pvc.yaml                   # Persistent volume for SQLite
│   └── ingress.yaml               # Nginx ingress routing
└── monitoring/
    ├── prometheus-config.yaml     # Scrape config + alert rules
    ├── prometheus-deployment.yaml # Prometheus deployment
    ├── grafana-deployment.yaml    # Grafana + persistent volume
    ├── alertmanager-config.yaml   # Slack notification config
    └── alertmanager-deployment.yaml # Alertmanager deployment
```

---

## Features

- **Docker** — Multi-stage build optimized for production, separating build and runtime layers
- **Docker Compose** — Single command local development environment
- **CI/CD** — GitHub Actions pipeline automatically builds and pushes Docker image to Docker Hub on every push to master, tagged with both `latest` and commit SHA
- **Kubernetes** — Full K8s deployment with health probes, persistent storage, and environment configuration via ConfigMap
- **Ingress** — Nginx Ingress controller for domain-based routing without port numbers
- **Prometheus** — Scrapes `/metrics` endpoint every 15 seconds with three alert rules (HighMemoryUsage, HighEventLoopLag, AppDown)
- **Grafana** — Persistent dashboards showing heap memory, event loop lag, and open connections
- **Alertmanager** — Fires Slack notifications when alert thresholds are breached, with resolved notifications when issues clear

---

## Prerequisites

- Docker
- kubectl
- Minikube
- GitHub account
- Docker Hub account

---

## Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/jimmyj13/uptime-kuma
cd uptime-kuma
```

### 2. Run locally with Docker Compose
```bash
docker compose -f docker-compose.custom.yml up -d
```
Access at `http://localhost:3001`

### 3. Deploy to Kubernetes

Start Minikube:
```bash
minikube start --driver=docker
minikube addons enable ingress
```

Deploy the app:
```bash
kubectl apply -f k8s/
```

Deploy monitoring stack:
```bash
kubectl apply -f monitoring/
```

Add local DNS entries:
```bash
echo "$(minikube ip) uptime-kuma.local grafana.local prometheus.local" | sudo tee -a /etc/hosts
```

### 4. Access the Services

| Service | URL |
|---|---|
| Uptime Kuma | http://uptime-kuma.local |
| Grafana | http://grafana.local |
| Prometheus | http://prometheus.local |

---

## CI/CD Pipeline

The GitHub Actions pipeline triggers on every push to master and:
1. Checks out the code
2. Logs into Docker Hub using repository secrets
3. Builds the Docker image using `Dockerfile.custom`
4. Pushes to Docker Hub with two tags: `latest` and the commit SHA

Required GitHub secrets:
- `DOCKER_USERNAME` — Docker Hub username
- `DOCKER_PASSWORD` — Docker Hub access token

---

## Monitoring & Alerting

Prometheus scrapes metrics from Uptime Kuma every 15 seconds. Three alert rules are configured:

| Alert | Condition | Severity |
|---|---|---|
| HighMemoryUsage | Heap > 250MB for 1 minute | Warning |
| HighEventLoopLag | Event loop lag > 500ms for 1 minute | Warning |
| AppDown | App unreachable for 30 seconds | Critical |

Alerts are routed through Alertmanager to Slack. The Slack webhook URL is stored as a Kubernetes Secret and injected at runtime — never committed to Git.

---

## Key Learnings

- Multi-stage Docker builds reduce final image size by separating build and runtime dependencies
- Kubernetes PersistentVolumeClaims ensure data survives pod restarts
- GitHub secret scanning blocks accidental credential commits — secrets belong in Kubernetes Secrets or environment variables
- Readiness and liveness probes prevent traffic from reaching unhealthy pods
- Ingress eliminates the need for port-based access in production-like environments

---

## Tech Stack

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat&logo=node.js&logoColor=white)