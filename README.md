# meddiagnose-monitoring

Monitoring stack for the MedDiagnose platform. Prometheus for metrics collection, Grafana for dashboards and alerting.

## Components

```
prometheus/
  prometheus.yml                          # Scrape configs for API, inference, Nginx, Redis
grafana/
  provisioning/
    datasources/prometheus.yml            # Auto-configured Prometheus datasource
alerting/
  alert_rules.yml                         # Prometheus alert rules
docker-compose.yml                        # Standalone monitoring stack
```

## Quick Start

```bash
# Ensure the meddiagnose network exists (created by meddiagnose-infra)
docker network create meddiagnose 2>/dev/null || true

# Start monitoring stack
docker compose up -d

# Prometheus:  http://localhost:9090
# Grafana:     http://localhost:3000  (admin / admin)
```

## Scrape Targets

| Job | Target | Interval | Path |
|-----|--------|----------|------|
| `meddiagnose-api` | `api:8000` | 10s | `/metrics` |
| `meddiagnose-inference` | `inference:8001` | 30s | `/health` |
| `nginx` | `nginx:80` | 15s | `/health` |
| `redis` | `redis:6379` | 30s | -- |

## Alert Rules

| Alert | Condition | Severity |
|-------|-----------|----------|
| **HighErrorRate** | 5xx error rate > 0.1/sec for 2min | Critical |
| **HighLatency** | Avg request duration > 5000ms for 5min | Warning |
| **APIDown** | API target unreachable for 1min | Critical |
| **InferenceDown** | Inference target unreachable for 2min | Critical |

## Available Metrics

The MedDiagnose API exposes these Prometheus metrics at `/metrics`:

| Metric | Type | Description |
|--------|------|-------------|
| `meddiagnose_requests_total` | Counter | Total HTTP requests |
| `meddiagnose_errors_total` | Counter | Total 5xx errors |
| `meddiagnose_request_duration_ms_avg` | Gauge | Average request duration (ms) |

## Grafana Dashboards

Grafana starts with Prometheus pre-configured as a datasource. Create dashboards for:
- Request rate and error rate
- Inference latency (P50, P95, P99)
- Active patients and diagnosis throughput
- System resource usage

## Related Repos

- [meddiagnose-api](https://github.com/AngadBindra46/meddiagnose-api) -- Backend API (exposes `/metrics`)
- [meddiagnose-infra](https://github.com/AngadBindra46/meddiagnose-infra) -- Infrastructure (runs the services being monitored)
