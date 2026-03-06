[![TruffleHog Secrets Scan](https://github.com/mrcable/homelab/actions/workflows/main.yml/badge.svg?branch=main)](https://github.com/mrcable/homelab/actions/workflows/main.yml)

# homelab

Self-hosted services running Ubuntu Server on my homeserver. Managed via Docker Compose, with Caddy as a reverse proxy providing internal HTTPS for all services.

## Architecture

```
Internet → ISP Modem (bridge) → Unifi USG → Netgear R7000 (WiFi)
                                           → Intel NUC i5 (Ubuntu-Server)     192.168.1.3
                                           → Intel NUC i5 (Proxmox/k3s)  192.168.1.5
                                           → RasPi 5 (Home Assistant OS)  192.168.1.12
```

All services are accessible via internal DNS handled by Pi-hole. Caddy handles TLS termination and reverse proxying — both for local Docker services and for Kubernetes services running on the k3s cluster.

## Services

| Service | Internal URL | Description |
|---|---|---|
| [Pi-hole](https://pi-hole.net) | `pihole.internal` | DNS server + network-wide ad blocking |
| [Unifi Controller](https://ui.com) | `unifi.internal` | Unifi network management (USG, switches, APs) |
| [Caddy](https://caddyserver.com) | — | Reverse proxy with automatic internal TLS |

Caddy also proxies the following Kubernetes services (running on the k3s cluster via [mrcable/homelab-k8s](https://github.com/mrcable/homelab-k8s)):

| Service | Internal URL | Backend |
|---|---|---|
| ArgoCD | `argocd.internal` | 192.168.1.211 |
| Grafana | `grafana.internal` | 192.168.1.212 |
| Falco UI | `falco.internal` | 192.168.1.213 |
| Proxmox | `proxmox.internal` | 192.168.1.5:8006 |

## Prerequisites

- Docker and Docker Compose installed
- Internal DNS resolving `.internal` domains to this host (via Pi-hole)

## Usage

```bash
# Clone the repo
git clone git@github.com:mrcable/homelab.git
cd homelab

# Copy the example env file and fill in your values
cp .env.example .env
vim .env

# Start all services
docker compose up -d

# Check status
docker compose ps
```

## Configuration

### Environment variables

Copy `.env.example` to `.env` and set the following:

| Variable | Description |
|---|---|
| `PIHOLE_PASS` | Pi-hole web interface password |

### Caddy

The `caddy/Caddyfile` defines the reverse proxy rules. All services are served over HTTPS using Caddy's internal TLS (`tls internal`). Caddy handles certificate generation automatically — no manual cert management needed.

For backends that use self-signed certificates (Unifi, Proxmox, ArgoCD), `tls_insecure_skip_verify` is set in the Caddyfile. This is safe because all traffic stays on the local network.

### Pi-hole

Pi-hole runs with `NET_ADMIN` capability and host networking to handle DNS traffic. The web interface is exposed on port `8089` to avoid conflicts with Caddy.

All `.internal` domains resolve to `192.168.1.3` via Pi-hole local DNS records. Caddy then routes to the correct backend.

## Security

- Secrets are managed via `.env` (not committed to git)
- This repo is automatically scanned for secrets on every push using [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- Caddy uses internal TLS — traffic between client and proxy is encrypted even on the local network
- External attack surface is limited to WireGuard only — all other ports are blocked at the USG

## Related repo

**[mrcable/homelab-k8s](https://github.com/mrcable/homelab-k8s)** — GitOps repository for the k3s cluster on Proxmox (ArgoCD, Prometheus, Grafana, Falco, Loki)





































































