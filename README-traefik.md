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

| Purpose | Traefik container path | Host path | Promtail container path |
|---|---|---|---|
| General logs | `/traefik/logs/traefik.log` | `/data/coolify/proxy/logs/traefik.log` | `/var/log/traefik/*.log` |
| Access logs (JSON) | `/traefik/access-logs/access.json` | `/data/coolify/proxy/access-logs/access.json` | `/var/log/traefik-access/*.json` |
| System auth log | n/a | `/var/log/auth.log` | `/var/log/auth.log` |

## Promtail Jobs

- **`auth`** — System auth logs (SSH, sudo, etc.)
- **`traefik`** — General Traefik logs (plain text)
- **`traefik-access-logs`** — JSON access logs with web analytics pipeline (OS, Browser, Device, GeoIP)

## Reference

Based on: https://oglimmer.medium.com/web-analytics-with-grafana-loki-ad64e05cf9f3
