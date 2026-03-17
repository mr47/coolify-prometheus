Update

# Traefik Commands for Current Setup

These commands should be added to the Coolify proxy Traefik `command:` section.

## Prerequisites

Create required directories on the server:

```bash
mkdir -p /data/coolify/proxy/logs
mkdir -p /data/coolify/proxy/access-logs
```

## Full Traefik Command Block

```yaml
command:
  - '--ping=true'
  - '--ping.entrypoint=http'
  - '--api.dashboard=true'
  - '--entrypoints.http.address=:80'
  - '--entrypoints.https.address=:443'
  - '--entrypoints.http.http.encodequerysemicolons=true'
  - '--entryPoints.http.http2.maxConcurrentStreams=250'
  - '--entrypoints.https.http.encodequerysemicolons=true'
  - '--entryPoints.https.http2.maxConcurrentStreams=250'
  - '--entrypoints.https.http3'
  - '--providers.file.directory=/traefik/dynamic/'
  - '--providers.file.watch=true'
  - '--metrics.prometheus=true'
  # dedicated metrics entrypoint for Prometheus scraping
  - '--entrypoints.metrics.address=:8082'
  - '--metrics.prometheus.entryPoint=metrics'
  #- '--certificatesresolvers.letsencrypt.acme.httpchallenge=true'
  #- '--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=http'
  #- '--certificatesresolvers.letsencrypt.acme.storage=/traefik/acme.json'
  - '--api.insecure=false'
  - '--providers.docker=true'
  - '--providers.docker.exposedbydefault=false'
  # general traefik log file
  - '--log.filepath=/traefik/logs/traefik.log'
  - '--log.level=INFO'
  # access logs for web analytics (JSON format)
  - '--accesslog=true'
  - '--accesslog.filepath=/traefik/access-logs/access.json'
  - '--accesslog.format=json'
  - '--accesslog.fields.defaultmode=keep'
  - '--accesslog.fields.headers.defaultmode=keep'
  - '--accesslog.fields.headers.names.User-Agent=keep'
  - '--accesslog.fields.headers.names.Referer=keep'
```

## Path Mapping

| Purpose | Traefik container path | Host path | Alloy container path |
|---|---|---|---|
| General logs | `/traefik/logs/traefik.log` | `/data/coolify/proxy/logs/traefik.log` | `/var/log/traefik/*.log` |
| Access logs (JSON) | `/traefik/access-logs/access.json` | `/data/coolify/proxy/access-logs/access.json` | `/var/log/traefik-access/*.json` |
| System auth log | n/a | `/var/log/auth.log` | `/var/log/auth.log` |

## Stack Versions

| Component | Version |
|---|---|
| Prometheus | v3.10.0 |
| Loki | v3.6.7 |
| Grafana Alloy | v1.14.0 (replaces Promtail) |
| Node Exporter | v1.10.2 |
| cAdvisor | v0.55.1 |
| Pushgateway | v1.11.2 |

## Alloy Log Collection Jobs

- **`auth`** — System auth logs (`/var/log/auth.log`)
- **`traefik`** — General Traefik logs (plain text, `/var/log/traefik/*.log`)
- **`traefik_access`** — JSON access logs with web analytics pipeline (OS, Browser, Device, GeoIP)

## Loki 3.x Upgrade Notes

- Schema upgraded to `v13` (TSDB), old `v11` (boltdb-shipper) entry removed (unsupported in Loki 3.6.x)
- `compress_responses` removed (deprecated in Loki 3.x)
- User changed from `1000:1000` to `10001:10001` (Loki 3.x default)

## Reference

Based on: https://oglimmer.medium.com/web-analytics-with-grafana-loki-ad64e05cf9f3
