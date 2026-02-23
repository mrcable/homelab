[![TruffleHog Secrets Scan](https://github.com/mrcable/homelab/actions/workflows/main.yml/badge.svg?branch=main)](https://github.com/mrcable/homelab/actions/workflows/main.yml)

# Homelab

Self-hosted services running on an Intel NUC i5 with Ubuntu Server. Managed via Docker Compose, with Caddy as a reverse proxy providing internal HTTPS for all services.

## Architecture

```
Internet → ISP Modem (bridge) → Unifi USG → Netgear R7000 (WiFi)
                                           → Intel NUC i5 (this repo)
                                           → RasPi 5 (Home Assistant OS)
```

All services are accessible via internal DNS handled by Pi-hole, with TLS termination by Caddy using internal certificates.

## Services

| Service | Internal URL | Description |
|---|---|---|
| [Pi-hole](https://pi-hole.net) | `pihole.internal` | DNS server + network-wide ad blocking |
| [Unifi Controller](https://ui.com) | `unifi.internal` | Unifi network management (USG, switches, APs) |
| [Caddy](https://caddyserver.com) | — | Reverse proxy with automatic internal TLS |

## Prerequisites

- Docker and Docker Compose installed
- Internal DNS resolving `.internal` domains to the host running this stack (via Pi-hole)

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

### Pi-hole

Pi-hole runs with `NET_ADMIN` capability and host networking to handle DNS traffic. The web interface is exposed on port `8089` to avoid conflicts.

## Security

- Secrets are managed via `.env` (not committed to git)
- This repo is automatically scanned for secrets on every push using [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- Caddy uses internal TLS — traffic between client and proxy is encrypted even on the local network
