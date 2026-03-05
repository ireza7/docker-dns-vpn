# Docker DNS VPN

A Docker Compose setup for DNS tunneling using **DNSTT** and **Slipstream** with an integrated SSH server and **dnsdist** load balancer.

## Architecture

```
                    ┌─────────────────────────────────────┐
   Port 53 ──────► │  dnsdist (DNS Load Balancer)         │
   (UDP/TCP)        │  10.220.0.2                          │
                    └────────┬───────────────┬─────────────┘
                             │               │
                  d1.meast.ir│    s1.meast.ir │
                             ▼               ▼
                    ┌────────────┐  ┌─────────────────┐
                    │ DNSTT      │  │ Slipstream       │
                    │ :5353      │  │ :8853            │
                    │ 10.220.0.3 │  │ 10.220.0.4       │
                    └──────┬─────┘  └───────┬──────────┘
                           │                │
                           ▼                ▼
                    ┌───────────────────────────────┐
                    │  SSH Server                    │
                    │  10.220.0.7:22                 │
                    └───────────────────────────────┘
```

## Services

| Service      | Image                       | IP Address   | Port  | Description                              |
| ------------ | --------------------------- | ------------ | ----- | ---------------------------------------- |
| dnsdist      | `powerdns/dnsdist-18:1.8.4` | `10.220.0.2` | 53    | DNS traffic router/load balancer         |
| dnstt-server | `golang:1.22-bookworm`      | `10.220.0.3` | 5353  | DNSTT tunnel server (domain: d1.meast.ir)|
| slipstream   | `debian:bookworm-slim`      | `10.220.0.4` | 8853  | Slipstream DNS tunnel (domain: s1.meast.ir)|
| ssh-server   | `debian:bookworm-slim`      | `10.220.0.7` | 22    | SSH server for tunnel endpoints          |

## Prerequisites

- Docker Engine 24.0+
- Docker Compose v2.20+
- DNS records configured for your domains (NS + A records)

## Quick Start

```bash
# Clone the repository
git clone https://github.com/ireza7/docker-dns-vpn.git
cd docker-dns-vpn

# Start all services
docker compose up -d

# Check service status
docker compose ps

# View logs
docker compose logs -f
```

## DNS Configuration

Configure your DNS records so that your server is the authoritative nameserver:

```
; For DNSTT tunnel
d1.meast.ir.   IN  NS  ns-d1.meast.ir.
ns-d1.meast.ir. IN  A   <YOUR_SERVER_IP>

; For Slipstream tunnel
s1.meast.ir.   IN  NS  ns-s1.meast.ir.
ns-s1.meast.ir. IN  A   <YOUR_SERVER_IP>
```

## Configuration

### SSH Credentials

| Username | Password |
| -------- | -------- |
| netmod   | 123456   |

> **Warning**: Change the default SSH password in production!

### DNSTT Private Key

The DNSTT server uses a private key for authentication. Replace the key in `compose.yml` with your own:

```bash
# Generate a new keypair (requires dnstt-server binary)
dnstt-server -gen-key -privkey-file server.key -pubkey-file server.pub
```

## Troubleshooting

```bash
# Check all services are running
docker compose ps

# View specific service logs
docker compose logs dnsdist
docker compose logs slipstream-server-1
docker compose logs dnstt-server-1
docker compose logs ssh-server

# Restart a specific service
docker compose restart slipstream-server-1

# Rebuild and restart everything
docker compose down && docker compose up -d
```

## Network

All services communicate over a custom bridge network `tunnel_net` with subnet `10.220.0.0/24`.

## License

MIT
